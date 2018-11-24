+++
title = "Building Android"
description = "How to build AOSP for Xperia devices"
date = 2018-10-19T22:14:54+02:00
weight = 10
draft = false
bref = "How to build AOSP for Xperia devices"
toc = true
+++

This guide will asume you follow the
[official Sony build guide](https://developer.sony.com/develop/open-devices/guides/aosp-build-instructions/build-aosp-android-p-9-0-0),
but add some extra tips.

## Initialize your build directory
Create a `~/android/build-env` folder, `cd` into it and run the provided
`repo init` command (assuming you installed `repo` into `~/.local/bin/repo`).

## Ubuntu chroot

<div class="message success">
Since Google use Ubuntu as a reference build environment, it is advisable you use it as well to avoid incompatibility problems.
<b>If you're on Ubuntu already, you can skip this step.</b>
</div>

If you're on another system like Fedora or Arch Linux, you can set up a Ubuntu
chroot build environment instead.

A build chroot takes about 1-1.5GB of space and will save you from headaches,
e.g. when Android modules aren't compiled against your host glibc.

Install `debootstrap`, then run
```
debootstrap --variant=buildd --arch=amd64 bionic ~/ubuntu-android
```
chroot into the newly created builder system, either via `chroot` or just using
`systemd-nspawn` (Assuming you created your android build environment via `repo
init` in `~/android/build-env`):
```
sudo systemd-nspawn \
--bind=/home/<your-username>/android/build-env/:/home/builder/build-env/ \
  -D /home/<your-username>/dev/android/ubuntu-android

useradd -m -s /bin/bash builder
# -> No need to add a password, nspawn will log you in automatically
```

## Install required packages
```
apt install software-properties-common python
add-apt-repository universe
apt update
apt install bison g++-multilib git gperf libxml2-utils make zlib1g-dev zip liblz4-tool libncurses5
apt install openjdk-8-jdk
# Optional:
apt install ccache
apt install rsync libssl-dev aapt
# Kernel recompilation:
apt install bc autoconf automake autopoint autotools-dev bsdmainutils
gawk groff-base libarchive-zip-perl libpipeline1 libtimedate-perl libtool
# gettext gettext-base dh-autoreconf dh-strip-nondeterminism
```
Cross-reference the needed packages with the
[official Sony build guide](https://developer.sony.com/develop/open-devices/guides/aosp-build-instructions/build-aosp-android-p-9-0-0)
should this document be out of date.

*If you're building in a chroot:*
Exit out of the build system by typing `exit` (and pressing `Ctrl-]` three times
for `nspawn`), then re-launch your build system and log in as your builder user.
If you want to share your own ccache etc. with your build environment, `bind`
the appropriate directories.
```
sudo systemd-nspawn \
--bind=/home/<your-username>/android/build-env/:/home/builder/build-env/ \
  --bind=/home/<your-username>/.ccache/:/home/builder/.ccache/ \
  --bind=/home/<your-username>/.local/bin/repo:/usr/local/bin/repo \
  --bind=/home/<your-username>/.repoconfig/:/home/builder/.repoconfig/ \
  -D /home/<your-username>/ubuntu-android \
  --user=builder
```
(Assuming you installed `repo` into `~/.local/bin/repo`).

## Applying needed patches

Clone the Sony `local_manifests` git repo as instructed and run
`repo_update.sh`. Your build directory will now be ready for a first build.

<div class="message focus">
<b>Felix:</b> If you want to recreate my own builds, use
<code>
git clone https://git.ix5.org/felix/local-manifests-ix5 local_manifests -b 'ix5-customizations'
</code>
instead.  After you run <code>repo_update.sh</code> also run
<code>ix5_repo_update.sh</code>. This will fetch my newest changes as well.
</div>

## Prepare build environment
Run `source build/envsetup.sh` inside your build environment and then `lunch`.
Select your device from the list. It will look like `aosp_f83xx-...` where
`f83xx` is the model number of your device. You can choose between `eng` and
`userdebug` builds, see [Choose a target](https://source.android.com/setup/build/building#choose-a-target).  
For your own usage, `userdebug` is most likely what you want.

The `user` target doesn't include root. You'll need to look into
`vendorsetup.sh` for your device if you want to use it.

## Optimize the build
A full build will take about two to three hours on a beefed-out recent
system(Core i7 8th gen 4 cores, 16GB RAM, good SSD). That time can however be
cut down for successive builds to around an hour by utilizing `ccache` and
parallelizing the build.

It is recommendable to use a ccache size of around 50-100GB, depending on
whether you plan to build different ROMS or for multiple devices.
```
prebuilds/misc/linux-x86/ccache/ccache -M 50G
```
Then set the environment variable with `export USE_CCACHE=1`. This variable
needs to be set anew every time you open a new terminal, so it is
advisable to put `export USE_CCACHE=1` at the end of your `.bashrc` in your home
directory (or `/home/builder/.bashrc` if you're in a chroot).

To use parallel compilation, you need to find out how many threads your CPU has
via `nproc`. Then use that number for `make -j<nr-of-threads>`. For most
computers it will be the number of CPU cores times 2, e.g. "8" for a 4-core CPU.

## Building
Run `make -j <nr-of-threads>`. If everything went smoothly, you'll have a folder
named `out/target/product/<device-codename>/` with `boot.img`, `recovery.img`
and `system.img`. You can then flash these files onto your device with `fastboot
flash`

## Distributing
If you want to share the fruits of your labor, you can create compressed
flashable .zip files. Use `make otapackage` instead of `make`. This will create
a flashable zip file in `out/target/product/<device-codename>` named something
like `aosp_f8331-ota-eng.hostname-2018-10-19`. You can flash this file via `adb
sideload` or use a custom recovery like TWRP.

<!--
Create the directory `dist_output` and use `make dist DIST_DIR=dist_output -j
<nr-of-threads>` to produce a `brotli`-compressed zip file named something like
`aosp_f8331-ota-eng.hostname-2018-10-19`. You can flash this file via `adb
sideload` or use a custom recovery like TWRP.
-->

After you have tested your build on your own device, you can share the file
freely, given that it contains free/libre software under the GPL and other free
licenses. The license also means you have to divulge your build sources.
So put your patches into a git repo and share them with the world.

## Speedier development
- Use `m module.name -j $(nproc)` to only rebuild a single module. You can then
  `adb push` or even `adb sync` the changes directly to your device
- Use `make bootimage` to re-generate kernel and ramdisk(`boot.img`)

---

## Extra References
[AlaskaLinuxUser videos](https://www.youtube.com/channel/UCnGqG_jyyXmTzdamBpKfeHA/playlists)
