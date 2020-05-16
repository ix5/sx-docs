---
title: "SODP 2020-02-05 Security state"
description: ""
date: 2020-02-08T21:11:11+01:00
draft: false
---

Reference: [2020-02-01 Google Bulletin][bulletin]

## Kernel

**ANDROID: binder: synchronize_rcu() when using POLLFREE**  
[android.googlesource.com/kernel/common/+/5eeb2ca0](https://android.googlesource.com/kernel/common/+/5eeb2ca0)  
Already merged in 4.9  
Already merged in 4.14

**coredump: fix race condition between mmget_not_zero()/get_task_mm() and core dumping**  
[android.googlesource.com/kernel/common/+/04f586](https://android.googlesource.com/kernel/common/+/04f5866e41fb70690e28397487d8bd8eea7d712a)  
Already merged in 4.9  
Already merged in 4.14

**ANDROID: fix binder change in merge of 4.9.188**  
[android.googlesource.com/kernel/common/+/3378ce](https://android.googlesource.com/kernel/common/+/3378ce511d7a792ddf0d69d11ce5e284309893fd)  
Fixed for 4.9 via 232r1-security-2020-02-05-binder  
Does not affect 4.14

## CAF Kernel

**msm: camera: isp: use correct number of entries**  
[source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=a2198a](https://source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=a2198a95c0a1a319bb6d7ed9fefa1b5e905e6418)  
Already merged in 4.9  
Already merged in 4.14 (copypasted from 4.9)

**ion: Ensure non-HLOS memory cannot be mapped by CPU**  
[source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=311420](https://source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=31142010ccaf6ddad331a7919a7fbf3da80b8359)  
Already merged in 4.9  
Already merged in 4.14

**diag: Mark Buffer as NULL after freeing**  
[source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=c0cb07](https://source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=c0cb074983060d6ff46e312a9db81fde869cd63b)  
Already merged in 4.9  
Already merged in 4.14

**ASoC: bolero: check for port validation before configuration**  
[source.codeaurora.org/quic/la/platform/vendor/opensource/audio-kernel/commit/?id=bab05c](https://source.codeaurora.org/quic/la/platform/vendor/opensource/audio-kernel/commit/?id=bab05c57cb51aee957a1fe926c7d3c54378acb6a)  
Not affected, we don't have or even ship the bolero codec

**msm: kgsl: Verify the offset of the profiling buffer**  
[source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=fb37ff](https://source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=fb37ff663a3d28e3a07549b074c54feb3e4376b5)  
Fixed for 4.9 via 232r1-security-caf  
Already merged in 4.14 via "Fast forward Adreno driver to LA.UM.8.1.r1-11600-sm8150.0"

**msm: kgsl: Use a bitmap allocator for global addressing**  
[source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=a8acbcb0](https://source.codeaurora.org/quic/la/kernel/msm-4.14/commit/?id=a8acbcb0)  
Fixed for 4.9 via 232r1-security-caf  
Already merged in 4.14 via "Fast forward Adreno driver to LA.UM.8.1.r1-11600-sm8150.0"

**msm: kgsl: Execute user profiling commands in an IB**  
[source.codeaurora.org/quic/la/kernel/msm-4.4/commit/?id=1aa9c6](https://source.codeaurora.org/quic/la/kernel/msm-4.4/commit/?id=1aa9c68484c49a7357c0835b38fa1581bb7d6865)  
Fixed for 4.9 via 232r1-security-caf  
Already merged in 4.14 via "Fast forward Adreno driver to LA.UM.8.1.r1-11600-sm8150.0"

**msm: camera: icp: Fix out of bound access issue in ICP**  
[source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=9c3d81](https://source.codeaurora.org/quic/la/kernel/msm-4.9/commit/?id=9c3d819d2d9563779fd1daa4eefef8628de22a86)  
Already merged in 4.9  
Already merged in 4.14

## Framework and System
Patched via sync of latest AOSP sources.

[bulletin]: https://source.android.com/security/bulletin/2020-02-01
