+++
title = "Treble for older devices"
description = "VNDK, fake /vendor, HALs, permissions and more"
date = 2018-11-16T08:42:43+01:00
draft = true
+++

<!-- Or make this article more about general treble changes? -->

## What is treble?
-> see docs/treble

# Making FULL_TREBLE work without a /vendor partition

## Challenges without /vendor
Because of the way symlinks are handled on Linux, both the source and the target
need to be accessible. On FULL_TREBLE, e.g. only /vendor/lib is accessible and
not /vendor/odm/lib. You need to monkey-patch the relevant files, e.g.
ld.config.28.txt for Pie to also allow /vendor/odm/lib to be searched and
accessed.

If you are re-using the oem partition as vendor and then symlinking /vendor/odm
to /odm to keep proprietary blobs working, you will face the same issue, since
only /odm/lib and not /vendor/odm/lib is accessible.
-> solution: just shuffle the blobs around from /vendor/odm/lib to
/vendor/lib


---
<!-- (maybe discard) -->

## Partitions
- `cache` as /vendor
- `Qnovo` as /cache

### Re-partitioning
-> Re-partitioning is likely not to happen, deemed too dangerous and messes with
people coming from and to stock, other ROMs

### Proposed partition layout for each device/storage config

Tone:
- dora
- kagura
- ?

-> needs twrp change as well, since it will write its results to /cache

Switch in device-sony-tone that enables separate vendor partition
(BoardConfig.mk)
