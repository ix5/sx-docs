+++
title = "Flashing AOSP on Xperia XZ"
description = "Definitve guide"
date = 2018-10-19T22:26:47+02:00
weight = 10
draft = false
bref = "Definitve guide"
toc = false
+++

Follow these steps closely. Do not leave out a step and do them in the correct order.

*A big thanks to [StaticGTF](https://forum.xda-developers.com/member.php?u=3928215)
for writing large parts of this guide!*

### 1. Computer Setup
In order to install this ROM, you will need a working computer environment to
install the latest FTFs, recoveries and the OEM binaries.

Download and install the following components to your computer:

- Minimal ADB and Fastboot from
  [xda: Minimal ADB and Fastboot](https://forum.xda-developers.com/showthread.php?t=2317790)
- Androxyde's FlashTool from:
  [xda: FlashTool](https://forum.xda-developers.com/showthread.php?t=920746)

You will also need to install these drivers for your phone:

- Regular Xperia XZ driver
- Flashmode Driver
- Fastboot (S1) Driver

The drivers are located in the FlashTool installation folder. You will probably
need to reboot your computer with disabled driver signature verification in
order to install these drivers.

### 2. Download necessary files

Make sure you have all the files needed before starting this process. Best
proceedings practice should mean that you've got all images located in a folder
where you can easily flash them to the phone with least amount of typing (same
folder as fastboot.exe).

- Download the `.FTF` file for the latest stock firmware.
  It should should be something ending in `.184` or `.192` for your region.
  See this [list of latest stock firmware files](https://forum.xda-developers.com/xperia-xz/how-to/xperia-xz-roll-android-7-0-nougat-39-2-t3510600).
- Download [AdrianDC's TWRP recovery](https://basketbuild.com/filedl/devs?dev=AdrianDC&dl=AdrianDC/Kagura/TWRP-Recovery/twrp-3.2.1-20171219-boot-kagura.img)
- Download Sony OEM binaries for `tone` on Android Pie, Kernel 4.9 from the
  [Sony Software Binaries page](https://developer.sony.com/develop/open-devices/downloads/software-binaries).  
  The filename will be `SW_binaries_for_Xperia_Android_9.0_2.3.2_v$VERSION_tone.zip`.
- Download the latest AOSP build from
  [sx.ix5.org](https://sx.ix5.org/files/builds/kagura/aosp/)
  <!-- The latest semi-stable build is 2018-11-16(there are no *stable* builds yet). -->
  <!-- It will be marked in [this xda thread](https://forum.xda-developers.com/xperia-xz/development/xz-aosp-pie-builds-t3864985/post78111505) as well. -->
- Download Magisk 17.3 from
  [xda: Magisk](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445)
- Download OpenGApps from
  [opengapps.org](https://opengapps.org/?arch=arm64&api=9.0&variant=pico).
  You need to chose `Android 9.0`, `ARM64`. You can choose any size variant, I would
  recommend the `pico` one because it has the least bloat.

### 3. Unlock Bootloader

You need an [unlocked bootloader](https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader).
**Read carefully what this means**.

<div class="message">
If your bootloader is already unlocked, you can skip ahead to the next step.
</div>

<div class="message warning">
  <h5>Warning!</h5>
  <a style="color: #1764de;" href="https://forum.xda-developers.com/crossdevice-dev/sony/universal-dirtycow-based-ta-backup-t3514236">
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
<pre><code>fastboot flash <span style="color:red">recovery</span> twrp-3.2.1-20171219-boot-kagura.img</code></pre>
(You need to use AdrianDC's one because the official TWRP builds can not mount /data).

### 6. Flash OEM binaries
<!-- Unzip it and flash the resulting `.img` file with fastboot. -->

Extract the Sony OEM Binaries zip-archive that contains the image-file needed for the next step.
Flash Sony's OEM binaries with this CLI command: (Note that the destination differs between the recovery and oem!)
<pre><code>fastboot flash <span style="color:red">oem</span> SW_binaries_for_Xperia_Android_9.0_2.3.2_v3_tone.img</code></pre>
<!--
Flash oem binaries version 1: Version 2 is buggy, and *calling does not work.*
Download oem version 1 from the
[Sony Software Binaries Archives](https://developer.sony.com/file/download/software-binaries-for-aosp-pie-android-9-0-kernel-4-9-tone-v1/)
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
  be able to skip this step if you are coming from a previous Pie build, but if
  you run into problems you need to at least wipe the caches, if not `/data` as
  well.
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

##### 7.3 Flash dualsim patcher (optional)
If you have a dualsim device, download the
[DualSim patcher for any custom ROM](https://forum.xda-developers.com/xperia-xz/how-to/f8332-dualsim-patcher-custom-rom-t3842672).

- Transfer the DualSim patcher zip file to the phone's internal storage.
- Click "Install"-tile and select the DualSim patcher
- Swipe from left to right to start the flashing procedure.

##### 7.4 Flash OpenGapps (optional)
If you have problems afer flashing GApps, wipe all data and flash the ROM again,
without GApps.

- Transfer the OpenGApps zip file to the phone's internal storage.
- Click "Install"-tile and select the OpenGApps zip file
- Swipe from left to right to start the flashing procedure.

##### 7.5 Flash Magisk (optional)
Flash [Magisk v17.1 or later](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445).
Then install the Magisk Manager app. But read carefully which modules are
compatible!

- Transfer the Magisk zip file to the phone's internal storage.
- Click "Install"-tile and select the Magisk zip file
- Swipe from left to right to start the flashing procedure.

### 8. Reboot

Go back to the TWRP home screen and select "Reboot". If TWRP prompts
you to install its own app, decline and choose to restart normally.

On first boot your phone will restart once. This is fully normal and is to be
expected. After the initial restart your phone will take a good while to finish
booting up (~5 minutes or more). Leave it alone until done.

## Optional Goodies
The included AOSP Camera cannot record video as of now. You can instead try
OpenCamera, Snap Camera(from LineageOS), or one of the numerous
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

##### Formatting /data
If you still cannot boot the AOSP builds, try formatting your `/data` partition.
Back up all your photos etc. to your computer as this will wipe your internal
storage completely.  

- Boot into TWRP recovery
- Check under the "Mount"-tile that the /data partition is mounted/checked.
  Otherwise click on /data to mount it. Return to "home screen" via the back
  button.
- Click "Wipe" and choose the lower-right button to format /data. Return to
  "home screen".

<div class="message warning">
  <h5>Attention!</h5>
Please also read the
<a style="color: #1764de;" href="https://sx.ix5.org/bugs-sx">full bug list</a>!
</div>

---

*A big thanks to [StaticGTF](https://forum.xda-developers.com/member.php?u=3928215)
for writing large parts of this guide!*
