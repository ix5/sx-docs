---
title: "Dynamic Partitions: Take Two"
description: ""
date: 2020-05-25T20:12:48+02:00
author: Felix
slug: "dynamic-partitions-take-two"
draft: false
---

These are just my thoughts and notes, reading this will bring you no
satisfaction whatsoever.

## Starting

So, turns out, the whole affair at least seems mildly well-documented now,
at least.
See last time: [Android Q: Dynamic Partitions](/info/post/android-q-dynamic-partitions/).

See:

- [DAP Overview][dap]
- [AB Launch][ab_launch]
- [AB Legacy][ab_legacy]
- [Non-AB][nonab]

### Adjustments

Kang all the needed stuff from crosshatch, because it didn't launch with DAP and
is also an SDM845 device.

- All in `BoardConfig` and device, platform, device-common
- `fstab`, `fstab.persist`
- Don't forget to copy fstab into `first_stage_mount` ramdisk of recovery
- init changes, `latemount`/`first_stage`

See:

- [Common](https://github.com/ix5/device-sony-common/tree/dynamic-partitions)
- [Tama](https://github.com/ix5/device-sony-tama/tree/dynamic-partitions)
- [Apollo](https://github.com/ix5/device-sony-apollo/tree/dynamic-partitions)

Now, run `make otardppackage` to generate the **R**etrofit **D**ynamic
**P**artitions over-the-air package.

This will also put `super_system.img`, `super_vendor.img` into the intermediate
target files.
```
$ ls obj/PACKAGING/target_files_intermediates/aosp_h8314-target_files-eng.builder/
BOOT/  IMAGES/  META/  OTA/  PREBUILT_IMAGES/  PRODUCT/  ROOT/  SYSTEM/  VENDOR/

$ ls IMAGES/
boot.img  dtbo.img  product.img  product.map  super_empty.img
system.img  system.map  userdata.img  vbmeta.img  vendor.img  vendor.map

$ ls OTA/
android-info.txt  super_system.img  super_vendor.img
```

But, seems Android doesn't want to boot my stuff unless it's `f2fs` instead of
`ext4` for userdata? Maybe some of their fancy checkpointing stuff is broken on
ext4? (More on that later)

So, let's use `f2fs` for userdata, set it in `fstab` and also in the
BoardConfig:
```
BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE := f2fs
```
Also add some tooling:
```
PRODUCT_PACKAGES += \
    sg_write_buffer \
    f2fs_io \
    check_f2fs
```

f2fs stuff with newer kernels is kinda broken:
```
2020-05-25 20:12:33 - build_image.py - ERROR   : Failed to build out/target/product/apollo/userdata.img from out/target/product/apollo/data
# Added this for debugging to see which command fails:
#mkfs_output: Command= ['mkf2fsuserimg.sh', \
#  'out/target/product/apollo/userdata.img', \
#  '51448807424', '-f', 'out/target/product/apollo/data', \
#  '-D', 'out/target/product/apollo/system', '-s', \
#  'out/target/product/apollo/obj/ETC/file_contexts.bin_intermediates/file_contexts.bin', \
#  '-t', 'data', '-L', 'data']
# Continues:
Out of space? Out of inodes? The tree size of out/target/product/apollo/data is 0 bytes (0 MB), with reserved space of 0 bytes (0 MB).
The max image size for filesystem files is 51448807424 bytes (49065 MB), out of a total partition size of 51448807424 bytes (49065 MB).
```

The message is entirely wrong and misleading. The underlying issue is that the
f2fs tools shipped with android choke on too new kernel versions (I was on 5.6
as of writing this). Google still recommends using Ubuntu 12.04 for building
Android somewhere - or they might've updated to the brand new 14.04? - and you
should keep this in mind.

This fails:
```
mkf2fsuserimg.sh \
    out/target/product/apollo/userdata.img 51448807424 \
    -f out/target/product/apollo/data \
    -D out/target/product/apollo/system \
    -s out/target/product/apollo/obj/ETC/file_contexts.bin_intermediates/file_contexts.bin \
    -t data \
    -L data
```

With f2fs tools `sload_f2fs` spitting errors:
```

[FSCK] Unreachable nat entries                        [Ok..] [0x0]
[FSCK] SIT valid block bitmap checking                [Ok..]
[FSCK] Hard link checking for regular file            [Ok..] [0x0]
[FSCK] valid_block_count matching with CP             [Ok..] [0x9]
[FSCK] valid_node_count matcing with CP (de lookup)   [Ok..] [0x4]
[FSCK] valid_node_count matcing with CP (nat lookup)  [Ok..] [0x4]
[FSCK] valid_inode_count matched with CP              [Ok..] [0x3]
[FSCK] free segment_count matched with CP             [Ok..] [0x5f2b]
[FSCK] next block offset is free                      [Ok..]
[FSCK] fixing SIT types
[FSCK] other corrupted bugs                           [Fail]
Info: Write valid nat_bits in checkpoint
```

This logic is wrong and broken in `fsck/fsck.c`:
```
fsck_chk_quota_files()
  [...]
  if (!ret) {
      c.bug_on = 1;
      DBG(1, "OK\n");
  } else {
      ASSERT_MSG("Unable to write quota file");
  }
```

Switch the logic (left as an exercise to the reader) and `make sload_f2fs-host`
and `make fsck.f2fs-host`.

Now we're talkin', baby!
```
[FSCK] other corrupted bugs                           [Ok..]

Done.
```

**Eh, wrong again, it was something else entirely.**

I should've read the stacktrace more diligently:
```
Traceback (most recent call last):
  [...]
  File "build/make/tools/releasetools/build_image.py", line 479, in BuildImage
    mkfs_output = BuildImageMkfs(in_dir, prop_dict, out_file, target_out, fs_config)
  File "build/make/tools/releasetools/build_image.py", line 335, in BuildImageMkfs
    mkfs_output = common.RunAndCheckOutput(build_command)
  [...]
OSError: [Errno 2] No such file or directory
```

The build system can't find the `mkf2fsuserimg` script unless it's either
referred to by full name or has previously been build and put into
`out/host/linux-x86/bin/` by the build system.

Run `make mkf2fsuserimg.sh-host` and you should have it.

<!--
Use f2fs, but fix f2fs kernel detection, then `make` the `make_f2fs`,
`sload_f2fs` and `fsck.f2fs` modules with `-host` appended.
-->

### Disappointment
Alright, after some shenanigans to enable adb in recovery, we're booting, but
only into recovery. Something is broken, or rather, a lot.


Seems the kernel `cmdline` is being overflowed...
```
(Reformatted for better readability)
[0.000000] Kernel command line:
rcupdate.rcu_expedited=1
lpm_levels.sleep_disabled=1
androidboot.bootdevice=soc/1d84000.ufshc
swiotlb=2048
service_locator.enable=1
msm_drm.dsi_display0=dsi_panel_somc_tama_cmd:config0
androidboot.selinux=permissive
androidboot.memcg=1
msm_rtb.filter=0x3F
ehci-hcd.park=3
coherent_pool=8M
sched_enable_power_aware=1
user_debug=31
printk.devkmsg=on
loop.max_part=16
kpti=0
androidboot.hardware=apollo
androidboot.super_partition=system
buildvariant=userdebug
androidboot.verifiedbootstate=orange
androidboot.keymaster=1
root=PARTUUID=f9cdf7ba-b834-a72a-f1c9-d6e0c0983896
androidboot.bootdevice=1d84000.ufshc
androidboot.baseband=msm
lcdid_adc=1145782
androidboot.slot_suffix=_a
skip_initramfs
rootwait
ro
init=/init
androidboot.dtbo_idx=2
androidboot.dtb_idx=2
androidboot.bootloader=xboot
oemandroidboot.xboot=1310-7079_X_Boot_SDM845_LA2.0.1_Q_206
androidboot.serialno=XXXXXXXXXX
!!! Not changed:
oemandroidboot.babe08a4=..empty
startup=0x00008000
warmboot=0x00000000
!!!! Below is cut off:
oemandroidboot.b
```

...and we have a nice kernel panic due to some mess:
```
[2.541154] Warning: unable to open an initial console.
[2.541276] md: Waiting for all devices to be available before autodetect
[2.541387] md: If you don't use raid, use raid=noautodetect
[2.541787] md: Autodetecting RAID arrays.
[2.541898] md: autorun ...
[2.541958] md: ... autorun DONE.
[2.544812] F2FS-fs (sda42): Magic Mismatch, valid(0xf2f52010) - read(0x0)
[2.544926] F2FS-fs (sda42): Can't find valid F2FS filesystem in 1th superblock
[2.545211] F2FS-fs (sda42): Magic Mismatch, valid(0xf2f52010) - read(0x0)
[2.545323] F2FS-fs (sda42): Can't find valid F2FS filesystem in 2th superblock
[2.545447] F2FS-fs (sda42): Magic Mismatch, valid(0xf2f52010) - read(0x0)
[2.545506] F2FS-fs (sda42): Can't find valid F2FS filesystem in 1th superblock
[2.545617] F2FS-fs (sda42): Magic Mismatch, valid(0xf2f52010) - read(0x0)
[2.545726] F2FS-fs (sda42): Can't find valid F2FS filesystem in 2th superblock
[2.545858] [EXFAT] trying to mount...
[2.546179] VFS: Cannot open root device "PARTUUID=f9cdf7ba-b834-a72a-f1c9-d6e0c0983896" or unknown-block(259,26): error -5
( -5 = -EIO =  "I/O error")
[2.546243] Please append a correct "root=" boot option; here are the available partitions:
[2.546360] 0100            8192 ram0 
[2.546362]  (driver?)
[...]
[2.548881]   0801            2048 sda1 02841782-0268-44d8-8339-11e9e3e5045c
[2.548883] 
[2.549064]   0802           16384 sda2 9de21554-8b13-4e7d-b1e4-0371eb92073f
[2.549065] 
[...]
(This is sda42 aka userdata:)
[2.555792]   103:0001a    4128768 sda42 f9cdf7ba-b834-a72a-f1c9-d6e0c0983896
[2.555793] 
[...]
[2.562123] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(259,26)
[2.562241] CPU: 0 PID: 1 Comm: swapper/0 Tainted: G        W       4.14.176-104797-g31ee8bf990bd-dirty #2
[2.562351] Hardware name: Sony Mobile Communications. Apollo(SDM845 v2.1) (DT)
```

Could be an error in the f2fs implementation, though I already tried using ext4
for userdata and it gave me the same error.

### Giving up for now
Yeah so apparently Google just wants everyone to beg Qcom for bootloader updates
instead, as they have done for crosshatch.

I haven't seen any successful retrofit DAP in the wild so far, only people
[begging for help and crickets in response][groups].

There's also the matter of the supposedly removed `skip_initramfs` parameter,
and how you're supposed to pass a new parameter named
`androidboot.force_normal_boot=1` now... I don't know, this all has been a
massive waste of time.

See: [System-as-Root changes][sar].

### Other ideas

Might have messed up the `androidboot.boot_devices` parameter?
See [Kernel command line changes][cmdline].

[cmdline]: https://source.android.com/devices/tech/ota/dynamic_partitions/implement#kernel-command-line-changes
[sar]: https://source.android.com/devices/tech/ota/dynamic_partitions/implement#system-as-root-changes
[groups]: https://groups.google.com/forum/#!msg/android-building/cZoXCdFCmi0/__pRVlnnAwAJ
[dap]: https://source.android.com/devices/tech/ota/dynamic_partitions
[dap]: https://source.android.com/devices/tech/ota/dynamic_partitions/implement
[ab_launch]: https://source.android.com/devices/tech/ota/dynamic_partitions/ab_launch
[ab_legacy]: https://source.android.com/devices/tech/ota/dynamic_partitions/ab_legacy
[nonab]: https://source.android.com/devices/tech/ota/dynamic_partitions/nonab
