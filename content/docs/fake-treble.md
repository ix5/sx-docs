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

## How-To

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

<!--
Because the tone devices are still on Kernel 4.9, you also need to run
`q_4.9_repo_update.sh`.
-->

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

If you want to build a vendor-only OTA package, add this to
`customization-noproduct.mk`:
```
# Skip adding system to ota
PRODUCT_BUILD_SYSTEM_IMAGE := false
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
fastboot flash oem vendor.img
```
<!-- fastboot flash system system.img -->

<div class="message warning">
Do not flash Sony’s software binary image again now, all the needed blobs are
already included in <code>vendor.img</code> for you!
</div>

### OTA
To create a flashable zip file, use `make otapackage`. You can install the
resulting zip file in `out/target/product/kagura` via TWRP or any other
recovery.

```
adb push out/target/product/kagura/aosp_f8331-ota-eng*.zip /sdcard/
```

### Compatible GSIs
You need to use system-as-root, `arm64_ab` GSIs since this is now the default
for Android 10.

Use `fastboot flash system <my-gsi.img>`.

Since SELinux is in enforcing mode, badly-built GSIs will fail to run. You can
set it to permissive by changing the commandline:
```
BOARD_KERNEL_CMDLINE += androidboot.selinux=permissive
```
or set `BOARD_USE_ENFORCING_SELINUX := false` in `Customization.mk`.

### Going back to regular builds
You need to:

- Unset `PRODUCT_FAKE_TREBLE_BUILD` in `customization-noproduct.mk`
- Unset `PRODUCT_BUILD_SYSTEM_IMAGE` in `customization-noproduct.mk`
- Revert `dtsi: tone: conjure oem into /vendor` in `kernel/sony/msm-4.9/kernel` in
  case you want to build non-treble `tone` builds again

<!--
- Undo the `revert: liblights: Migrate to kernel 4.14 LED class for RGB
  tri-led` commit in `device/sony/common` in case you want to build for 4.14
  devices like the `tama` or `kumano` platform
-->

<div class="message">
After you've flashed your fake treble build, you need to re-flash the regular
Software Binaries to the <code>oem</code> partition if you want to return to
regular builds.
</div>

## Explanation

The fake treble methodology relies on some patches on top of the SODP repos.

### git scripts
Here is an example of picking git patches, see
[q_repo_update](https://git.ix5.org/felix/ix5_repo_update/src/commit/2134e86de59f0e4519428d82270715064365f7e6/q_repo_update.sh#L234-L247):

```

pushd $ANDROOT/device/sony/tone
LINK="https://git.ix5.org/felix/device-sony-tone"
(git remote --verbose | grep -q $LINK) || git remote add ix5 $LINK
do_if_online git fetch ix5

# git checkout 'disable-verity-no-forceencrypt'
# Change forceencrypt to encryptable for userdata
apply_commit af592265685fddf24100cbc1fdcdcb5bfd2260c1
# Disable dm-verity
apply_commit b611c8d91a374f246be393d89f20bbf3fc2ab9f7

# git checkout 'treble-odm-3'
# Use oem as /vendor
apply_commit 796ff85b93d28a301cce7bd6b3e0852a35180e04

popd
```

This piece of code could be simplified as:
```
cd device/sony/tone

git fetch https://git.ix5.org/felix/device-sony-tone \
  'disable-verity-no-forceencrypt'
# Change forceencrypt to encryptable for userdata
git cherry-pick af592265685fddf24100cbc1fdcdcb5bfd2260c1
# Disable dm-verity
git cherry-pick b611c8d91a374f246be393d89f20bbf3fc2ab9f7

git fetch https://git.ix5.org/felix/device-sony-tone \
   'treble-odm-3'\
# Use oem as /vendor
git cherry-pick 796ff85b93d28a301cce7bd6b3e0852a35180e04
```

So it enters the `device/sony/tone` repository, fetches several branches from
the ix5 git repos and applies the needed commits.

For example, the treble patches for tone devices:
[branch](https://git.ix5.org/felix/device-sony-tone/commits/branch/treble-odm-3) -
[Use oem as vendor](https://git.ix5.org/felix/device-sony-tone/commit/796ff85b93d28a301cce7bd6b3e0852a35180e04).

This patch modifies the `fstab` file used to mount partitions by Android, sets
the size of the fake `vendor` partition to the size of the real `oem` partition,
sets `TARGET_COPY_OUT_VENDOR` and checks whether `PRODUCT_FAKE_TREBLE_BUILD` is
set.

### Another example

Same for the needed `dtsi` changes:
[treble_repo_update](https://git.ix5.org/felix/ix5_repo_update/src/commit/2134e86de59f0e4519428d82270715064365f7e6/treble_repo_update.sh#L76) -
[dtsi: Conjure oem into vendor](https://git.ix5.org/felix/ix5_repo_update/src/commit/2134e86de59f0e4519428d82270715064365f7e6/patches/dtsi-tone-conjure-oem-into-vendor.patch).

This changes the early-mount arguments so that the kernel treats the `oem`
partition correctly on early bootup.

### More

This same methodology applies to the rest of the needed commits. Should they be
out of date, please fetch them, resolve the git conflicts yourself, and maybe
send a pull request so that others might benefit.

## Not a Lawyer

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
