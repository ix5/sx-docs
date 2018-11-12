---
title: "Treble for Dora!"
date: 2018-11-07T01:18:00+01:00
draft: true
author: sjll
---

TARGET_COPY_OUT_VENDOR := system/vendor
This setting refers to the location where the files is output.
We could only set it with system/vendor and /vendor.
It could not be /oem, otherwise, the compiler will report an error.

So, the BOARD_ROOT_EXTRA_SYMLINKS := /system/vendor/lib/dsp:/dsp
We need modify it to, BOARD_ROOT_EXTRA_SYMLINKS := /vendor/lib/dsp:/dsp

Then, the compiler will build a vendor.img , instead of copying all the files we needed to /system/vendor.
After done this, we need to enable VNDK. VNDK is needed for treble.

In /device/sony/dora/device.mk, we need to add the VNDK configs.
And in device/sony/dora/BoardConfig.mk

Besides , we need to modify the fstab and add support to vendor  partition.
The fstab is in device/sony/tone/rootdir/vendor/etc

Then, we also need to add selinux configs for Vendor partition.
In /device/sony/tone/sepolicy_platform/file_contexts

Then all the configs has done for Treble.

This is only worked in AOSP 9.0. Because enable VNDK need the drivers support it. The SODP offcial is modify the AOSP 9.0 drivers  source codes for VNDK.

For treble, all the things we need to do are enable VNDK support and create a vendor.img.

To add a vedor partition, we only need use the sgdisk.
Treble for dora try to split 800MB space from the userdata partition, and create a vendor partition.
You just need to calculate how many sector you needed.

Sorry, there is a very very important thing I forget to say.
We need to modify the fstab config in kernel.
In kernel/sony/msm-4.9/kernel/arch/arm64/boot/dts/qcom/msm8996-tone-common.dtsi

In /device/sony/dora/BoardConfig.mk
BOARD_SYSTEMIMAGE_PARTITION_SIZE

If you don't want to resize any partitions. I can offer you another method.
You can use a loop partition.Like a hybird partition.
https://github.com/phhusson/treble_experimentations/tree/master/no-vendor

You can use odm partition as a loop partition.

This requires too many files to be modified, although it does not need to be repartitioned.
I also tried another way to support treble.
https://github.com/cryptomilk/android_device_sony_lilac

This is a lineage ROM for XZ1 Compact. It is based on the sony offical ROM.
It supports treble, but I have a problem, I don't know how to create sepolicy files.
