+++
title = "Flashing AOSP on Xperia XZ"
description = "Rough guide"
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
  <a style="color: #3794de;" href="https://forum.xda-developers.com/crossdevice-dev/sony/universal-dirtycow-based-ta-backup-t3514236">
  Back up your Trim Area("TA") partition
  </a> before unlocking the booatloader. You cannot undo an unlock and your
  DRM keys, possibly along with your warranty, will be lost forever!
</div>

### 1. Flash latest stock firmware
Flash latest stock Oreo .ftf ([see list](https://forum.xda-developers.com/xperia-xz/how-to/xperia-xz-roll-android-7-0-nougat-39-2-t3510600))
via [FlashTool](https://forum.xda-developers.com/showthread.php?t=920746).
You can access flashmode by powering down the device, then holding Volume-Down
and then connecting the USB cable.
<div class="message">
If you have previously upgraded to the latest stock firmware(<code>.192</code>)
and only tested out e.g. OmniROM, there is no need to re-flash the stock
firmware and you can skip this step.
</div>

### 2. Flash OEM binaries
Flash oem binaries version 3.
Download them from the
[Sony Software Binaries page](https://developer.sony.com/develop/open-devices/downloads/software-binaries)
The filename will be `SW_binaries_for_Xperia_Android_9.0_2.3.2_v3_tone.zip`.
Unzip it and flash the resulting `.img` file with fastboot.
<!--
Flash oem binaries version 1: Version 2 is buggy, and *calling does not work.*
Download oem version 1 from the
[Sony Software Binaries Archives](https://developer.sony.com/file/download/software-binaries-for-aosp-pie-android-9-0-kernel-4-9-tone-v1/)
-->

### 3. Flash TWRP Recovery
Flash TWRP ([AdrianDC's one](https://basketbuild.com/filedl/devs?dev=AdrianDC&dl=AdrianDC/Kagura/TWRP-Recovery/twrp-3.2.1-20171219-boot-kagura.img)).
```
fastboot flash recovery twrp-3.2.1-20171219-boot-kagura.img
```
(You need to use AdrianDC's one because the official TWRP builds can not mount /data).

### 4. Download latest (semi-stable) build
Get the latest build from [sx.ix5.org](https://sx.ix5.org/files/builds/kagura/aosp).
The latest semi-stable build is 2018-11-16(there are no *stable* builds yet).
It will be marked in [this xda thread](https://forum.xda-developers.com/xperia-xz/development/xz-aosp-pie-builds-t3864985/post78111505)
as well.

### 5. Flash the ROM zip file from TWRP
You can access the TWRP recovery by powering down the device, then holding down
Volume-Down and the Power button.  Wait until the first vibration, release the
buttons, the TWRP logo will appear after about 15 seconds.

### 6. Wipe data (optional, but highly recommended)
Wipe `/data`, `/cache` and Dalvik cache in TWRP. If you are coming from stock
firmware, you absolutely need to do this. You might be able to skip this step if
you are coming from a previous Pie build, but if you run into problems you need
to at least wipe the caches, if not `/data` as well.

### 7. Flash dualsim patcher from TWRP (optional)
If you have a dualsim device, use
[DualSim patcher for any custom ROM](https://forum.xda-developers.com/xperia-xz/how-to/f8332-dualsim-patcher-custom-rom-t3842672).

### 8. OpenGapps (optional)
You need `Android 9`, `ARM64`.  You can choose any size variant, I would
recommend
[2018-10-16 pico](https://github.com/opengapps/arm64/releases/download/20181016/open_gapps-arm64-9.0-pico-20181016.zip)  
If you have problems afer flashing GApps, wipe all data and flash the ROM again,
without GApps.

### 9. Magisk (optional)
Flash [Magisk v17.1 or later](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445).
Then install Magisk Manager app. But read carefully which modules are
compatible!

## Optional Goodies
The included AOSP Camera cannot record video as of now. You can instead try
OpenCamera, Snap Camera(from LineageOS), or one of the numerous
[Google Camera ports](https://www.celsoazevedo.com/files/android/google-camera/).

## Caveats
**Adding fingerprint crashes device:** You need to tap your finger onto the
scanner once during the initial "How-to" screen, then *wait for 3 animations*
when the "Lift, then touch again" screen is shown. Then put your finger on the
scanner and let it get scanned like you normally would.

<div class="message warning">
  <h5>Attention!</h5>
Please also read the
<a style="color: #3794de;" href="https://etherpad.net/p/r.2c613f4824d5e9f7764e4c7c45972aac">full bug list</a>!
</div>
