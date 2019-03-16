---
title: "Governor Debugging"
date: 2019-03-01T17:41:48+01:00
draft: true
---

Put all the CPU and GPU cores in `performance` governor mode, then, as `root`:
```
for f in /sys/class/devfreq/*; do echo "performance" > "$(realpath $f)/governor"; done
```
Then confirm it worked with:
```
for f in /sys/class/devfreq/*; do cat "$(realpath $f)/governor"; done
```

```
chown root:root /sys/devices/system/cpu/cpuquiet/*
chown root:root /sys/devices/system/cpu/cpuquiet/rqbalance/*
```
