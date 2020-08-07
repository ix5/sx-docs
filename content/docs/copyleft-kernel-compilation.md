---
title: "Copyleft Kernel Compilation"
slug: "copyleft-kernel-compilation"
description: ""
date: 2020-06-20T21:24:49+02:00
weight: 20
draft: true
bref: ""
toc: true
---

TODO: Use generated defconfig fragments, dtbo, stuff

Sony link: [How to build and flash a Linux Kernel from the Copyleft Archives][sony]

### Download cross compiler

### Export cross compiler

### Download kernel source
```
git clone https://github.com/sonyxperiadev/kernel-copyleft
cd kernel-copyleft
```

### Identify defconfig

### Configure kernel

### Build kernel

### Build bootimage and dtboimage

See also: [Handling Android Images]({{<ref "images.md" >}}).

### Download fastboot

### Flash via fastboot
[sony]: https://developer.sony.com/develop/open-devices/guides/kernel-compilation-guides/how-to-build-and-flash-a-linux-kernel-from-sony-copyleft-archives
