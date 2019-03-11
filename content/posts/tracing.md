---
title: "Android Kernel Suspend Debugging and Tracing"
date: 2019-01-25T08:18:00+01:00
bref: "Some notes, and a short primer on the tracing system"
draft: false
author: Felix
---

While [debugging suspend]({{< relref "suspend.md" >}}) on the Sony Xperia XZ, 
it turned out that the Android kernel has some great tracing function built in.
Pretty much any event available in `/d/tracing/event/*` can be traced, logged
and analyzed.

We wanted to know the source of wakeups, and also investigate whether the device
might have been awoken by hardware interrupts("IRQs").

**You need to be root for the following commands.**

To turn on tracing for wakeup sources:
```
echo 1 > /d/tracing/events/power/wakeup_source_activate/enable
```
To trace interrupts:
```
echo 1 > /d/tracing/events/irq/irq_handler_entry/enable
```
But also filter out some noise:
```
# filter out arch_timer irqs, (qcom,smd-rpm, qcom,glink-smem-native-xprt-rpm):
echo "irq != 5 && irq != 7 && irq != 131" > /d/tracing/events/irq/irq_handler_entry/filter
```
Timers might also be of interest:
```
echo 1 > /d/tracing/events/timer/timer_expire_entry/enable
echo 1 > /d/tracing/events/timer/timer_start/enable
echo "function != calculate_load_timer && function != process_timeout" \
  > /d/tracing/events/timer/timer_start/filter
```

**To start the actual tracing:**
```
echo 1 > /d/tracing/tracing_on
```

Then pull the resulting trace, easiest is to use adb and analyze on the computer:
```
adb shell cat /d/tracing/trace
```
To stop tracing again, write a `0` into `/d/tracing/tracing_on`

## Analysis

After letting the device idle and try to enter deep sleep, some culprits quickly showed up.

First was `dev_pn547`, which is the NFC driver in use. Disabling it in the
kernel defconfig was done via:
```
-CONFIG_NFC_PN547=y
-CONFIG_NFC_PN547_PMC_CLK_REQ=y
+CONFIG_NFC_PN547=n
+CONFIG_NFC_PN547_PMC_CLK_REQ=n
```

The next suspect was the mmc driver for the internal sdcard. It turned out the
tone platform was the only one not to have `CONFIG_MMC_BLOCK_DEFERRED_RESUME=y`,
while both loire and yoshino had it set.

Sadly, both of those ideas lead to nowhere.

Next, we tried to find out what function might be waking up the kernel, since
there was still a lot of noise(we're talking about logs 20MB big in only a
matter of minutes).

We tried inserting `WARN_ON(1);` at various places, e.g. in `__pm_stay_awake`,
but it turns out this function is called so often that the resulting dmesg is
being filled in a matter of seconds and thus completely useless for analysis.

Our next hunch was to disallow the creation of alarms via eventpoll, but
preventing `ep_wakeup_source_register` in the kernel meant the device would
crash too fast to yield any useful results.

So next we tried preventing all wakeup sources created by eventpoll from waking
up the device:

```
drivers/base/power/wakeup.c
static void wakeup_source_activate(struct wakeup_source *ws)
[...]
	if (!strncmp("epoll_", ws->name, 6)) {
		pr_err("%s: skipping ws %s\n", __func__, ws->name);
		return;
	}
```
But since the `wakeup_source_activate` function is called only when the device
is already awake, this is already too late to prevent the wakeup[^1], and adding
`WARN_ON` to trace the calling function yet again yielded too much noise to
investigate.

## Wrapping up
Even though we didn't find the root cause for the weird suspend bug, we learned
a lot about the tracing system, which is really neat.

For more info, check [Android source: ftrace][src-android-ftrace] and
[ftrace: trace your kernel functions!][jvns-ftrace].

### (Addendum)
Idea: `echo 1 > /sys/kernel/debug/clk/debug_suspend`

[^1]: See the [documentation in wakeup.c](https://github.com/sonyxperiadev/kernel/blob/aosp/LE.UM.2.3.2.r1.4/drivers/base/power/wakeup.c#L490-L525)
[src-android-ftrace]: https://source.android.com/devices/tech/debug/ftrace
[jvns-ftrace]: https://jvns.ca/blog/2017/03/19/getting-started-with-ftrace/
