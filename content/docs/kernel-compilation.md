---
title: "DRAFT: Kernel Compilation"
slug: "kernel-compilation"
description: "Building and flashing a Linux kernel for Open Devices AOSP"
date: 2020-06-20T21:24:31+02:00
weight: 20
draft: false
bref: "Building and flashing a Linux kernel for Open Devices AOSP"
toc: true
---

Sony link: [How to build and flash a Linux Kernel for AOSP][sony]

<!--
TODO:
- Remove 4.8 GCC info
- Remove automatic AOSP build kernel section, not supported on Q+
-->

### Unlock bootloader
See [Unlock Bootloader][unlock] on sony.com

### Download cross compiler
Linaro or clang

**Clang:**
```
git clone https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86 -b android-10.0.0_r39 clang-linux-x86
```

**Linaro GCC:**
Download
<!-- TODO: Use linux releases, not mingw -->
`gcc-arm-9.2-2019.12-mingw-w64-i686-arm-none-linux-gnueabihf.tar.xz`
and
`gcc-arm-9.2-2019.12-mingw-w64-i686-aarch64-none-linux-gnu.tar.xz`
from the [ARM website][linaro-arm], extract the files.
<!--
Linaro: Download `gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz` from [releases.linaro.org][linaro].
-->

### Export cross compiler
<!-- TODO: Fix linaro link, clang git clone, set clang version per Android level -->

Linaro GCC:
```
export CROSS_COMPILE=<android-dir>/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-
export CROSS_COMPILE_ARM32=<android-dir>/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-
```

For clang:
```
export CC=clang-linux-x86/clang-r353983c1/bin
export CLANG_TRIPLE=aarch64-linux-gnu
```

### Download kernel source
```
git clone https://github.com/sonyxperiadev/kernel
cd kernel
```
Also techpack/audio, data-kernel, wifi modules

### Identify defconfig

The defconfig `name` is constructed using this schema:
`aosp_<platform>_<codename>_defconfig`.

As for the `codename`, search the
[Sony Devices overview]({{<ref "sony-devices.md" >}}) for the device `Code
name`, e.g. `kagura` for the Xperia XZ.

For `platform`, use the "board" name, e.g. `tone` for Xperia XZ.

So, Xperia XZ becomes `aosp_tone_kagura_defconfig`.

For easier reference, here's an overview:

| Device name          | defconfig                        | DTBO  |
| -------------------- | -------------------------------- | ----- |
| Xperia X             | `aosp_loire_suzu_defconfig`      | false |
| Xperia X Compact     | `aosp_loire_kugo_defconfig`      | false |
| Xperia Touch         | `aosp_loire_blanc_defconfig`     | false |
| Xperia X Performance | `aosp_tone_dora_defconfig`       | false |
| Xperia XZ            | `aosp_tone_kagura_defconfig`     | false |
| Xperia XZs           | `aosp_tone_keyaki_defconfig`     | false |
| Xperia XZ Premium    | `aosp_yoshino_maple_defconfig`   | false |
| Xperia XZ1           | `aosp_yoshino_poplar_defconfig`  | false |
| Xperia XZ1 Compact   | `aosp_yoshino_lilac_defconfig`   | false |
| Xperia XA2           | `aosp_nile_pioneer_defconfig`    | false |
| Xperia XA2 Ultra     | `aosp_nile_discovery_defconfig`  | false |
| Xperia XA2 Plus      | `aosp_nile_voyager_defconfig`    | false |
| Xperia XZ2           | `aosp_tama_akari_defconfig`      | true  |
| Xperia XZ2 Compact   | `aosp_tama_apollo_defconfig`     | true  |
| Xperia XZ3           | `aosp_tama_akatsuki_defconfig`   | true  |
| Xperia 10            | `aosp_ganges_kirin_defconfig`    | true  |
| Xperia 10 Plus       | `aosp_ganges_mermaid_defconfig`  | true  |
| Xperia 1             | `aosp_kumano_griffin_defconfig`  | true  |
| Xperia 5             | `aosp_kumano_bahamut_defconfig`  | true  |
| Xperia 10 II         | `aosp_seine_pdx201_defconfig`    | true  |

### Configure kernel
```
make ARCH=arm64 CROSS_COMPILE=$CROSS_COMPILE CROSS_COMPILE_ARM32=$CROSS_COMPILE_ARM32 <device_defconfig>
```

For clang:
```
make ARCH=arm64 CC=$CC CLANG_TRIPLE=$CLANG_TRIPLE CROSS_COMPILE=$CROSS_COMPILE CROSS_COMPILE_ARM32=$CROSS_COMPILE_ARM32 <device_defconfig>
```

### Build kernel
```
make \
  ARCH=arm64 \
  CROSS_COMPILE=$CROSS_COMPILE \
  CROSS_COMPILE_ARM32=$CROSS_COMPILE_ARM32 \
  -j $(nproc)
```
For clang:
```
make \
  ARCH=arm64 \
  CC=$CC \
  CLANG_TRIPLE=$CLANG_TRIPLE \
  CROSS_COMPILE=$CROSS_COMPILE \
  CROSS_COMPILE_ARM32=$CROSS_COMPILE_ARM32 \
  -j $(nproc)
```

### Copy kernel and dtbo

#### Devices with appended DTB
Copy `<kernel>/arch/arm64/boot/Image.gz-dtb` to
`<android-dir>/kernel/sony/msm-4.14/common-kernel/kernel-dtb-<device>`.

#### DTBO devices
Copy `<kernel>/arch/arm64/boot/Image.gz` to
`<android-dir>/kernel/sony/msm-4.14/common-kernel/kernel-dtb-<device>`.

Then, build `mkdtimg` from inside your Android tree and run it:
```
export MKDTIMG=<android-dir>/out/host/linux-x86/bin/mkdtimg
$MKDTIMG create <android-dir>kernel/sony/msm-4.14/common-kernel/dtbo-<device>.img \
  `find <kernel-dir>/arch/arm64/boot/dts -name "*.dtbo"`
```

### Build bootimage and dtboimage

Navigate into top-level Android directory.

```
source build/envsetup.sh && lunch
make -j $(nproc) bootimage
```

The DTBO image will be generated automatically when building boot image.

See also: Marijn's [oot script][oot].

See also: [Handling Android Images]({{<ref "images.md" >}}).

### Download fastboot

On Linux: Install fastboot package

On Windows: Link to drivers

### Flash via fastboot
Make sure the current slot is correct:
```
$ fastboot getvar current-slot
-> "a"
```

Else, set it:
```
fastboot set_active a
```

Then, flash images:
```
fastboot flash boot out/target/product/<device>/boot.img
# For DTBO devices:
fastboot flash dtbo out/target/product/<device>/dtbo.img
```

<!--
TODO: Logical partitions? avoid fastbootd
-->

Finally:
```
fastboot reboot
```

```
# https://t.me/PixelExperienceSony/34425
#!/bin/bash

export ANDROID_ROOT=/home/kgvarunkanth/kgv/arrow
export KERNEL_TOP=$ANDROID_ROOT/kernel/sony/msm-4.14
export KERNEL_TMP=$ANDROID_ROOT/out/kernel-4.14
export CC=$ANDROID_ROOT/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/
export CC32=$ANDROID_ROOT/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin/
export CLANG=$ANDROID_ROOT/prebuilts/clang/host/linux-x86/clang-r353983c/bin/
export PATH=$CC:$CC32:$CLANG:$PATH
rm -rf $KERNEL_TMP
make O=$KERNEL_TMP clean
make O=$KERNEL_TMP mrproper
make O=$KERNEL_TMP ARCH=arm64 aosp_yoshino_maple_defconfig
make O=$KERNEL_TMP ARCH=arm64 CC=clang CLANG_TRIPLE=aarch64-linux-gnu- CROSS_COMPILE=aarch64-linux-android- CROSS_COMPILE_ARM32=arm-linux-androideabi- -j1
```


[unlock]: https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader
[sony]: https://developer.sony.com/develop/open-devices/guides/kernel-compilation-guides/how-to-build-and-flash-a-linux-kernel-for-aosp-supported-devices#AutoLinuxKernel
[oot]: https://github.com/MarijnS95/oot
[linaro]: https://releases.linaro.org/components/toolchain/binaries/latest-7/aarch64-linux-gnu/
[linaro-arm]: https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads
