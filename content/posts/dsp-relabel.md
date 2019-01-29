+++
title = "DSP file relabling for SODP"
description = "Fix wrong SELinux labels"
date = 2019-01-29T12:33:55+01:00
bref = "Fix wrong SELinux labels"
draft = false
author: Felix
+++

If you have previously flashed *any* version of a Sony-Open-Devices-based ROM
from before **December 17th 2018**, your `dsp` partition might have wrong
SELinux labels and you might thus have problems with audio.

## Short version
Either re-flash the `dsp` partition with stock firmware via Sony's own
[Emma][emma] or the open-source [FlashTool][flashtool], or manually force
the correct file labels. You need to be root via `adb shell` or in recovery:
```
su
# It might be either /system/vendor/dsp or /vendor/dsp depending on your device
# Just try both until one succeeds
mount -o remount,rw /system/vendor/dsp
chcon -R u:object_r:adsprpcd_file:s0 /dsp
```
Reboot, et voilà - you should have proper audio again.

If you only want to fix your audio problem, you can stop reading now.

## Long version
Sony devices have a `dsp` partition that holds firmware files and libraries
needed for the [Qualcomm "Hexagon" DSP][hexagon] - *Digital Signal Processor*.

The partition is shippped with the [SELinux file label][source-selinux-label]
`u:object_r:adsprpcd_file:s0` by default, and stock firmware uses this label
as well when flashed via `Emma` or `FlashTool`.

When stellirin rewrote the SODP sepolicy in late 2017, they
[chose to use `qdsp_file`][rewrite-qdsp] instead of `adsprpcd_file` for `/dsp`.
A few lines of code [were added in `init.$device.rc`][tone-init] - which gets
run under the label `vendor_init` to re-mount the dsp partition and update it to
use the `qdsp_file` label.

Because re-labeling these files as `vendor_init` is a
[neverallow in Pie][relabel-neverallow], we noticed that we should rather be
re-using the stock file labels, i.e. `adsprpcd_file`.  
The relabeling bit(`restorecon_recursive`) [was removed][init-drop-dsp]
from its new location in `init.common.rc` and the "new" old label
[was restored][sepolicy-adsp].

As stated in the "short version", fixing the problem is easy: Run `chcon` or run
`Emma` - Emma [can now flash single partitions][emma-single], which is great.

## Conclusion
A small inconsistency can cause a whole lot of trouble. Who could have predicted
back in 2017 that Google would tighten the security around re-labeling files?  
The transition is going to be a bit painful, but in the end, re-using
`adsprpcd_file` is the better solution.

[hexagon]: https://en.wikipedia.org/wiki/Qualcomm_Hexagon
[source-selinux-label]: https://source.android.com/security/selinux/implement#context-files
[emma]: https://software.sonymobile.com/www/
[flashtool]: https://forum.xda-developers.com/showthread.php?t=920746
[rewrite-qdsp]: https://github.com/sonyxperiadev/device-sony-sepolicy/commit/14fe403ac06cdb2d1338795a9f6dc169f0a54f19#diff-a0a34322e457d2d0a31d35c32ca9ef01R31
[tone-init]: https://github.com/sonyxperiadev/device-sony-tone/commit/0bfc628924d467b8b916c7ed4ff681e6dfbc535d
[relabel-neverallow]: https://android.googlesource.com/platform/system/sepolicy/+/android-9.0.0_r30/public/domain.te#423
[init-drop-dsp]: https://github.com/sonyxperiadev/device-sony-common/commit/ec07d05d7d7cd7830a6f756cc06e3d2278e2a071
[sepolicy-adsp]: https://github.com/sonyxperiadev/device-sony-sepolicy/commit/633d33919acc1328115a688c412d1e7435bb60b4
[emma-single]: https://developer.sony.com/posts/flash-tool-updated-with-new-feature/
