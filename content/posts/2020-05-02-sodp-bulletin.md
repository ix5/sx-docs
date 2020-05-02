---
title: "SODP 2020-04-05 Security state"
description: ""
date: 2020-05-02T21:11:11+01:00
draft: false
---

Reference: [2020-04-01 Google Bulletin][bulletin]

## Kernel

**Input: ff-memless - kill timer in destroy()**
[https://git.kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=fa3a5a1880c91bb92594ad42dfe9eedad7996b86)  
Already merged in 4.9 via 4f039b4fc7d3a "treewide: Linux 4.9.203"  
Already merged in 4.14 via 07063e00169b5 "treewide: Linux 4.14.155"

**HID: Fix assumption that devices have inputs**
[https://git.kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d9d4b1e46d9543a82c23f6df03f4ad697dab361b)  
Already merged in 4.9 via d0f1e408899a4 "treewide: Linux 4.9.199"  
Already merged in 4.14 via 0e329f9e62b98 "treewide: Linux 4.14.152"

**ALSA: timer: Fix incorrectly assigned timer instance**
[https://git.kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e7af6307a8a54f0b873960b32b6a644f2d0fbd97)  
Already merged in 4.9 via 69c2d155bb0ce "treewide: Linux 4.9.201"  
Already merged in 4.14 via b3cdc9dadb7ae "treewide: Linux 4.14.154"

## CAF Kernel

**asoc: msm-pcm: Added lock in controls put() and get()**
[source.codeaurora.org/quic/la/kernel/msm-4.4/commit?id=1c9afa](https://source.codeaurora.org/quic/la/kernel/msm-4.4/commit?id=1c9afab264e3cafa461d746e9dcfd3c0487754cb)  
4.9: TODO  
4.14: TODO

**soc: msm-pcm: Add mutex lock to protect prvt data**
[source.codeaurora.org/quic/la/kernel/msm-4.4/commit/?id=42ffbf](https://source.codeaurora.org/quic/la/kernel/msm-4.4/commit/?id=42ffbf03ec54dc14824d078f36c350b24c217f8d)
4.9: TODO  
Already fixed in 4.14 in kernel-techpack-audio via f2a2905f "asoc: msm-pcm: Add mutex lock to protect prvt data"

**msm: camera: context: Add null check on context pointer**
[source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=9910e8](https://source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=9910e89b27224fbddbf7d15d307597e13d9b9258)  
4.9: TODO  
4.14: TODO

**net: qrtr: Handle error from skb_put_padto**
[source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=870f0](https://source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=870f0ba0fc05bc6ebac1486b39dc9d94c993eafb)  
4.9: TODO  
4.14: TODO

**vdec: Set correct output buffer size: dynamic meta mode**
[source.codeaurora.org/quic/le/platform/hardware/qcom/media/commit/?id=9e80e](https://source.codeaurora.org/quic/le/platform/hardware/qcom/media/commit/?id=9e80e1db4b56b42f9150d4d51166560d10839f5f)  
4.9: TODO  
4.14: TODO

## Framework and System
Patched via sync of latest AOSP sources.

[bulletin]: https://source.android.com/security/bulletin/2020-04-01
