---
title: "Debugging suspend ideas"
date: 2019-01-24T08:18:00+01:00
draft: false
author: Felix
---

## The problem
When porting the Sony Xperia XZ from Kernel 4.4 to 4.9, a nasty bug showed up:
It wouldn't deep sleep for more than ~5.2-5.7 seconds.

A lot had changed between the kernel versions, especially since the Sony Open
Devices Project is not using the standard Linux kernel, but rather a heavily
modified version based on Qualcomm's CodeAurora Forum("CAF") releases.

There are also a lot of modifications to bring the kernel in line with upstream
mainline Linux, so we find ourselves in a weird in-between state where drivers
may be slightly incompatible in weird ways.

## Ideas
Kernel wakeup-sources report an epoll from system_server to be the reason:
```
wakeup_source_activate: epoll_system_server_file:[timerfd7_system_server] state=0x13680002
```
(To have the wakeup_source formatted not only as `eventpoll`, cherry-pick this
commit: [fs: Improve eventpoll logging to stop indicting timerfd][timerfd-commit])

system_server(a.k.a. SystemServer in frameworks/base) seems to set
alarms(`CLOCK_REALTIME_ALARM` and `CLOCK_BOOTTIME_ALARM`) that have the
capability to wake up the system only in `AlarmManagerService`.

```
diff --git a/services/core/jni/com_android_server_AlarmManagerService.cpp b/services/core/jni/com_android_server_AlarmManagerService.cpp
index 47350c11f95..8da96d1e9a2 100644
--- a/services/core/jni/com_android_server_AlarmManagerService.cpp
+++ b/services/core/jni/com_android_server_AlarmManagerService.cpp
@@ -60,6 +60,7 @@ static constexpr int ANDROID_ALARM_TIME_CHANGE_MASK = 1 << 16;
  */
 static const size_t ANDROID_ALARM_TYPE_COUNT = 5;
 static const size_t N_ANDROID_TIMERFDS = ANDROID_ALARM_TYPE_COUNT + 1;
+/* typedef int clockid_t */
 static const clockid_t android_alarm_to_clockid[N_ANDROID_TIMERFDS] = {
     CLOCK_REALTIME_ALARM,
     CLOCK_REALTIME,
@@ -115,6 +116,8 @@ int AlarmImpl::set(int type, struct timespec *ts)
     memset(&spec, 0, sizeof(spec));
     memcpy(&spec.it_value, ts, sizeof(spec.it_value));
 
+    ALOGE("AlarmManagerService: Setting alarm (type %d) for %lu ns", type, ts->tv_nsec);
+
     return timerfd_settime(fds[type], TFD_TIMER_ABSTIME, &spec, NULL);
 }
 
@@ -184,6 +187,7 @@ int AlarmImpl::waitForAlarm()
         uint64_t unused;
         ssize_t err = read(fds[alarm_idx], &unused, sizeof(unused));
         if (err < 0) {
+            /* ANDROID_ALARM_TYPE_COUNT = CLOCK_REALTIME_ALARM*/
             if (alarm_idx == ANDROID_ALARM_TYPE_COUNT && errno == ECANCELED) {
                 result |= ANDROID_ALARM_TIME_CHANGE_MASK;
             } else {
@@ -191,6 +195,10 @@ int AlarmImpl::waitForAlarm()
             }
         } else {
             result |= (1 << alarm_idx);
+            if ((int)alarm_idx == CLOCK_REALTIME_ALARM ||
+                (int)alarm_idx == CLOCK_BOOTTIME_ALARM) {
+                ALOGE("AlarmManagerService: woke up from alarm %d", alarm_idx);
+            }
         }
     }
 
@@ -337,6 +345,7 @@ static jlong android_server_AlarmManagerService_init(JNIEnv*, jobject)
     }
 
     for (size_t i = 0; i < fds.size(); i++) {
+        /* timerfd_create() creates a timerfd object and returns the fd */
         fds[i] = timerfd_create(android_alarm_to_clockid[i], 0);
         if (fds[i] < 0) {
             log_timerfd_create_error(android_alarm_to_clockid[i]);
@@ -346,6 +355,7 @@ static jlong android_server_AlarmManagerService_init(JNIEnv*, jobject)
             }
             return 0;
         }
+        ALOGE("AlarmManagerService: fds[%d] points to fd %d", (int)i, fds[i]);
     }
 
     AlarmImpl *ret = new AlarmImpl(fds, epollfd, wall_clock_rtc());
@@ -368,6 +378,7 @@ static jlong android_server_AlarmManagerService_init(JNIEnv*, jobject)
     /* 0 = disarmed; the timerfd doesn't need to be armed to get
        RTC change notifications, just set up as cancelable */
 
+    /* Seems this is just for testing that settime works */
     int err = timerfd_settime(fds[ANDROID_ALARM_TYPE_COUNT],
             TFD_TIMER_ABSTIME | TFD_TIMER_CANCEL_ON_SET, &spec, NULL);
     if (err < 0) {
```

However, looking at the log statements, it seems no alarms(type 5 or 6) are
being set, only timers of type 2 and 3.

Next idea would be `min_futurity`, `listener_timeout`, `allow_while_idle_short_time`
All of these are set to 5 seconds. So test increasing them to 10 seconds:
```
diff --git a/services/core/java/com/android/server/AlarmManagerService.java b/services/core/java/com/android/server/AlarmManagerService.java
index 20bb7fb4789..76bf5de75e4 100644
--- a/services/core/java/com/android/server/AlarmManagerService.java
+++ b/services/core/java/com/android/server/AlarmManagerService.java
@@ -297,13 +297,13 @@ class AlarmManagerService extends SystemService {
                 "standby_never_delay",
         };
 
-        private static final long DEFAULT_MIN_FUTURITY = 5 * 1000;
+        private static final long DEFAULT_MIN_FUTURITY = 10 * 1000;
         private static final long DEFAULT_MIN_INTERVAL = 60 * 1000;
         private static final long DEFAULT_MAX_INTERVAL = 365 * DateUtils.DAY_IN_MILLIS;
         private static final long DEFAULT_ALLOW_WHILE_IDLE_SHORT_TIME = DEFAULT_MIN_FUTURITY;
         private static final long DEFAULT_ALLOW_WHILE_IDLE_LONG_TIME = 9*60*1000;
         private static final long DEFAULT_ALLOW_WHILE_IDLE_WHITELIST_DURATION = 10*1000;
-        private static final long DEFAULT_LISTENER_TIMEOUT = 5 * 1000;
+        private static final long DEFAULT_LISTENER_TIMEOUT = 10 * 1000;
         private final long[] DEFAULT_APP_STANDBY_DELAYS = {
                 0,                       // Active
                 6 * 60_000,              // Working
```
Unfortunately, the suspend still only last 5 seconds, so this was a dead end.

---

To be continued in ["Android Kernel Suspend and Tracing"]({{< relref "tracing.md" >}}).

[timerfd-commit]: (https://github.com/sonyxperiadev/kernel/commit/c1c412fc03c9817dac8cae3353bce1c32af3196a)
