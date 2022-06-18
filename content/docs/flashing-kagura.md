---
title: "Flashing AOSP on Xperia XZ"
description: "Definitve guide"
date: 2018-10-19T22:26:47+02:00
weight: 10
draft: false
bref: "Definitve guide"
slug: flashing-aosp-on-xperia-xz
toc: true
---

<div class="message warning">
The builds for the Xperia XZ will no longer be updated. This guide is severely
out of date (as of 2022). The builds should still work, but proceed with caution.
</div>

Follow these steps closely. Do not leave out a step and do them in the correct order.

*A big thanks to [StaticGTF](https://forum.xda-developers.com/member.php?u=3928215)
for writing large parts of this guide!*

## TL;DR
For people who know what they are doing:

1. Flash latest stock image, then unlock bootloader
2. Download v9 binaries from [developer.sony.com](https://developer.sony.com/file/download/software-binaries-for-aosp-pie-android-9-0-kernel-4-9-tone/),
   Flash binaries image via fastboot to `oem` partition
4. Flash this [tweaked TWRP recovery](https://sx.ix5.org/files/builds/kagura/misc/twrp-3.2.1-0-kagura-ab-x.img), credits to Artem Labazov(@ab123321)
3. Flash latest build from [sx.ix5.org](https://sx.ix5.org/files/builds/kagura/aosp-10/) from inside TWRP

That's it.

## Detailed Steps

### 1. Computer Setup
In order to install this ROM, you will need a working computer environment to
install the latest FTFs, recoveries and the OEM binaries.

There are numerous other guides out there on how to set up fastboot, but this
should be enough for most.

**On Windows:** Download and install the following components to your computer:

- Minimal ADB and Fastboot from
  [xda: Minimal ADB and Fastboot](https://forum.xda-developers.com/showthread.php?t=2317790)
- Androxyde's FlashTool from:
  [xda: FlashTool][flashtool-xda]

You will also need to install these drivers for your phone:

- Regular Xperia XZ driver
- Flashmode Driver
- Fastboot (S1) Driver

The drivers are located in the FlashTool installation folder. You will probably
need to reboot your computer with disabled driver signature verification in
order to install these drivers.

**On Linux:**
Download Androxyde's FlashTool from: [xda: FlashTool][flashtool-xda].

Then, install the `fastboot` and `adb` tools via your distro's package manager.

- On Ubuntu: `apt install android-tools-adb android-tools-fastboot`
- On Arch Linux: `pacman -Syu android-tools`.

### 2. Download necessary files

Make sure you have all the files needed before starting this process. Best
proceedings practice should mean that you've got all images located in a folder
where you can easily flash them to the phone with least amount of typing (same
folder as fastboot.exe).

- Download the `.FTF` file for the latest stock firmware.
  It should should be something ending in `.184` or `.192` for your region.
  See this [list of latest stock firmware files](https://forum.xda-developers.com/xperia-xz/how-to/xperia-xz-roll-android-7-0-nougat-39-2-t3510600).
  You can also use the tool `XperiFirm` for downloading, which is included with
  `FlashTool`.
- Download this [tweaked TWRP recovery](https://sx.ix5.org/files/builds/kagura/misc/twrp-3.2.1-0-kagura-ab-x.img), credits to Artem Labazov(@ab123321)
- Download Sony OEM binaries for `tone` on Android Pie, Kernel 4.9 from the
  [Sony Software Binaries page](https://developer.sony.com/file/download/software-binaries-for-aosp-pie-android-9-0-kernel-4-9-tone/).  
  The filename will be
  `SW_binaries_for_Xperia_Android_9.0_2.3.2_v$VERSION_tone.zip`, where
  `$VERSION` is currently v9.
- Download the latest AOSP 10 build from
  [sx.ix5.org](https://sx.ix5.org/files/builds/kagura/aosp-10/)
  <!-- The latest semi-stable build is 2018-11-16(there are no *stable* builds yet). -->
  <!-- It will be marked in [this xda thread](https://forum.xda-developers.com/xperia-xz/development/xz-aosp-pie-builds-t3864985/post78111505) as well. -->
- Optional: Download the latest stable Magisk from
  [xda: Magisk](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445)
- Optional: Download OpenGApps from
  [opengapps.org](https://opengapps.org/?arch=arm64&api=10.0&variant=pico).
  You need to chose Android 10.0, ARM64. You can choose any size variant, I would
  recommend the `pico` one because it has the least bloat.

### 3. Unlock Bootloader

You need an [unlocked bootloader](https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader).
**Read carefully what this means**.

<div class="message">
If your bootloader is already unlocked, you can skip ahead to the next step.
</div>

<div class="message warning">
  <h5>Warning!</h5>
  <a href="https://forum.xda-developers.com/crossdevice-dev/sony/universal-dirtycow-based-ta-backup-t3514236">
  Back up your Trim Area("TA") partition
  </a> before unlocking the booatloader. You cannot undo an unlock and your
  DRM keys, possibly along with your warranty, will be lost forever!
</div>

If you are on a locked bootloader you will need to unlock your bootloader using this page:
[Sony Developer - Unlock bootloader](https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader)

### 4. Flash latest stock firmware

The following steps are done from your computer with your device attached via USB-cable.

<div class="message">
If you have previously upgraded to the latest stock firmware(<code>.192</code>)
and only tested out e.g. OmniROM, there is no need to re-flash the stock
firmware and you can skip this step.
</div>

You can access S1 flash mode by powering down the device completely, then holding
<kbd>VOLUME-DOWN</kbd> while connecting the USB cable. The notification LED
should turn green and you can stop holding the volume button.

Flash the latest available Oreo .FTF stock firmware using Androxyde's
[FlashTool](https://forum.xda-developers.com/showthread.php?t=920746).  
(Guide for flashing FTF with FlashTool: [xda: How to use Flashtool](https://forum.xda-developers.com/showthread.php?t=2658952))


### 5. Flash TWRP Recovery
You can access fastboot mode by powering down the device completely, then holding
<kbd>VOLUME-UP</kbd> while connecting the USB cable. The notification LED should
turn blue and you can stop holding the volume button.

Flash the TWRP recovery from the command line:
<pre><code>fastboot flash <span style="color:red">recovery</span> twrp-3.2.1-0-kagura-ab-x.img</code></pre>
(You need to use this tweaked recovery because the official TWRP builds can not mount /data or handle encryption).

### 6. Flash OEM binaries
<!-- Unzip it and flash the resulting `.img` file with fastboot. -->

Extract the Sony OEM Binaries zip-archive that contains the image-file needed for the next step.
Flash Sony's OEM binaries with this CLI command: (Note that the destination differs between the recovery and oem!)
<pre><code>fastboot flash <span style="color:red">oem</span> SW_binaries_for_Xperia_Android_9.0_2.3.2_v3_tone.img</code></pre>
<!--
Download oem version 1 from the
[Sony Software Binaries Archives](https://developer.sony.com/file/download/software-binaries-for-aosp-pie-android-9-0-kernel-4-9-tone/)
-->

### 7. TWRP Recovery operations

Remove the USB cable from phone and check that it is completely powered down.
Then boot up your device to recovery by pressing and holding <kbd>VOLUME-DOWN</kbd>
and the <kbd>POWER</kbd> button at the same time.  When the white Sony logo
appears, release all buttons.

##### 7.1 Wipe data (optional, but highly recommended)

Wipe `/data`, `/cache` and Dalvik cache in TWRP:

- When booted into TWRP, swipe from left to right to enable system
  modifications.  Do not proceed with system partition marked as "Read only"!
- Click "Wipe" and choose to perform a factory reset. Swipe to proceed.
  (This will keep the files on your internal storage but delete all your app
  data).

<div class="message">
If you are coming from stock firmware, you absolutely need to do this. You might
be able to skip this step if you are upgrading from a previous Pie or Q build,
but if you run into problems you need to at least wipe the caches, if not
`/data` as well.
</div>

##### 7.2 Flash the AOSP ROM

<!-- You can access the TWRP recovery by powering down the device, then holding down -->
<!-- Volume-Down and the Power button.  Wait until the first vibration, release the -->
<!-- buttons, the TWRP logo will appear after about 15 seconds. -->

- When booted into TWRP, swipe from left to right to enable system
  modifications.  Do not proceed with system partition marked as "Read only"!
- Connect your phone to your computer via USB. The Phone should appear in your
  devices-list to the left in Windows Explorer.
- Transfer AOSP ROM zip-file to the phone's internal storage.
- Click "Install"-tile and select the AOSP ROM file
- Swipe from left to right to start the flashing procedure.

##### 7.3 Fix /dsp file labels
<div class="message">
This only for Android Pie builds. Android 10 does not need this fix any more.
</div>

If you ever installed an Android Oreo-based custom ROM - e.g. OmniROM 8.1 - your
SELinux file labels for the `dsp` partition will be wrong. For a more detailed
explanation and Download, see
[File relabeling for SODP]({{< ref "dsp-relabel.md" >}}).

Simply flash the DSP label fixer zip file in TWRP.

<!--
<div class="message">
<b>New:</b> Simply flash the
<a href="https://sx.ix5.org/files/builds/kagura/misc/dsp-label-fixer.zip">DSP label fixer</a>
zip file in TWRP.
</div>

To check whether your labels are wrong, mount the `dsp` partition inside TWRP
and type `ls -laZ /dsp/`.
If it shows files with the `qdsp_file` label, do this:
```
chcon -R u:object_r:adsprpcd_file:s0 /dsp/
```
Check again with `ls -lZ /dsp/` and confirm that all files are labeled
`adsprpcd_file`.
-->

<!-- 7.4 Fix /persist file labels and permissions -->

##### 7.4 Flash dualsim patcher (optional)
If you have a dualsim device, download the
[DualSim patcher]({{<ref "2020-01-10-dualsim-patcher.md" >}}).

- Transfer the DualSim patcher zip file to the phone's internal storage.
- Click "Install"-tile and select the DualSim patcher
- Swipe from left to right to start the flashing procedure.

##### 7.5 Flash OpenGapps (optional)
If you have problems afer flashing GApps, wipe all data and flash the ROM again,
without GApps.

- Transfer the OpenGApps zip file to the phone's internal storage.
- Click "Install"-tile and select the OpenGApps zip file
- Swipe from left to right to start the flashing procedure.

##### 7.6 Flash Magisk (optional)
Flash [Magisk v18.1 or later](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445).
Then install the Magisk Manager app. But read carefully which modules are
compatible!

- Transfer the Magisk zip file to the phone's internal storage.
- Click "Install"-tile and select the Magisk zip file
- Swipe from left to right to start the flashing procedure.

If you are using Magisk, your phone will restart once on first boot. This is
fully normal and is to be expected.

### 8. Reboot

Go back to the TWRP home screen and select "Reboot". If TWRP prompts
you to install its own app, decline and choose to restart normally.

Your phone will take a good while to finish booting up (~5 minutes or more).
Leave it alone until done.

<div class="message warning">
  <h5>Attention!</h5>
Please also read the
<a href="https://sx.ix5.org/bugs-sx">full bug list</a>!
</div>

## Optional Goodies
<!-- The included AOSP Camera cannot record video as of now. -->
You can try OpenCamera, Snap Camera(from LineageOS), or one of the numerous
[Google Camera ports](https://www.celsoazevedo.com/files/android/google-camera/).

## Troubleshooting

If your phone can't boot past the white Sony logo or freezes on Android
boot-animation, try to simultaneously press <kbd>POWER</kbd> + <kbd>VOLUME-UP</kbd>
until the first vibration. This will reboot your phone.

If you need to re-enter TWRP, your phone must first be powered off. You can
always force this no matter where and when by long-pressing
<kbd>POWER</kbd> + <kbd>VOL-UP</kbd> until you feel three vibrations. Then
release buttons and boot to TWRP with <kbd>POWER</kbd> + <kbd>VOLUME-DOWN</kbd>.

##### Magisk
If you cannot boot and have Magisk modules installed, it might be that they are
incompatible with this ROM. Before reporting a bug, please remove all Magisk
modules(by flashing the Magisk uninstall zip) and try booting again.

##### Encryption does not work
See "Formatting /data" below.

##### Formatting /data
If you still cannot boot the AOSP builds, try formatting your `/data` partition.

If you ever had encryption turned on while running the stock firmware, you also
might have to format your internal storage. This is because the stock firmware
seems to be using a different encryption version or algorithm than the AOSP
builds. The stock firmware runs encryption by default since Android Nougat.

<div class="message warning">
Back up all your photos, apps etc. to your computer as this will <b>wipe your
internal storage completely!</b>
</div>

<!--
To format your internal storage, download
[userdata.img](https://sx.ix5.org/files/builds/kagura/misc/userdata.img)
and flash it using fastboot:
<pre><code>fastboot flash <span style="color:red">userdata</span> userdata.img</code></pre>
-->

For 32Gb (single-SIM) devices, use this command:
```
fastboot format:ext4:0x58dffc000 userdata
```

For 64Gb (dual-SIM) devices, use:
```
fastboot format:ext4:0xcd5bfc000 userdata
```

You need to use the fastboot method because wiping in TWRP does not leave enough
space for the crypto footer (needs to be 32Mb), which you need to run Full Disk
Encryption (FDE).

Method for calculating needed hex values for `fastboot format`, including space
for the crypto footer:
```make
# Single-SIM:
# Reserve space for data encryption (23857201152-16384)
BOARD_USERDATAIMAGE_PARTITION_SIZE := 23857184768
# Dual-SIM:
# Reserve space for data encryption (55125737472-16384)
BOARD_USERDATAIMAGE_PARTITION_SIZE := 55125721088
```
Then use python to calculate the hex values:
```
$ python
hex(23857184768)
 '0x58dffc000'
hex(55125721088)
 '0xcd5bfc000'
```


---

*A big thanks to [StaticGTF](https://forum.xda-developers.com/member.php?u=3928215)
for writing large parts of this guide!*

[flashtool-xda]: https://forum.xda-developers.com/showthread.php?t=920746
