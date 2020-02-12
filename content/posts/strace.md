---
title: "Strace Tricks"
description: ""
date: 2019-03-06T19:25:06+01:00
draft: false
---

To shim strace into service startup so it gets started with correct permissions
and environment:
```
service irq_balancer /system/bin/strace -o /data/local/tmp/msm_irqb.strace
/odm/bin/irq_balancer -f /vendor/etc/irq_balancer.conf
    socket irq_balancer seqpacket 0660 root root
    user root
    group root
    disabled
```

Remount `/system` as read-write, then:
```
chcon u:object_r:irq_balancer_exec:s0 /system/bin/strace
```

For more information, see the [article on strace at Android source][strace].

[strace]: https://source.android.com/devices/tech/debug/strace
