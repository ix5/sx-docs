---
title: "DRAFT: Kernel Compilation"
slug: "kernel-compilation"
description: ""
date: 2020-06-20T21:24:31+02:00
weight: 20
draft: false
bref: ""
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

### Export cross compiler
```
git clone https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86 -b android-10.0.0_r39 clang-linux-x86
```
<!-- TODO: Fix linaro link, clang git clone, set clang version per Android level -->
```
export CROSS_COMPILE=<android-dir>/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-
export CROSS_COMPILE_32=<android-dir>/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-
# For Q and clang:
export CC=prebuilts/clang/host/linux-x86/clang-r353983c1/bin
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

| Device name          | defconfig                        |
| -------------------- | ----------------------------     |
| Xperia X             | `aosp_loire_suzu_defconfig`      |
| Xperia X Compact     | `aosp_loire_kugo_defconfig`      |
| Xperia Touch         | `aosp_loire_blanc_defconfig`     |
| Xperia X Performance | `aosp_tone_dora_defconfig`       |
| Xperia XZ            | `aosp_tone_kagura_defconfig`     |
| Xperia XZs           | `aosp_tone_keyaki_defconfig`     |
| Xperia XZ Premium    | `aosp_yoshino_maple_defconfig`   |
| Xperia XZ1           | `aosp_yoshino_poplar_defconfig`  |
| Xperia XZ1 Compact   | `aosp_yoshino_lilac_defconfig`   |
| Xperia XA2           | `aosp_nile_pioneer_defconfig`    |
| Xperia XA2 Ultra     | `aosp_nile_discovery_defconfig`  |
| Xperia XA2 Plus      | `aosp_nile_voyager_defconfig`    |
| Xperia XZ2           | `aosp_tama_akari_defconfig`      |
| Xperia XZ2 Compact   | `aosp_tama_apollo_defconfig`     |
| Xperia XZ3           | `aosp_tama_akatsuki_defconfig`   |
| Xperia 10            | `aosp_ganges_kirin_defconfig`    |
| Xperia 10 Plus       | `aosp_ganges_mermaid_defconfig`  |
| Xperia 1             | `aosp_kumano_griffin_defconfig`  |
| Xperia 5             | `aosp_kumano_bahamut_defconfig`  |
| Xperia 10 II         | `aosp_seine_pdx201_defconfig`    |

### Configure kernel
```
make ARCH=arm64 CROSS_COMPILE=$CROSS_COMPILE CROSS_COMPILE_32=$CROSS_COMPILE_32 <device_defconfig>
```

For clang:
```
make ARCH=arm64 CC=$CC CROSS_COMPILE=$CROSS_COMPILE CROSS_COMPILE_32=$CROSS_COMPILE_32 <device_defconfig>
```

### Build kernel
```
make \
  ARCH=arm64 \
  CROSS_COMPILE=$CROSS_COMPILE \
  CROSS_COMPILE_32=$CROSS_COMPILE_32 \
  -j $(nproc)
```
For clang:
```
make \
  ARCH=arm64 \
  CC=$CC \
  CROSS_COMPILE=$CROSS_COMPILE \
  CROSS_COMPILE_32=$CROSS_COMPILE_32 \
  -j $(nproc)
```

### Build bootimage and dtboimage
Navigate into top-level Android directory.

```
source build/envsetup.sh && lunch
make -j $(nproc) bootimage
```

See also: Marijn's [oot script][oot].

See also: [Handling Android Images]({{<ref "images.md" >}}).

### Download fastboot

On Linux: Install fastboot package

On Windows: Link to drivers

### Flash via fastboot
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

[unlock]: https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader
[sony]: https://developer.sony.com/develop/open-devices/guides/kernel-compilation-guides/how-to-build-and-flash-a-linux-kernel-for-aosp-supported-devices#AutoLinuxKernel
[oot]: https://github.com/MarijnS95/oot
