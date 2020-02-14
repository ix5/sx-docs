---
title: "Universal Sony DualSIM Patcher"
slug: "dualsim-patcher"
description: ""
date: 2020-01-10T17:23:28+01:00
draft: false
---

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

We also need to patch the vintf manifest at `/vendor/etc/vintf/manifest.xml` to
include a second instance of the telephony-related HALs. See the added `slot2`
and new instances from the `_ss` to the `_ds` variants here:

- `android.hardware.radio`: [single-sim][radio_ss] vs [dual-sim][radio_ds]
- `vendor.qti.hardware.radio`: [single-sim][qti_ss] vs [dual-sim][qti_ds].

To view the actual inner workings, [take a look at the source][source].
Particularly interesting is [the script that does the heavy lifting][script].

## Download
[sony-dualsim-patcher-v4.zip at sx.ix5.org][download]

### Changelog

- Version 4: Fix issues with A/B devices, Android 10 compat
- Version 3: Internal rework, more devices
- Version 2: Compatibility with Xperia XZ Project Treble builds
- Version 1: Initial release

[radio_ss]: https://github.com/sonyxperiadev/device-sony-common/blob/dc5aa15bba3c8e7bf6a37eea14dc8157ae9493f4/vintf/android.hw.radio_ss.xml
[radio_ds]: https://github.com/sonyxperiadev/device-sony-common/blob/dc5aa15bba3c8e7bf6a37eea14dc8157ae9493f4/vintf/android.hw.radio_ds.xml
[qti_ss]: https://github.com/sonyxperiadev/device-sony-common/blob/dc5aa15bba3c8e7bf6a37eea14dc8157ae9493f4/vintf/vendor.hw.radio_ss.xml
[qti_ds]: https://github.com/sonyxperiadev/device-sony-common/blob/dc5aa15bba3c8e7bf6a37eea14dc8157ae9493f4/vintf/vendor.hw.radio_ds.xml
[download]: https://sx.ix5.org/files/builds/sony/sony-dualsim-patcher-v4.zip
[source]: https://git.ix5.org/felix/dualsim-patcher
[script]: https://git.ix5.org/felix/dualsim-patcher/src/branch/master/tmp/patch_dualsim.sh
