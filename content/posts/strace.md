---
title: "Strace Tricks"
description: ""
date: 2019-03-06T19:25:06+01:00
draft: true
---

To shim strace into service startup so it gets started with correct permissions
and environment:
```
service msm_irqbalance /system/bin/strace -o /data/local/tmp/msm_irqb.strace
/odm/bin/msm_irqbalance -f /vendor/etc/msm_irqbalance.conf
    socket msm_irqbalance seqpacket 0660 root root
    user root
    group root
    disabled
```

Remount `/system` as read-write, then:
```
chcon u:object_r:msm_irqbalance_exec:s0 /system/bin/strace
```

https://source.android.com/devices/tech/debug/strace
