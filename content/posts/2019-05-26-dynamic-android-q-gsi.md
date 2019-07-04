---
title: "Dynamic Android: Q GSIs"
description: ""
date: 2019-05-27T17:51:12+02:00
draft: true
---

Overview: [Dynamic System Updates](https://developer.android.com/topic/dsu)
(The feature has been renamed countless times)

```
gsi_tool install \
  --gsi-size 2147483648 \
  --userdata-size 4294967296 \
  /path/to/my-gsi.img
```

Sizes are in byte

```
device:/ # gsi_tool install --gsi-size 2147483648 --userdata-size 4294967296 /sdcard/my-gsi.img
write gsi         100% [================================================================================]
create userdata   100% [================================================================================]
create system     100% [================================================================================]
write gsi           0% [--------------------------------------------------------------------------------]
```

But it's stuck at `write gsi` forever.

Seems that dynamic partitions are a requirement, may need to investigate that
and how it affects fastboot, which is supposed to move into userspace somehow.

Also need a different data format for the GSI image.
