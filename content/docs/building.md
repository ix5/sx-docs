+++
title = "Building Android"
description = "How to build AOSP for Xperia devices"
date = 2018-10-19T22:14:54+02:00
weight = 10
draft = false
bref = "How to build AOSP for Xperia devices"
toc = true
+++

This is just a build manual.

*(For Linux)*

## Ubuntu chroot

Since Google use Ubuntu as a reference build environment, it is advisable you
use it as well to avoid incompatibility problems. A build chroot takes about
1-1.5GB of space and will save you from headaches, e.g. when Android modules
aren't compiled against your host glibc.

Install `debootstrap`, then run
```
debootstrap --variant=buildd --arch=amd64 bionic ubuntu-android
```
chroot into the newly created builder system, either via `chroot` or just using
`systemd-nspawn`:
```
sudo systemd-nspawn \
--bind=/home/<your-username>/dev/android/build-env/:/home/builder/build-env/ \
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
apt install ccache
apt install rsync
```
Cross-reference with the
[official build guide](https://developer.sony.com/develop/open-devices/guides/aosp-build-instructions/build-aosp-android-p-9-0-0)
should this document be out of date.

Exit out of the build system by typing `exit` (and pressing `Ctrl-]` three times
for `nspawn`), then re-launch your build system and log in as your builder user.
If you want to share your own ccache etc. with your build environment, `bind`
the appropriate directories.
```
sudo systemd-nspawn \
--bind=/home/<your-username>/dev/android/build-env/:/home/builder/build-env/ \
  --bind=/home/<your-username>/.ccache/:/home/builder/.ccache/ \
  --bind=/home/<your-username>/.local/bin/repo:/usr/local/bin/repo \
  --bind=/home/<your-username>/.repoconfig/:/home/builder/.repoconfig/ \
  -D /home/<your-username>/dev/android/ubuntu-android \
  --user=builder
```

## Prepare the sources
Prepare the sources for building with `repo sync` and run Sony's
`repo_update.sh`. Confirm everything went without errors, and apply your own
patches if you like.

## Prepare build environment
Run `source build/envsetup.sh` and then `lunch`. Select your device from the
list. It will look like `aosp_f83xx-...` where `f83xx` is the model number of
your device. You can choose between `debug` and `userdebug` builds, see [Choose
a target](https://source.android.com/setup/build/building#choose-a-target).
For your own usage, `userdebug` is most likely what you want.

The `user` target doesn't include root. You'll need to look into
`vendorsetup.sh` for your device if you want to use it.

## Optimize the build
A full build will take about two hours on a beefed-out recent system(Core i7 8th
gen 4 cores, 16GB RAM, good SSD). That time can however be cut down to around an
hour by utilizing `ccache` and parallelizing the build.

It is recommendable to use a ccache size of around 50-100GB, depending on
whether you plan to build different ROMS or for multiple devices.
```
prebuilds/misc/linux-x86/ccache/ccache -M 50G
```
Then set the environment variable with `export USE_CCACHE=1`. This variable
needs to be set anew every time you re-login to your builder chroot, so it is
advisable to put `export USE_CCACHE=1" into the `.bashrc` in `/home/builder`.

To use parallel compilation, you need to find out how many threads your CPU has
via `nproc`. Then use that number for `make -j<nr-of-threads>`

## Building
Run `make -j <nr-of-threads>`. If everything went smoothly, you'll have a folder
named `out/target/product/<device-codename>/` with `boot.img`, `recovery.img`
and `system.img`. You can then flash these files onto your device with `fastboot
flash`

## Distributing
If you want to share the fruits of your labor, you can create compressed
flashable .zip files.

Create the directory `dist_output` and use `make dist DIST_DIR=dist_output -j
<nr-of-threads>` to produce a `brotli`-compressed zip file named something like
`aosp_f8331-ota-eng.hostname-2018-10-19`. You can flash this file via `adb
sideload` or use a custom recovery like TWRP.

After you have tested your build on your own device, you can share the file
freely, given that it contains free/libre software under the GPL and other free
licenses. The license also means you have to divulge your build sources.
So put your patches into a git repo and share them with the world.
