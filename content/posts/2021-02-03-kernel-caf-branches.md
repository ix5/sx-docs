---
title: "Kernel Caf Branches"
date: 2021-02-03T11:44:03+01:00
draft: true
---

**INTERNAL:** Overview of current CAF kernel/userspace branch candidates

- 4.19
======

LAHAINA lahaina SM8350 888 5G -ENOSONY
- Kernel: Not yet released

SMxx49 kona SM8250 865 5G edo
- Kernel: https://source.codeaurora.org/quic/la/kernel/msm-4.19/tag/?h=LA.UM.9.12.r1-10000-SMxx50.0

SAIPAN lito SM7250-AB 765G 5G -ENOSONY
- Kernel: https://source.codeaurora.org/quic/la/kernel/msm-4.19/tag/?h=LA.UM.8.13.r1-11200-SAIPAN.0

KAMORTA bengal SM6115 662 -ENOSONY
- Kernel: https://source.codeaurora.org/quic/la/kernel/msm-4.19/tag/?h=LA.UM.9.15.1.r1-01900-KAMORTA.0

MANNAR holi SM4350 480 5G -ENOSONY
- Kernel: Not yet released

- 4.14
======

SMxxx0 msmnile SM8150 855 kumano
- Kernel: https://source.codeaurora.org/quic/la/kernel/msm-5.14/tag/?h=LA.UM.9.1.r1-08300-SMxxx0.0

NICOBAR trinket SM6125 665 seine
- Kernel: https://source.codeaurora.org/quic/la/kernel/msm-4.14/tag/?h=LA.UM.9.11.r1-02600-NICOBAR.0

## WIFI Drivers

```
Initial merge:

git remote add qcacld-3.0 https://source.codeaurora.org/quic/la/platform/vendor/qcom-opensource/wlan/qcacld-3.0
git fetch qcacld-3.0 <TAG>git merge -s ours --no-commit --allow-unrelated-histories FETCH_HEAD
git read-tree --prefix=drivers/staging/qcacld-3.0 -u FETCH_HEAD
git commit

Updating to a newer tag:
git fetch qcacld-3.0 <TAG>
git merge -X subtree=drivers/staging/qcacld-3.0 FETCH_HEAD

Repeat the above for 
qca-wifi-host-cmn  and fw-api as well.

qcacld-3.0 source: https://source.codeaurora.org/quic/la/platform/vendor/qcom-opensource/wlan/qcacld-3.0
fw-api source: https://source.codeaurora.org/quic/la/platform/vendor/qcom-opensource/wlan/fw-api
qca-wifi-host-cmn source: https://source.codeaurora.org/quic/la/platform/vendor/qcom-opensource/wlan/qca-wifi-host-cmn
```
