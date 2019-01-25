+++
title = "Handling Android Images"
description = ".img, sparse images, ramdisks"
date = 2018-10-19T22:14:54+02:00
weight = 20
draft = true
bref = "Dealing with .img, sparse images, ramdisks"
toc = true
+++

## Unpack boot.img
See [stackoverflow](https://unix.stackexchange.com/questions/64628/how-to-extract-boot-img/459881).

Get `android_bootimg_tools` and extract to `~/.local/bin`:  
`wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/android-serialport-api/android_bootimg_tools.tar.gz /tmp/tools.tar.gz`  
`tar xzvf /tmp/tools.tar.gz -C ~/.local/bin/`

Copy the boot.img you want to disect into your local folder and run
`mkdir boot/ && unpackbootimg -i boot.img -o boot/`

`cd` into `boot/` and unpack the ramdisk:
```
gunzip --to-stdout --uncompress boot.img-ramdisk.gz | cpio --extract \
  --make-directories --no-absolute-filenames
```

To re-pack a boot.img, first gzip-compress the ramdisk files, then use
`mkbootimg` with the appropriate flags.
<!-- TODO: List the flags here -->

## Handling sparse images
What is a sparse image?
simg2img

## Filesystems
ext4, ext2, exfat, ntfs, fat, fuse, permissions

## Partitions
FOTAKernel, boot, system, vendor, oem/odm, dsp, frp, modem

## Tools
- `adb`, `fastboot`
- `android_bootimg_tools`
- `android-simg2img`
- `smali`/`baksmali`
- `sdat2img`
- `vdexExtractor`
- `apktool`
