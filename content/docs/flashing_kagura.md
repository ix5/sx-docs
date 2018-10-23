+++
title = "Flashing AOSP on Xperia XZ"
description = ""
date = 2018-10-19T22:26:47+02:00
weight = 20
draft = false
bref = "Rough guide"
toc = false
+++

Follow these steps closely. Do not leave out a step and do them in the correct order.

You need an [unlocked bootloader](https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader).
**Read carefully what this means**.

<div class="message warning">
  <h5>Warning!</h5>
  <a href="https://forum.xda-developers.com/crossdevice-dev/sony/universal-dirtycow-based-ta-backup-t3514236">
  Back up your Trim Area("TA") partition
  </a> before unlocking the booatloader. You cannot undo an unlock and your
  DRM keys, possibly along with your warranty, will be lost forever!
</div>

**Flash latest stock Oreo .ftf** ([see list](https://forum.xda-developers.com/xperia-xz/how-to/xperia-xz-roll-android-7-0-nougat-39-2-t3510600)) via [FlashTool](https://forum.xda-developers.com/showthread.php?t=920746). You can access flashmode by powering down the device, then holding Volume-Down and then connecting the USB cable.

**Flash TWRP** ([AdrianDC's one](https://basketbuild.com/filedl/devs?dev=AdrianDC&dl=AdrianDC/Kagura/TWRP-Recovery/twrp-3.2.1-20171219-boot-kagura.img)).
```
fastboot flash recovery twrp-3.2.1-20171219-boot-kagura.img
```
(You need to use AdrianDC's one because the official TWRP builds can not mount /data).

**Get the latest build** from [sx.ix5.org](https://sx.ix5.org/files/builds/kagura/).

**Flash the ROM zip file from TWRP**. You can access the TWRP recovery by powering down the device, then holding down Volume-Down and the Power button. Wait until the first vibration, release the buttons, the TWRP logo will appear after about 15 seconds.

**Flash OpenGapps** (optional): You need the [2018-10-16 one](https://github.com/opengapps/arm64/releases/tag/20181016) (This is the last version I can confirm works, the latest GApps break because pixelsetup has insufficient permissions)
You can choose any variant, I would recommend [2018-10-16
pico](https://github.com/opengapps/arm64/releases/download/20181016/open_gapps-arm64-9.0-pico-20181016.zip)  
**Important:** You need to disable `DialerFramework` in your
[`gapps-config.txt`](https://github.com/opengapps/opengapps/wiki/Advanced-Features-and-Options) because it doesn't work
with the new oem v2 binaries.
```
Exclude
DialerFramework
```

**Flash [Magisk v17.1 or later](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445)** (optional)

**Install Magisk Manager app** (Optional). But read carefully which modules are compatible!

### Caveats
**Adding fingerprint crashes device:** You need to tap your finger onto the
scanner once during the initial "How-to" screen, then *wait for 3 animations*
when the "Lift, then touch again" screen is shown. Then put your finger on the
scanner and let it get scanned like you normally would.
