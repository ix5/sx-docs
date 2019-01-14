+++
title = "Kernel compilation"
description = ""
date = 2018-10-19T22:15:22+02:00
weight = 20
draft = true
bref = ""
toc = true
+++

mkbootimg --kernel boot-8.1.img-zImage --ramdisk boot-9.0.img-ramdisk.gz --cmdline "androidboot.bootdevice=7464900.sdhci androidboot.selinux=permissive msm_rtb.filter=0x3F ehci-hcd.park=3 coherent_pool=8M sched_enable_power_aware=1 user_debug=31 androidboot.hardware=kagura buildvariant=userdebug" --base 80000000 --pagesize 4096 -o boot-repack.img
