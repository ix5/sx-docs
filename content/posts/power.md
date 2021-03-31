---
title: "QuicK: Debugging Power Management"
date: 2021-03-31
draft: false
---

Quick reference over battery-related debugging, mainly for myself.

### adb

```
adb shell dumpsys battery unplug
adb shell dumpsys deviceidle step
```

Link: StackOverflow: [Force Android to sleep](https://stackoverflow.com/questions/3417308/force-an-android-phone-to-sleep-in-order-to-test)

### Kernel

```sh
echo 1 > /sys/power/pm_print_times
```

```sh
adb connect xxxx
adb wait-for-device
adb shell "echo mem > /sys/power/state"
```
