---
title: "Kernel Compilation"
description: ""
date: 2018-10-19T22:15:22+02:00
weight: 20
draft: true
bref: "Outside of the Android tree"
toc: false
---

*In case you do not have a computer powerful enough to compile all of Android,
but still want to dabble in kernel matters, you can compile the (Android) Linux
kernel out-of-tree.*

---

## Preparation
You need:
- A `gcc` compiler capable of building Android kernels
- The kernel source
- A kernel `defconfig`
- A boot image that you can transplant the newly built kernel into, best from a
  kernel of the same version(e.g. a 4.9 `boot.img` for building a 4.9 kernel)

## Building

```
export CROSS_COMPILE="path-to-compiler/arm-eabi-4.7/bin/arm-eabi-"
export ARCH=arm64
```
Best set a `KERNEL_OUT` dir

`make aosp_tone_kagura_defconfig`

`make`

you can use `make mkbootimg`, but it's best to do it manually
<!-- TODO -->

```
mkbootimg --kernel boot-8.1.img-zImage --ramdisk boot-9.0.img-ramdisk.gz --cmdline "androidboot.bootdevice=7464900.sdhci androidboot.selinux=permissive msm_rtb.filter=0x3F ehci-hcd.park=3 coherent_pool=8M sched_enable_power_aware=1 user_debug=31 androidboot.hardware=kagura buildvariant=userdebug" --base 80000000 --pagesize 4096 -o boot-repack.img
```

More info on building (mainline) Linux kernels is given at the
[Sony Developer World page][sonyguide].

[sonyguide]: https://developer.sony.com/develop/open-devices/guides/kernel-compilation-guides/how-to-build-mainline-linux-kernel-for-xperia-devices
