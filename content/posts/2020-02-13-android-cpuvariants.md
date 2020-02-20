---
title: "Android CPU Variants"
description: "Switch to generic?"
date: 2020-02-13T13:55:27+01:00
draft: false
---

See [CpuVariantNames in bionic's libc][cpuvariantnames]:
```
static constexpr CpuVariantNames cpu_variant_names[] = {
    {"cortex-a76", kCortexA55},
    {"kryo385", kCortexA55},
    {"cortex-a75", kCortexA55},
    {"kryo", kKryo},
    {"cortex-a73", kCortexA55},
    {"cortex-a55", kCortexA55},
    {"cortex-a53", kCortexA53},
    {"krait", kKrait},
    {"cortex-a9", kCortexA9},
    {"cortex-a7", kCortexA7},
    // kUnknown indicates the end of this array.
    {"", kUnknown},
};
```

That means:

- `kryo` is treated as `Kryo`
- `cortex-a53` is treated as `Cortex A53`
- `kryo385`, `cortex-a55`, `cortex-a73`, `cortex-a75` and `cortex-a76` are all
  treated as a `Cortex A55`.
- `krait` is treated as `Krait`
- `cortex-a9` is treated as `Cortex A9`
- `cortex-a7` is treated as `Cortex A7`

## Generic and runtime variants

We adjusted our makefiles as follows for an `msm8996` device:
```diff
 TARGET_ARCH := arm64
 TARGET_ARCH_VARIANT := armv8-a
 TARGET_CPU_ABI := arm64-v8a
 TARGET_CPU_ABI2 :=
-TARGET_CPU_VARIANT := kryo
+TARGET_CPU_VARIANT := generic
+TARGET_CPU_VARIANT_RUNTIME := kryo

 TARGET_2ND_ARCH := arm
 TARGET_2ND_ARCH_VARIANT := armv8-a
 TARGET_2ND_CPU_ABI := armeabi-v7a
 TARGET_2ND_CPU_ABI2 := armeabi
-TARGET_2ND_CPU_VARIANT := kryo
+TARGET_2ND_CPU_VARIANT := generic
+TARGET_2ND_CPU_VARIANT_RUNTIME := kryo
```

Similar for an `sdm845` device:
```diff
 TARGET_ARCH := arm64
-TARGET_ARCH_VARIANT := armv8-2a
+TARGET_ARCH_VARIANT := armv8-a
 TARGET_CPU_ABI := arm64-v8a
 TARGET_CPU_ABI2 :=
-TARGET_CPU_VARIANT := kryo385
+TARGET_CPU_VARIANT := generic
+TARGET_CPU_VARIANT_RUNTIME := kryo385

 TARGET_2ND_ARCH := arm
-TARGET_2ND_ARCH_VARIANT := armv8-2a
+TARGET_2ND_ARCH_VARIANT := armv8-a
 TARGET_2ND_CPU_ABI := armeabi-v7a
 TARGET_2ND_CPU_ABI2 := armeabi
-TARGET_2ND_CPU_VARIANT := kryo385
+TARGET_2ND_CPU_VARIANT := generic
+TARGET_2ND_CPU_VARIANT_RUNTIME := kryo385
```

### Links

- https://android.googlesource.com/device/google/crosshatch/+/302a36e5192d5821e42dc010945a1d2c4fe784c6%5E%21/#F0
- https://android.googlesource.com/device/google/coral/+/2c479a98ab3a6158bfb0050d7d1766c90c496def%5E%21/
- https://android.googlesource.com/device/google/crosshatch/+/ed4abfb7bc3c48039985b09609880192d5981fd8%5E%21/#F0
- https://android.googlesource.com/device/google/crosshatch/+/37f229e80db6abaff3cf437bbe2037ca01f7e56b%5E!/#F0
- https://android.googlesource.com/device/google/crosshatch/+/5e58194bb5b96cbe9f031bda2dbc7488480ba999%5E%21/#F0
- https://android.googlesource.com/device/google/crosshatch/+/64a47c6390b3bfae47b0118568a9c2d51f1a541a%5E!/#F0
- https://android.googlesource.com/device/google/crosshatch/+/4c075f2a5426cc6975c6fb8b2cf079150b10e13c%5E!/#F0
- https://android.googlesource.com/platform/build/soong/+/d3072b0c7cef7f5a217a055e66d85890c78620bc
- https://android.googlesource.com/platform/build/soong/+/refs/tags/android-10.0.0_r29/cc/config/arm64_device.go#30
- https://android.googlesource.com/platform/build/soong/+/refs/tags/android-10.0.0_r29/cc/config/arm_device.go#53

[cpuvariantnames]: https://android.googlesource.com/platform/bionic/+/refs/tags/android-10.0.0_r29/libc/arch-arm/dynamic_function_dispatch.cpp#51
