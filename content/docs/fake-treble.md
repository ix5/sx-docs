---
title: "Fake Treble for Xperia XZ"
description: "How to build GSI support on tone devices"
date: 2019-12-20T16:55:47+01:00
weight: 18
draft: false
bref: "How to build GSI support on tone devices"
toc: true
---

This guide should help you create builds that can run Generic System Images
(GSIs) for the Xperia XZ, even though it was never shipped with a `/vendor`
partition.

We will move the "Software Binaries", as Sony calls them, from the `/odm`
mountpoint to `/vendor/odm` and move the `vendor` part from the `system`
partition (`/system/vendor`) to the `oem` partition.

**Pre:**
```
Partition   Mountpoint
system      /system
             ↳/vendor
oem         /odm
```

**Post:**
```
Partition   Mountpoint
system      /system
oem         /vendor
             ↳/odm
```

Most steps in the [regular Android build guide](../building-android/) and the
[official Sony build guide][sony-buildguide]
apply here as well.

### local_manifests

Clone my own `local_manifests` instead of Sony’s:
```
git clone https://git.ix5.org/felix/local-manifests-ix5 \
  local_manifests -b 'ix5-customizations-10'
```

### repo_update
After you run `repo_update.sh`, also run `q_repo_update.sh`. Then, run
`treble_repo_update.sh`. These scripts will fetch all the relevant patches you
need to create a fake treble build.

Because the tone devices are still on Kernel 4.9, you also need to run
`q_4.9_repo_update.sh`.

You will also need to download the Android 9, Kernel 4.9 Software Binaries. Do
not use the Kernel 4.14 binaries!

### device-sony-customization
Then, create the folder `device/sony/customization` inside your build
environment.

It needs to contain two files:

`Customization.mk:` (note the uppercase `C`!)
```
BUILD_KERNEL := false
# Include buildvar whether to build a fake treble build or not
include device/sony/customization/customization-noproduct.mk
```

`customization-noproduct.mk:`
```
# Fake treble builds for kagura
PRODUCT_FAKE_TREBLE_BUILD := true
```

### device-sony-odm
Clone `https://git.ix5.org/felix/device-sony-odm` into `device/sony/odm`.

Then, you need to extract the "Software Binaries" - "blobs" for short - from the
official images.

When you download an image from the
[Software Binaries page][swbins],
you will receive a `.zip` archive, which contains a `.img` file.
This `.img` file is a sparse android image[^source-sparse]:
```
$ file SW_binaries_for_Xperia_Android_10.0.7.1_r1_v2a_tama.img
 ↳ SW_binaries_for_Xperia_Android_10.0.7.1_r1_v2a_tama.img: \
   Android sparse image, version: 1.0, [...]
```

### Sparse Images & Loop Mount
You can un-sparse the image using the [simg2img][simg2img][^simg2img-intree] tool. Run
`/path/to/simg2img SW_<name>.img swbins-unsparse.img` and you will receive a
regular loop image of an ext4 file system.
```
$ file swbins-unsparse.img
 ↳ swbins-unsparse.img: Linux rev 1.0 ext4 filesystem data, \
   [..] volume name "odm" (extents) (large files) (huge files)
```

You can then mount that image using `loop`[^udisksctl]:
```
sudo mount -o loop swbins-unsparse.img /mnt/
```
Create `device/sony/odm/blobs/` and copy the contents of the mounted image
there:
```
cd device/sony/odm
mkdir blobs
sudo cp -r /mnt/* blobs/
# Delete ext4 metadata
sudo rm -rf blobs/lost+found/
# Don't forget the colon
sudo chown -R $(whoami): blobs/
```

### Build it!
First, because building the kernel on Android Q from inside the Android tree is
quite complicated, we use a script to build them instead.
```
cd kernel/sony/msm-4.9/common-kernel
```
Edit `build-kernels-gcc.sh` and uncomment/edit the `Override` lines so the section
looks like this
```
# Override:
TONE="kagura"
PLATFORMS="tone"
```
This will save you from building the kernel for *all* Sony devices and only
build for the Xperia XZ ("kagura").

Then, build the kernel with patched `dtb`:
```
bash build-kernels-gcc.sh
```
After this has finished, you should have a file named `kernel-dtb-kagura` in
your `kernel/sony/msm-4.9/common-kernel` folder.

Now, go back to your Android root dir and run the build commands:
```
source build/envsetup.sh && lunch aosp_f8331-userdebug
make bootimage systemimage vendorimage
```

In case your build stops at about 90% with this error, just re-start the
build[^metalava].
```
[ 93% 12745/13686] //frameworks/base:hiddenapi-lists-docs Metalava [common]
FAILED: out/soong/[...]hiddenapi-lists-docs-stubs.srcjar
```

Flash the resulting files in
`out/target/product/kagura/`:
```
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash oem vendor.img
```
<div class="message warning">
Do not flash Sony’s software binary image again now, all the needed blobs are
already included in <code>vendor.img</code> for you!
</div>

### Compatible GSIs
You need to use system-as-root, `arm64_ab` GSIs since this is now the default
for Android 10.

Since SELinux is in enforcing mode, badly-built GSIs will fail to run. You can
set it to permissive by changing the commandline:
```
BOARD_KERNEL_CMDLINE += androidboot.selinux=permissive
```
or set `BOARD_USE_ENFORCING_SELINUX := true` in `Customization.mk`.

### Going back to regular builds
You need to unset `PRODUCT_FAKE_TREBLE_BUILD` in `customization-noproduct.mk`
and:
- Undo the `revert: liblights: Migrate to kernel 4.14 LED class for RGB
  tri-led` commit in `device/sony/common` in case you want to build for 4.14
  devices like the `tama` or `kumano` platform
- Undo `dtsi: tone: conjure oem into /vendor` in `kernel/sony/msm-4.9/kernel` in
  case you want to build non-treble `tone` builds again

<div class="message">
After you've flashed your fake treble build, you need to re-flash the regular
Software Binaries to the <code>oem</code> partition if you want to return to
regular builds.
</div>

### Not a Lawyer

You should decide for youself whether this approach falls within the purview of
the blobs EULA. In the EU, you might have some more rights than in other areas
so some passages of the following might be void, but I've highlighted the
relevant parts of the EULA below:

**2. END USER RIGHTS AND USE.**
> Sony Mobile grants You non-exclusive, non-transferable end-user copyright
> rights to download the Software on the local hard disk(s) or other permanent
> storage media of one computer and install the Software, by using the Fastboot
> Protocol, on a single Sony Mobile device with an unlocked boot loader at a
> time. Except as provided by any Open Source License Terms, You may use the
> Software only for the sole purpose of testing and validating Your own
> applications and/or content (“Purpose”). [...]

**3. LIMITATIONS ON END USER RIGHTS.**
> a) You may not use (other than as expressly permitted for the Purpose),
> modify, translate, reproduce, or transfer the right to use the Software or
> copy the Software. You may not use the Software for any reason other than the
> Purpose.


[^source-sparse]: For more info, see
  [source.android.com](https://source.android.com/devices/bootloader/partitions-images#sparse-image-format)
  or search for "Sparse Image".
[^simg2img-intree]: You should already have the tool in
  `out/host/linux-x86/bin/simg2img` as well.
[^udisksctl]: You also use `udisksctl` to mount the image:
  `udisksctl loop-setup -r -f swbins-unsparse.img`
[^metalava]: Android's metalava takes up enormous amounts of RAM. It attempts to
  verify the whole Android API at once and is badly designed.

[sony-buildguide]: https://developer.sony.com/develop/open-devices/guides/aosp-build-instructions/build-aosp-android-android-10-0-0
[swbins]: https://developer.sony.com/develop/open-devices/downloads/software-binaries
[simg2img]: https://github.com/anestisb/android-simg2img
