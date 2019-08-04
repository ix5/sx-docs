---
title: "SailfishOS Installation on Xperia XZ"
description: ""
date: 2019-03-19T20:03:22+01:00
draft: false
---

## Preparation
Download the following files from [sx.ix5.org][sx-sailfish]:

- `2019-03-14-system.img` (save as `system.img`)
- `hybris-boot.img`
- `sfe-f8331-3.0.1.11-heroic.tar.bz2`

Then download these additional files:

- [AdrianDC's TWRP recovery][adrian-twrp] named
  `twrp-3.2.1-20171219-boot-kagura.img`
- [Software binaries version 16][swbins16] from Sony, named
  `SW_binaries_for_Xperia_Android_8.1.6.4_r1_v16_tone.zip` (you need to use
  version 16 because 17 has a problem with the sensors)

Install `fastboot` on your computer. Instructions can be found at
[developer.android.com][dev-fastboot]. An alternative installer can be found at
[xda: Minimal ADB and Fastboot][xda-minimal].

For accessing `fastboot` mode on the device, see [Accessing special
Modes][specialmodes].

## Install required Recovery

Install AdrianDC's TWRP recovery via:
```
fastboot flash recovery twrp-3.2.1-20171219-boot-kagura.img
```

## Install Software Binaries
Unzip the downloaded file so that you get
`SW_binaries_for_Xperia_Android_8.1.6.4_r1_v16_tone.img`.
Flash the image via `fastboot` to the `oem` partition:
```
fastboot flash oem SW_binaries_for_Xperia_Android_8.1.6.4_r1_v16_tone.img
```

## Flashing Android Parts
Flash `hybris-boot.img` via `fastboot`:
```
fastboot flash boot hybris-boot.img
```

Then flash the system image via `fastboot` as well:
```
fastboot flash system system.img
```

## Flashing Sailfish Parts
Push `sfe-f8331-3.0.1.11-heroic.tar.bz2` to `/sdcard/` on the device.

Then run these commands manually in the device's recovery, e.g. `TWRP`:
```
rm -rf /data/.stowaways/sailfishos/
mkdir -p /data/.stowaways/sailfishos
tar --numeric-owner \
    -xvjf /sdcard/sfe-f8331-3.0.1.11-heroic.tar.bz2 \
    -C /data/.stowaways/sailfishos
rm /sdcard/sfe-f8331-3.0.1.11-heroic.tar.bz2
```

Reboot and you should be done.

## Note about going back to Pie ROMs
Since this SailfishOS port is based on Android Oreo, when you want to go back to
a Pie ROM after flashing SailfishOs, you need to either flash the stock firmware
for the `dsp` partition via FlashTool/Emma or manually re-label the files in
`/dsp` to `adsprpcd_file`.

The exact reasons and procedure are described in
[DSP File Relabeling for SODP][relabel].

[sx-sailfish]: https://sx.ix5.org/files/sailfish/
[adrian-twrp]: https://basketbuild.com/filedl/devs?dev=AdrianDC&dl=AdrianDC/Kagura/TWRP-Recovery/twrp-3.2.1-20171219-boot-kagura.img
[swbins16]: https://developer.sony.com/file/download/software-binaries-for-aosp-oreo-android-8-1-kernel-4-4-tone-v16/
[dev-fastboot]: https://developer.android.com/studio/releases/platform-tools.html#downloads
[xda-minimal]: https://forum.xda-developers.com/showthread.php?t=2317790
[specialmodes]: /info/sony-devices/#accessing-special-modes
[relabel]: /info/post/dsp-file-relabling-for-sodp/
