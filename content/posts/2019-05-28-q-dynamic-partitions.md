---
title: "Android Q: Dynamic Partitions"
description: ""
date: 2019-05-29T16:15:09+02:00
draft: false
---

See [Linux Plumber's Conf: Dynamic Partitions][plumbersconf].

Buildvars:
```
PRODUCT_USE_DYNAMIC_PARTITIONS
PRODUCT_USE_DYNAMIC_PARTITION_SIZE
PRODUCT_RETROFIT_DYNAMIC_PARTITIONS
PRODUCT_BUILD_SUPER_PARTITION
```

Need to unset `BOARD_BUILD_SYSTEM_ROOT_IMAGE`, `TARGET_NO_RECOVERY` is
unaffected.

[AOSP build/make: Unset system-as-root for mainline][unset]:
```diff
-BOARD_BUILD_SYSTEM_ROOT_IMAGE := true
```
> This flag is not allowed to be true when dynamic partitions are
> enabled, which mainline devices are expected to do.

[plumbersconf]: https://linuxplumbersconf.org/event/2/contributions/225/attachments/49/56/06._Dynamic_Partitions_-_LPC_Android_MC_v2.pdf
[unset]: https://android-review.googlesource.com/c/platform/build/+/932362
