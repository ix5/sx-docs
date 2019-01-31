the modifications are 3 lines
<device name="SND_DEVICE_OUT_BT_SCO_WB" backend="bt-sco-wb" interface="AUX_PCM_RX"/>
       <device name="SND_DEVICE_OUT_BT_SCO" backend="bt-sco" interface="AUX_PCM_RX"/>
<device name="SND_DEVICE_OUT_VOICE_TX" backend="afe-proxy" interface="SLIMBUS_0_RX"/>
tell me that it works :stuck_out_tongue:
to explain the reasons of that
device-sony-kagura mixer_paths.xml
<path name="voicemmode2-call bt-sco">
       <ctl name="AUX_PCM_RX_Voice Mixer VoiceMMode2" value="1" />
<path name="voicemmode2-call">
       <ctl name="SLIM_0_RX_Voice Mixer VoiceMMode2" value="1" />
voicemmode2-call mode gets audio on the SLIMBUS_0_RX
and in the bt-sco(-wb) usecase (backend) it routes the audio to AUX_PCM_RX
./hal/msm8974/platform.c:    hw_interface_table[SND_DEVICE_OUT_BT_SCO] = strdup("SEC_AUX_PCM_RX");
in the HAL
this is erroneously being routed by default
to SEC_AUX_PCM_RX instead
