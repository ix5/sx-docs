---
title: "Airplane Mode: Kill Cell only"
description: ""
date: 2020-05-17T20:47:24+02:00
draft: false
---

You can change Android's behaviour when you toggle airplane mode via adb
commands.

Check current status:
```
adb shell settings get global airplane_mode_radios
# -> cell,bluetooth,wifi,nfc,wimax
adb shell content query --uri content://settings/global  --projection name:value --where "name='airplane_mode_radios'"
```

The list of `airplane_mode_radios` defines which "radios" will get turned off
when airplane mode is turned on.

By removing from that list, you can keep the omitted radios on.

```
# Keep Wi-Fi on when turning on airplane mode:
adb shell settings put global airplane_mode_radios cell,bluetooth,nfc,wimax
# Keep both Wi-Fi and bluetooth on:
adb shell settings put global airplane_mode_radios cell,nfc,wimax

# Revert to default:
adb shell settings delete global airplane_mode_radios
```

> There’s also a way to stop a device from turning on one of these radios when
> Airplane Mode has been enabled. 
> By default, the options given to this command are WiFi, Bluetooth, and NFC.
```
adb shell settings put global airplane_mode_toggleable_radios bluetooth,nfc

# Revert to default:
adb shell settings delete global airplane_mode_toggleable_radios
```

**Sources:** See [this answer][androidstackexchange], [xda][xda].

[androidstackexchange]: https://android.stackexchange.com/questions/59664/possible-to-turn-on-airplane-mode-with-wifi-on-only
[xda]: https://www.xda-developers.com/customize-radios-airplane-mode-android/
