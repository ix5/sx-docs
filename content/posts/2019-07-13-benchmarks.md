---
title: "Benchmarks with Dhrystone"
description: ""
date: 2019-07-13T19:17:46+02:00
draft: true
---

https://github.com/tomari/NDKdhry/

Set rqbalance to all cores

```
echo "performance" > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor
echo "performance" > /sys/devices/system/cpu/cpufreq/policy2/scaling_governor
chown root:root /sys/devices/system/cpu/cpufreq/policy0/scaling_governor
chown root:root /sys/devices/system/cpu/cpufreq/policy2/scaling_governor
chown root:root /sys/devices/system/cpu/cpuquiet/nr_min_cpus
chown root:root /sys/devices/system/cpu/cpuquiet/nr_max_cpus
chown root:root /sys/devices/system/cpu/cpuquiet/nr_power_max_cpus
chown root:root /sys/devices/system/cpu/cpuquiet/nr_thermal_max_cpus
```

