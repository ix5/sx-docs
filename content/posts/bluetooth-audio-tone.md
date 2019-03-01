---
title: "Bluetooth audio debugging"
date: 2019-01-31T08:18:00+01:00
draft: true
author: Felix
---

- kernel: techpack audio vs normal
- 4.4 wasn't checking topology
- 4.9 is checking topology, but the tx number is wrong(15 instead of 1)
- some weirdness with 15 being converted to 1 or 2
- approach: only read topology on nile, tama and
  newer(`of_machine_is_compatible`):
  [Do not load bt audio topology on legacy devices][tp-no-legacy]
- ultimate decision to just invalidate the acdb id for bt audio

the modifications are 3 lines
```
<device name="SND_DEVICE_OUT_BT_SCO_WB" backend="bt-sco-wb" interface="AUX_PCM_RX"/>
       <device name="SND_DEVICE_OUT_BT_SCO" backend="bt-sco" interface="AUX_PCM_RX"/>
<device name="SND_DEVICE_OUT_VOICE_TX" backend="afe-proxy" interface="SLIMBUS_0_RX"/>
```
to explain the reasons of that

device-sony-kagura mixer_paths.xml
```
<path name="voicemmode2-call bt-sco">
       <ctl name="AUX_PCM_RX_Voice Mixer VoiceMMode2" value="1" />
<path name="voicemmode2-call">
       <ctl name="SLIM_0_RX_Voice Mixer VoiceMMode2" value="1" />
```
voicemmode2-call mode gets audio on the SLIMBUS_0_RX
and in the bt-sco(-wb) usecase (backend) it routes the audio to AUX_PCM_RX
```
./hal/msm8974/platform.c:    hw_interface_table[SND_DEVICE_OUT_BT_SCO] = strdup("SEC_AUX_PCM_RX");
```
in the HAL
this is erroneously being routed by default
to SEC_AUX_PCM_RX instead

See:

- [tone: audio_platform_info: Disable ACDB for OUT_BT_SCO_WB][acdb-pull-tone]
- [yoshino: audio_platform_info: Disable ACDB for OUT_BT_SCO_WB][acdb-pull-yoshino]

---

Also: [GCC 8 support, SMP2P RDBG disablement, clock fixes, audio fix][pull-audiofix]
with commit
[tp: audio: Don't care about reusing calibrations for other usecases][tp-audio-commit]


[tp-no-legacy]: https://github.com/ix5/kernel-sony/commit/42dd56ac06b115ae228567ab10c94f86010f5061
[acdb-pull-tone]: https://github.com/sonyxperiadev/device-sony-tone/pull/163
[acdb-pull-yoshino]: https://github.com/sonyxperiadev/device-sony-yoshino/pull/124
[pull-audiofix]: https://github.com/sonyxperiadev/kernel/pull/1909
[tp-audio-commit]: https://github.com/sonyxperiadev/kernel/pull/1909/commits/c4966d1ce6d4127f52619c1d1bbaf0c4b5383cf5
