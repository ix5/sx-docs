+++
title = "Handling Android Images"
description = ".img, sparse images, ramdisks"
date = 2018-10-19T22:14:54+02:00
weight = 20
draft = false
bref = "Dealing with .img, sparse images, ramdisks"
toc = true
+++

## Unpack boot.img
See [stackoverflow](https://unix.stackexchange.com/questions/64628/how-to-extract-boot-img/459881).

Get `android_bootimg_tools` and extract to `~/.local/bin`:  
`wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/android-serialport-api/android_bootimg_tools.tar.gz /tmp/tools.tar.gz`  
`tar xzvf /tmp/tools.tar.gz -C ~/.local/bin/`  
You will now have the `mkbootimg` and `unpackbootimg` programs available.

Copy the boot.img you want to disect into your local folder and run
`mkdir boot/ && unpackbootimg -i boot.img -o boot/`

You will now have the following files inside the `boot/` folder:

- `boot.img-base`
- `boot.img-cmdline`
- `boot.img-pagesize`
- `boot.img-ramdisk.gz`
- `boot.img-zImage`

The `boot.img-zImage` file is the kernel.

## Unpack ramdisk

*This is only necessary if you want to change files inside your boot.img. If you
only want to use a different kernel, you can skip this step.*

`cd` into `boot/` and unpack the ramdisk:
```
gunzip --to-stdout --uncompress boot.img-ramdisk.gz | cpio --extract \
  --make-directories --no-absolute-filenames
```
The ramdisk has a special format named `cpio`. More information on
[Wikipedia][cpio-wiki] and the [manpage][cpio-manpage].

## Re-pack boot.img
*If you unpacked your ramdisk, first re-create a cpio archive and gzip-compress
it again.*

Use `mkbootimg` with the appropriate flags. To find out the values for
`cmdline`, `base` and `pagesize`, take a look at the generated `boot.img-*`
files, e.g. `boot.img-base` should contain the text `80000000`.

```
$CMDLINE="androidboot.bootdevice=7464900.sdhci msm_rtb.filter=0x3F \
  ehci-hcd.park=3 coherent_pool=8M sched_enable_power_aware=1 \
  user_debug=31 cgroup.memory=nokmem printk.devkmsg=on \
  androidboot.hardware=kagura buildvariant=userdebug \
  androidboot.selinux=permissive"
$BASE="80000000" 
$PAGESIZE="4096"
$RAMDISK="boot.img-ramdisk.gz"
$KERNEL="boot.img-zImage"
mkbootimg \
  --cmdline "$CMDLINE" \
  --base "$BASE" \
  --pagesize "$PAGESIZE" \
  --ramdisk "$RAMDISK" \
  --kernel "$KERNEL" \
  -o boot-repacked.img
```

The newly packged boot image will be named `boot-repacked.img`(or whatever name
you chose for `-o`).

## Gotchas
In case you compiled your own kernel, don't forget to think about `.dtb` files!
Many older devices(e.g. the Xperia XZ) need the
[Device Tree Binaries("dtb")][dtb] appended to the kernel `zImage.

<!-- TODO: List the flags here -->

<!-- ## Handling sparse images -->
<!-- What is a sparse image? -->
<!-- simg2img -->

<!-- ## Filesystems -->
<!-- ext4, ext2, exfat, ntfs, fat, fuse, permissions -->

<!-- ## Partitions -->
<!-- FOTAKernel, boot, system, vendor, oem/odm, dsp, frp, modem -->

<!-- ## Tools -->
<!-- - `adb`, `fastboot` -->
<!-- - `android_bootimg_tools` -->
<!-- - `android-simg2img` -->
<!-- - `smali`/`baksmali` -->
<!-- - `sdat2img` -->
<!-- - `vdexExtractor` -->
<!-- - `apktool` -->

[cpio-wiki]: https://en.wikipedia.org/wiki/Cpio
[cpio-manpage]: https://www.gnu.org/software/cpio/manual/cpio.html
[dtb]: https://elinux.org/Device_Tree_Reference
