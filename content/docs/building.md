---
title: "Building Android"
description: "How to build AOSP for Xperia devices"
slug: "building-android"
date: 2018-10-19T22:14:54+02:00
weight: 15
draft: false
bref: "How to build AOSP for Xperia devices"
toc: true
---

<div class="message warning">
The builds for the Xperia XZ will no longer be updated. This guide is severely
out of date (as of 2022) and will most likely no longer work. Proceed with caution.
</div>

This guide will asume you follow the
[official Sony build guide](https://developer.sony.com/develop/open-devices/guides/aosp-build-instructions/build-aosp-android-android-10-0-0),
but add some extra tips.

## Initialize your build directory
Create a `~/android/build-env` folder, `cd` into it.

## Using the repo tool
Download and install the [repo][git-repo] tool, either via your package manager
or directly from Google.
Instead of copying the file, it's better to symlink:
```
mkdir -p ~/.local/bin
# Add ~/.local/bin to PATH
echo "export PATH=$HOME/.local/bin:$PATH" >> ~/.bashrc
# Symlink
ln -sf ~/android/.repo/repo/repo ~/.local/bin/repo
# Check:
readlink $(which repo)
# -> /home/builder/android/.repo/repo/repo
```

Then, run the `repo init` command:
```
repo init -u https://android.googlesource.com/platform/manifest \
  -b android-10.0.0_r39 -g default,-x86,-mips,-darwin,-notdefault
```
Use the appropriate branch tag instead of `_r39`. For an overview of currently
published tags, see [Codenames, Tags and Numbers][codenames]. Most often, you'll
want to pick the latest Pixel tag, e.g. the Pixel 4.

## Ubuntu chroot

<div class="message focus">
Since Google use Ubuntu as a reference build environment, it is advisable you use it as well to avoid incompatibility problems.
<b>If you're on Ubuntu already, you can skip this step.</b>
</div>

If you're on another system like Fedora or Arch Linux, you can set up a Ubuntu
chroot build environment instead.

Another alternative to nspawn is to use `docker`, see
[Sony AOSP on docker](https://github.com/chris42/android_build).

A build chroot takes about 1-1.5GB of space and will save you from headaches,
e.g. when Android modules aren't compiled against your host glibc.

Install `debootstrap`, then run this (`bionic` is the ubuntu version codename for
18.04 LTS):
```
debootstrap --variant=buildd --arch=amd64 bionic ~/android/ubuntu-android
```
chroot into the newly created builder system, either via `chroot` or just using
`systemd-nspawn` (Assuming you created your android build environment via `repo
init` in `~/android/build-env`):
```
sudo systemd-nspawn \
  -D /home/<your-username>/android/ubuntu-android
useradd -m -s /bin/bash builder
# -> No need to add a password, nspawn will log you in automatically
```

For Android Q, the way `.img` files are mounted during build has changed, e.g.
`resize2fs` fails with a standard nspawn container.

```
ln -s /proc/self/mounts /etc/mtab
```
And add this to the options of systemd-nspawn:
```
    --bind=/dev/loop0:/dev/loop0 \
    --bind=/dev/loop1:/dev/loop1 \
    --bind=/dev/loop2:/dev/loop2 \
    --bind=/dev/loop3:/dev/loop3 \
```
(So that enough loop devices are available)

## Install required packages
```
apt install software-properties-common python
add-apt-repository universe
apt update
apt install bison g++-multilib git gperf libxml2-utils make \
  zlib1g-dev zip liblz4-tool libncurses5
apt install openjdk-8-jdk
# Might be needed on Android 10:
apt install flex

# The following dependencies are all optional:
##############################################
# Faster recompilation
apt install ccache
apt install rsync libssl-dev aapt
# Kernel recompilation:
apt install bc autoconf automake autopoint autotools-dev bsdmainutils gawk \
  groff-base libarchive-zip-perl libpipeline1 libtimedate-perl libtool kmod
# For selinux:
apt install setools python-networkx policycoreutils
```
Cross-reference the needed packages with the
[official Sony build guide](https://developer.sony.com/develop/open-devices/guides/aosp-build-instructions/build-aosp-android-p-9-0-0)
should this document be out of date.

**If you're building in a chroot:**
Exit out of the build system by typing `exit` (and pressing `Ctrl-]` three times
for `nspawn`), then re-launch your build system and log in as your builder user.
If you want to share your own ccache etc. with your build environment, `bind`
the appropriate directories.
```
sudo systemd-nspawn \
--bind=/home/<your-username>/android/build-env/:/home/builder/build-env/ \
  --bind=/home/<your-username>/.ccache/:/home/builder/.ccache/ \
  -D /home/<your-username>/android/ubuntu-android \
  --user=builder
```

## Local manifests
Clone the Sony `local_manifests` git repo as instructed.
For legacy devices, meaning `loire` and `tone`, you need to checkout the
`android-10_legacy` branch of `local_manifests`. For recent devices have to
use the `master` branch.

```
git clone https://github.com/sonyxperiadev/local_manifests .repo/local_manifests
cd .repo/local_manifests
# For recent devices:
git checkout master
# For legacy devices
git checkout android-10_legacy
```

## Applying needed patches

Run `repo_update.sh`. Your build directory will now be ready for a first build.

<div class="message focus">
<b>Felix:</b> If you want to recreate my own builds, use
<code>
git clone https://git.ix5.org/felix/local-manifests-ix5 local_manifests -b 'ix5-customizations-10'
</code>
instead. After you run <code>repo_update.sh</code>, also run
<code>q_repo_update.sh</code>. This will fetch my newest changes as well.
<br>
<!--
<b>For Android Pie and ealier:</b> Use the <code>ix5-customizations</code> branch and
<code>ix5_repo_update.sh</code> instead.
-->
</div>

## Prepare build environment
Run `source build/envsetup.sh` inside your build environment and then `lunch
aosp_<product>-userdebug`.

To find out the product name of your device for `aosp_<product>-userdebug`, see
the [Sony Devices overview]({{<ref "sony-devices.md" >}}). E.g. for a single-SIM
("SS") Xperia XZ, choose `aosp_f8331`, for a dual-SIM XZ, choose `aosp_f8332`.

You can also run `lunch` without any arguments to get a list of all available
build targets. They will look like `aosp_f83xx-...` where `f83xx` is the model
number of your device. You can choose between `eng` and `userdebug` builds, see
[Choose a target](https://source.android.com/setup/build/building#choose-a-target).  
For your own usage, `userdebug` is most likely what you want.

The `user` target is meant for public releases and doesn't include root. You'll
need to look into `vendorsetup.sh` (`AndroidProducts.mk` for Android Q) - and
perhaps add it for your device - if you want to use it.

## Build kernels
On Android Q, you need to use a script to build the kernel images. Navigate into
`kernel/sony/msm-4.9/common-kernel` and run `build-kernels-gcc.sh`.  
If building for a Kernel 4.14 device, go into
<code>kernel/sony/msm-<b>4.14</b>/common-kernel</code> and use
`build-kernels-clang.sh` instead.

Edit `build-kernels-{gcc,clang}.sh` and change the list of platforms in the
script so that it looks like this:
```
# Override:
TONE="kagura"
PLATFORMS="tone"
```
This will save you from building the kernel for *all* Sony devices and only
build for the Xperia XZ ("kagura").

## Build & Flash
For legacy non-A/B devices like the Xperia XZ, run this command:
```
make bootimage systemimage
```

Newer devices require building for more partitions, like `vendor`, `dtbo`,
`product`, `vbmeta` and others.

In case your build stops at about 90% with this error, most of the time you
should be fine by just restarting the
build with `-j 1`[^metalava].
```
[ 93% 12745/13686] //frameworks/base:hiddenapi-lists-docs Metalava [common]
FAILED: out/soong/[...]hiddenapi-lists-docs-stubs.srcjar
```

However, if this still fails, try cherry-picking
[this patch from LineageOS][heapsize]. See also [this reddit post][reddit]

Then, use `fastboot` to flash the generated images
```
cd out/target/product/kagura
fastboot flash boot boot.img
fastboot flash system system.img
```

For 4.14 devices, also flash the other partitions like `vendor`, `dtbo` etc.

Do not forget to flash the appropriate [Software Binaries][swbins] to the `oem`
partition as well!

## Optimize the build
A full build will take about two to three hours[^time] on a beefed-out recent
system (Core i7 8th gen 4 cores, 16GB RAM, good SSD). That time can however be
cut down for successive builds to around an hour by utilizing `ccache` and
parallelizing the build.

It is recommendable to use a ccache size of around 50-100GB, depending on
whether you plan to build different ROMS or for multiple devices.
```
# On Android Pie and before:
_CCACHE_EXEC=prebuilts/misc/linux-x86/ccache/ccache
# On Android Q:
_CCACHE_EXEC=/usr/bin/ccache
# Set ccache size:
$_CCACHE_EXEC -M 50G
```
Then set the environment variable with `export USE_CCACHE=1`. This variable
needs to be set anew every time you open a new terminal, so it is
advisable to put `export USE_CCACHE=1` at the end of your `.bashrc` in your home
directory (or `/home/builder/.bashrc` if you're in a chroot).

`ccache` compression might also save a lot of disk space for you. Set
`export CCACHE_COMPRESS=1` in your `.bashrc`. This cuts ccache disk usage down
to about 5GB per device, but may incur some performance penalty.

<div class="message warning">
On Android Q, you need to install the <code>ccache</code> package and set
<code>CCACHE_EXEC</code> in your <code>.bashrc</code> manually to e.g.
<code>/usr/bin/ccache</code>. The reason for this is that Google no longer ships
a prebuilt <code>ccache</code> with Android.
<br>
For more info, see
<a href="{{< ref "2019-03-16-q-changes.md" >}}">Android Q changes</a> and 
<a href="https://android.googlesource.com/platform/build/+/refs/tags/android-q-preview-1/core/ccache.mk#17">
build/core/ccache.mk</a>.
</div>

To use parallel compilation, you need to find out how many threads your CPU has
via `nproc`. Then use that number for `make -j<nr-of-threads>`. For most
computers it will be the number of CPU cores times 2, e.g. "8" for a 4-core CPU.

You can also set `export WITHOUT_CHECK_API=true` in your `~/.bashrc` to skip the
metalava checks which take up a lot of RAM. Obviously, you should run the
metalava checks when you change any Java code yourself, but you can save a bit
of time if you skip them for routine builds.

<div class="message warning">
If you're on AOSP Android Q, you need to manually cherry-pick
<a href="https://android-review.googlesource.com/#/c/1112165">Bring back env flag to skip checkapi</a>.
On LineageOS and on current (as of 2019-09-07) <code>master</code>, the CL has
been merged already.
</div>

## Distributing
If you want to share the fruits of your labor, you can create compressed
flashable .zip files. Use `make otapackage` instead of `make systemimage [...]`.
This will create a flashable zip file in `out/target/product/<device-codename>`
named something like `aosp_f8331-ota-eng.hostname-2018-10-19`. You can flash
this file via `adb sideload` or use a custom recovery like TWRP.

If you intend to share your builds publicly, you should generate and guard your
own set of signing keys. See [Signing builds for release][signing] for more
information.

<!--
Create the directory `dist_output` and use `make dist DIST_DIR=dist_output -j
<nr-of-threads>` to produce a `brotli`-compressed zip file named something like
`aosp_f8331-ota-eng.hostname-2018-10-19`. You can flash this file via `adb
sideload` or use a custom recovery like TWRP.
-->

After you have tested your build on your own device, you can share the file
freely, given that it contains free/libre software under the GPL and other free
licenses. Anything GPL-licensed also demands you have to divulge your build
sources. So put your patches into a git repo and share them with the world.

## Speedier development
- Use `m module.name -j $(nproc)` to only rebuild a single module. You can then
  `adb push` or even `adb sync` the changes directly to your device
- Use `make bootimage` to re-generate kernel and ramdisk only (`boot.img`)
- Use `adb push` or `adb sync` instead of flashing full images. N.b.: This is
  only possible with disabled vbmeta.

<!--
## Disabling signature verification
- Disable in TWRP
```
make avbtool -j
avbtool make_vbmeta_image -\-flag 2 -\-output vbmeta.img
```
-->

## Going further
To show all available build targets, use `make modules`.

Show commands used live: `make showcommands`.

Good references:  
[eLinux: The Android Build System](https://elinux.org/Android_Build_System)

[^metalava]: Android's metalava takes up enormous amounts of RAM. It attempts to
  verify the whole Android API at once and is badly designed. If it is run in
  parallel to other processes, it will frequently go OOM. By restarting the
  process, metalava will be the only one running and hopefully succeed.
[^time]: On Android Q, this time goes up to five hours for me.

[git-repo]: https://gerrit.googlesource.com/git-repo/
[codenames]: https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds
[signing]: https://source.android.com/devices/tech/ota/sign_builds
[swbins]: https://developer.sony.com/develop/open-devices/downloads/software-binaries
[heapsize]: https://review.lineageos.org/c/LineageOS/android_build_soong/+/266411
[reddit]: https://old.reddit.com/r/LineageOS/comments/fgbs5v/how_to_deal_with_outofmemoryerror_when_building/
