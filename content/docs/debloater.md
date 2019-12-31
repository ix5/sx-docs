---
title: "Xperia XZ Debloater"
description: "De-bloat the Stock Firmware"
date: 2019-12-30T22:12:12+01:00
weight: 20
draft: true
bref: ""
toc: false
---

Based on:

- [AROMA Stock Light ROM/Patch v6](https://forum.xda-developers.com/xperia-xz/development/oreo-stock-rom-cleaner-bloatware-t3725330)
  by [korom42](https://forum.xda-developers.com/member.php?u=5033594).
- [Rooted Stock Kernels F8331-F8332 41.3.A.2.192](https://forum.xda-developers.com/xperia-xz/how-to/rooted-kernels-f8332-41-3-2-588-0-t3748987)
  by [YasuHamed](https://forum.xda-developers.com/member.php?u=5613633)
- [drmfix and more](https://forum.xda-developers.com/xperia-xz/how-to/guide-stock-kernel-twrp-safetynet-patch-t3710781)
  by korom42, tobias.waldvogel and munjeni

More:

- AROMA installer
- Roots your device, magisk-ready
- De-bloats
- Swaps in compatible Google replacements for Sony crap
- Updates Sony default apps to those taken from XZ2 or newer
- Keeps `/oem` partition intact, preserving SafetyNet

Kernel:

- `RIC` disabled
- `force_encrypt` disabled
- Added DRM patch support (`drmfix`)
- Bypass SafetyNet
