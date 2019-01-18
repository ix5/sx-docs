+++
title = "Power Management"
description = ""
date = 2018-12-22T18:07:04+01:00
draft = true
+++

# Debugging stuff

```
adb shell dumpsys battery unplug
adb shell dumpsys deviceidle step
```

https://stackoverflow.com/questions/3417308/force-an-android-phone-to-sleep-in-order-to-test

`echo 1 > /sys/power/pm_print_times`

`adb connect xxxx; adb wait-for-device; adb shell "echo mem > /sys/power/state"`
