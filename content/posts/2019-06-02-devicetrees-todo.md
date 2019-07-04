---
title: "SODP Device Trees TODO"
description: ""
date: 2019-06-02T15:00:00+02:00
draft: false
---

## What do we want? What can we do?
- Stability
- Better battery life
- More performance
- Fix bugs
- Better sound
- Better network throughput
- Faster storage access
- More free RAM thanks to fixing memory leaks or better memory management

## Where can we improve things?
- Kernel
- Firmware (unlikely)
- HALs
- Properties
- Configuration files
- Write replacements for Qualcomm HALs or proprietary binaries

# Device Trees
Things we can set in the device trees

Some things overlap of course, but not too much.

### Audio
- Config files:
  - `a2dp_audio_policy_configuration.xml`
  - `audio_effects.conf`
  - `audio_ext_spkr.conf`
  - `audio_output_policy.conf`
  - `audio_platform_info.xml`
  - `audio_platform_info_extcodec.xml`
  - `audio_platform_info_i2s.xml`
  - `audio_policy_configuration`
  - `audio_policy_volumes_drc.xml`
  - `audio_tuning_mixer.txt`
  - `mixer_paths.xml`
  - `mixer_paths_mtp.xml`
- check TX/RX paths
- Soundtrigger/hotword detection:
  - `sound_trigger_mixer_paths.xml`
  - `sound_trigger_platform_info.xml`
- ACDB tweaks
- Configure Qualcomm Fluence, voice processing
- Tweak latencies:
  "nile-common: audio: Disable ULL mode"
  https://github.com/omnirom/android_device_sony_nile-common/commit/d9ff18ec6414595481ed5d4d44648c6c7b61a6b8
  https://github.com/LineageOS/android_device_sony_nile-common/commit/4dc8401ba95a682e61585f563d78392d5b35df2c

### Bluetooth
- Offloading, offloading HALs
- A2DP implementation
- Enable all possible codecs
- LDAC
- `bdroid_buildconfig.h`
- Low-Energy
- Channels

### Block Devices
- Tune fstab
- Maybe use f2fs

https://github.com/lazerl0rd/device-sony-yoshino/commit/c6a9a94a88e85ac16c759990ff42eebfb888162b#diff-2a70d4c28762e80752b2d8d1ac74dbfe

### GPS
- `gps.conf`
- Check servers
- Check `XTRA` versions

### NFC
- Configs
- Check that models are correct

### Sensors
- [Angler: sensorlist.cpp][angler-sensorlist]

### Debugging features
- Add our custom RPM etc. paths to the dump output of the `dumpstate` HAL
- [Angler: dumpstate][angler-dumpstate]

### Treble
- Update DM, FCM versions
- Update HAL versions of default ones
- Update `radio@1.1` to 1.4
- Check that all devices go on `PRODUCT_FULL_TREBLE`

### Wi-Fi
- Configs
  - `p2p_supplicant_overlay.conf`
  - `wpa_supplicant_overlay.conf`
  - `wpa_supplicant_wcn.conf`
  - `wifi_concurrency_cfg.txt`
  - `WCNSS_qcom_cfg.ini`
- Configure usable frequencies(DFS)

### sec_config
https://github.com/LineageOS/android_device_google_bonito/blob/lineage-16.0/sec_config
https://github.com/PixelExperience-Devices/device_xiaomi_tulip/blob/pie/configs/sec_config

### seccomp
https://github.com/omnirom/android_device_motorola_potter/tree/android-9.0/seccomp_policy

### Suspend HAL?
`config_powerDecoupleAutoSuspendModeFromDisplay`
`config_powerDecoupleInteractiveModeFromDisplay`
`overlay/frameworks/base/core/res/res/values/config.xml`
https://github.com/omnirom/android_device_sony_nile-common/commit/c62a3be7b01f6a4f5da7f497d00c024e30448f96

### msm_irqbalance
- Confirm whether IRQs are actually correct
- Ban/unban if needed
- Implement OSS irqbalance

https://github.com/PixelExperience-Devices/device_xiaomi_tulip/blob/pie/configs/msm_irqbalance.conf

# HALs/Blobs
Things that we have to tweak HALs for or try to get proprietary binaries changed
or write our own replacements.

### Camera
vendor-qcom-opensource-camera:

- Shutter speed, exposure, whitebalance etc. parameters
- Fix CASH with timeouts instead of shutting off sensors
- Flash and shutter coordination


# Kernel
- Loire: instabilities
- Tone: 5s bug, crashy mtp, heat on even low loads, bad battery life
- Yoshino: Buggy SDE (why does it only happen on GSIs?)
- Nile: Buggy SDE/sluggish stuttery display
- Tama: (Camera)


# Update forked HALs
TODO: List of forked HALs that we should update from CAF

# TODO/Inspiration

- https://github.com/LineageOS/android_device_huawei_angler/
- https://github.com/cryptomilk/android_device_sony_lilac/blob/lineage-16.0/system.prop
- https://github.com/omnirom/android_device_xiaomi_beryllium
- https://github.com/ResurrectionRemix-Devices/device_lenovo_kuntao/blob/pie/vendor_prop.mk
- https://github.com/ResurrectionRemix-Devices/device_lenovo_kuntao/blob/pie/bluetooth/bdroid_buildcfg.h
- https://github.com/ResurrectionRemix-Devices/device_lenovo_kuntao/tree/pie/audio
- https://github.com/omnirom/android_device_motorola_potter/blob/android-9.0/system.prop
- https://github.com/omnirom/android_device_motorola_potter/blob/android-9.0/device.mk
- https://github.com/omnirom/android_device_motorola_potter/tree/android-9.0/audio
- `compress_offload`:
- https://github.com/omnirom/android_device_sony_nile-common/blob/8edcbb4cebd597e39ccaa844ab25bc01d156ac2e/audio/audio_policy_configuration.xml#L92
- https://github.com/LineageOS/android_device_essential_mata
- https://github.com/omnirom/android_device_oneplus_oneplus5
- https://github.com/LineageOS/android_device_google_crosshatch
- https://github.com/LineageOS/android_device_google_wahoo
- https://github.com/LineageOS/android_device_google_bonito
- https://github.com/LineageOS/android_device_xiaomi_dipper
- https://github.com/PixelExperience-Devices/device_xiaomi_tulip/tree/pie/configs/gps
- https://github.com/PixelExperience-Devices/device_xiaomi_tulip/blob/pie/bluetooth/bdroid_buildcfg.h
- https://github.com/PixelExperience-Devices/device_xiaomi_tulip/commits/pie/audio
- https://github.com/enesuzun2002/android_device_samsung_zero-common
- https://github.com/MoKee/android_device_xiaomi_cepheus

## Misc
- Re-enable Android Beam on Q

[angler-sensorlist]: https://github.com/LineageOS/android_device_huawei_angler/blob/lineage-16.0/sensorhal/sensorlist.cpp
[angler-dumpstate]: https://github.com/LineageOS/android_device_huawei_angler/blob/lineage-16.0/dumpstate/DumpstateDevice.cpp
