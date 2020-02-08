---
title: "Quick: Kernel dts Tools"
description: ""
date: 2020-02-04T21:37:35+01:00
draft: false
---

Get a plain-text file from a generated `.dtb`:
```
dtc \
  arch/arm64/boot/dts/qcom/msm8996-v3.0-pmi8996-tone-kagura_generic.dtb \
  > msm8996-v3.0-pmi8996-tone-kagura_generic-generated.dts
```

The generated `.dts` file is in plain-text and can be viewed in a text editor.

More: [eLinux: Device Tree Usage][elinux].

[elinux]: https://elinux.org/Device_Tree_Usage
