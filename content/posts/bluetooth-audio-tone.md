---
title: "Bluetooth Audio Debugging"
description: "Solving call audio with bluetooth headset"
date: 2019-03-12T08:18:00+01:00
draft: false
bref: "Solving call audio"
author: Felix
---

<!-- ## Angelo -->

<!-- the modifications are 3 lines -->
<!-- ``` -->
<!-- <device name="SND_DEVICE_OUT_BT_SCO_WB" backend="bt-sco-wb" interface="AUX_PCM_RX"/> -->
<!--        <device name="SND_DEVICE_OUT_BT_SCO" backend="bt-sco" interface="AUX_PCM_RX"/> -->
<!-- <device name="SND_DEVICE_OUT_VOICE_TX" backend="afe-proxy" interface="SLIMBUS_0_RX"/> -->
<!-- ``` -->
<!-- to explain the reasons of that -->

<!-- device-sony-kagura mixer_paths.xml -->
<!-- ``` -->
<!-- <path name="voicemmode2-call bt-sco"> -->
<!--        <ctl name="AUX_PCM_RX_Voice Mixer VoiceMMode2" value="1" /> -->
<!-- <path name="voicemmode2-call"> -->
<!--        <ctl name="SLIM_0_RX_Voice Mixer VoiceMMode2" value="1" /> -->
<!-- ``` -->
<!-- voicemmode2-call mode gets audio on the SLIMBUS_0_RX -->
<!-- and in the bt-sco(-wb) usecase (backend) it routes the audio to AUX_PCM_RX -->
<!-- ``` -->
<!-- ./hal/msm8974/platform.c:    hw_interface_table[SND_DEVICE_OUT_BT_SCO] = strdup("SEC_AUX_PCM_RX"); -->
<!-- ``` -->
<!-- in the HAL -->
<!-- this is erroneously being routed by default -->
<!-- to SEC_AUX_PCM_RX instead -->

After the switch from Kernel version 4.4(`LA.UM.6.4.r1`) to
4.9(`LE.UM.2.3.2.r1.4`), the Sony Xperia XZ("kagura") could not output any call
sound at all while a bluetooth headset was connected.

While digging through the `dmesg` logs, some things popped up:
```
qdsp_cvp_callback: cmd = 0x13199 returned error = 0x2
voice_send_cvp_device_channels_cmd: DSP returned error[ADSP_EBADPARAM]
voice_send_cvp_media_fmt_info_cmd: Set channel info failed err: -22
voice_setup_vocproc: Set media format info failed err:-22
setup voice failed
```

The messages are emitted by [q6voice.c][q6voice-le23]. (The `techpack/`
directory contains backports/imports that are not in the regular kernel.)

Let's add some logging to 4.9:
```
2019-01-29-1845-4.9-with-logging-dmesg10.txt
voc_set_device_config: path_dir=0 port_id=100a, channels=1, sample_rate=16000, \
  bits_per_sample=2
voice_send_cvp_create_cmd: veeeery_early: v->dev_rx.no_of_channels = 1
voice_send_cvp_create_cmd: veeeery_early: v->dev_tx.no_of_channels = 1
voice_set_topology_specific_info: Topology Rx no of channels: 15
voice_set_topology_specific_info: Topology Tx no of channels: 15
voice_send_cvp_create_cmd: early: v->dev_rx.no_of_channels = 15
voice_send_cvp_create_cmd: early: v->dev_tx.no_of_channels = 15
tx_topology: 69491 tx_port_id=4107, rx_port_id=4106, mode: 0x10f7c
rx_topology: 69514, profile_id: 0x1135e, pkt_size: 64
voice_send_cvp_create_cmd: late: v->dev_rx.no_of_channels = 15
voice_send_cvp_create_cmd: late: v->dev_tx.no_of_channels = 15
voice_send_cvp_device_channels_cmd: cvp_set_dev_channels_cmd struct follows:
voice_send_cvp_device_channels_cmd: hdr: hdr_field=592, pkt_size=24, src_port=6, \
  dest_port=64, token=0, opcode=78233
voice_send_cvp_device_channels_cmd: cvp_set_channels: \
  rx_num_channels=1, tx_num_channels=15
qdsp_cvp_callback: cmd = 0x13199 returned error = 0x2
voice_send_cvp_device_channels_cmd: DSP returned error[ADSP_EBADPARAM]
voice_send_cvp_media_fmt_info_cmd: Set channel info failed err: -22
voice_setup_vocproc: Set media format info failed err:-22
setup voice failed
```
...and compare with 4.4:
```
2019-01-29-1845-4.4-with-logging-dmesg.txt
voice_send_cvp_device_channels_cmd: cvp_set_dev_channels_cmd struct follows:
voice_send_cvp_device_channels_cmd: hdr: hdr_field=592, pkt_size=24, src_port=6, \
  dest_port=64, token=0, opcode=78233
voice_send_cvp_device_channels_cmd: cvp_set_channels: \
  rx_num_channels=1, tx_num_channels=1
```
We can immediately see something is odd with `rx_num_channels` and
`tx_num_channels`.
<!-- some weirdness with 15 being converted to 1 or 2 -->

Hardcoding `tx_num_channels` to `1` seems to temporarily solve the issue, we
have call audio again.

But that's not a good solution, let's dig a bit deeper:  
The number of channels is frozen on 4.4, but starting on 4.9
the kernel loads voice topology information via
[voc_get_tx_rx_topology()][le23-topology] - including the number of channels -
from the ACDB("Audio Calibration Database").
ACDB is a proprietary mechanism, and we cannot modify the binary acdb files
without specialized tools.

It seems the calibration data, at least in respect to bluetooth voice channels,
is off for legacy "tone" devices such as the Xperia XZ("kagura"), and possibly
"loire" too.

So we arrive at the next idea: Only read topology on nile, tama and newer
platforms(where the ACDB is correct) via `of_machine_is_compatible`:
[Do not load bt audio topology on legacy devices][tp-no-legacy].

But that's also not a very good idea since we are now nuking all ACDB topology
info in `q6voice` for legacy when only bluetooth seems wrongly configured.

The final decision was to do it more elegantly: Just disable the `acdb_id` for
`OUT_BT_SCO_WB` in the legacy platforms' `audio_platform_info.xml` files.

See:

- [tone: audio_platform_info: Disable ACDB for OUT_BT_SCO_WB][acdb-pull-tone]
- [yoshino: audio_platform_info: Disable ACDB for OUT_BT_SCO_WB][acdb-pull-yoshino]

<strike>But there is still some way to go, when switching from bluetooth to
earpiece/speaker the sound is gone irrevocably, one needs to restart call to
restore it.</strike>
It seems the issue is completely gone with the v6 Software Binaries realease.

---

<!-- Also: [GCC 8 support, SMP2P RDBG disablement, clock fixes, audio fix][pull-audiofix] -->
<!-- with commit -->
<!-- [tp: audio: Don't care about reusing calibrations for other usecases][tp-audio-commit] -->


[q6voice-le23]: https://github.com/sonyxperiadev/kernel/blob/aosp/LE.UM.2.3.2.r1.4/techpack/audio/dsp/q6voice.c
[le23-topology]: https://github.com/sonyxperiadev/kernel/blob/2637f52c94db33b498c6cc6c01df04fd2dde677c/techpack/audio/dsp/q6voice.c#L2421
[tp-no-legacy]: https://github.com/ix5/kernel-sony/commit/42dd56ac06b115ae228567ab10c94f86010f5061
[acdb-pull-tone]: https://github.com/sonyxperiadev/device-sony-tone/pull/163
[acdb-pull-yoshino]: https://github.com/sonyxperiadev/device-sony-yoshino/pull/124
<!-- [pull-audiofix]: https://github.com/sonyxperiadev/kernel/pull/1909 -->
<!-- [tp-audio-commit]: https://github.com/sonyxperiadev/kernel/pull/1909/commits/c4966d1ce6d4127f52619c1d1bbaf0c4b5383cf5 -->
