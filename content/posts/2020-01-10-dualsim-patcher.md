---
title: "DualSIM Patcher"
description: ""
date: 2020-01-10T17:23:28+01:00
draft: false
---
## Universal Sony DualSIM patcher

Because the differences between single- and dual-SIM builds are very small,
almost all developers of Sony devices - including me - distribute only
single-SIM builds.

We need to patch some props in /system/build.prop (or /system/vendor/build.prop,
/vendor/build.prop, depending on the device), mainly:
```
persist.vendor.radio.multisim.config=dsds
ro.telephony.default_network=9,1 # (or 9,0, or 9,9, depending on the device)
ro.product.name=<dualsim device name>
```

And some others.

We also need to patch the vintf manifest at /vendor/etc/vintf/manifest.xml to
include a second instance of the telephony-related HALs.

Download: [sony-dualsim-patcher-v4.zip at sx.ix5.org][download]

## Changelog

- Version 4: Fix issues with A/B devices, Android 10 compat
- Version 3: Internal rework, more devices
- Version 2: Compatibility with Xperia XZ Project Treble builds
- Version 1: Initial release

[download]: https://sx.ix5.org/files/builds/sony/sony-dualsim-patcher-v4.zip
