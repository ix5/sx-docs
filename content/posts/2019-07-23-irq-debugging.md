---
title: "Kernel IRQ Debugging"
description: ""
date: 2019-07-23T21:59:55+02:00
draft: true
---

https://github.com/kholk/kernel/commits/232r14-wakeup-reasons
```
CONFIG_DEDUCE_WAKEUP_REASONS=y
```

```
cat /sys/kernel/debug/wakeup_sources
[...]
cat /sys/kernel/wakeup_reasons/last_resume_reason
-1 misconfigured IRQ 75 qcom,glink-smem-native-xprt-rpm
```
