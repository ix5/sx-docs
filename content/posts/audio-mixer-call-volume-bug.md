---
title: "Audio Mixer Bug"
description: "Changing Call Volume suddenly fixes the bottom speaker?"
date: 2019-03-17T20:54:13+01:00
draft: true
---


When changing call volume:
```
audio_route: Apply path: handset
audio_hw_primary: enable_audio_route: usecase(1) apply and update mixer path: low-latency-playback
audio_route: Apply path: low-latency-playback
audio_hw_primary: disable_audio_route: usecase(1) reset and update mixer path: low-latency-playback
```

So the `handset` path leads to different settings being applied.


(only kanged from `handset`:)
```
<path name="compress-offload-playback">
    <ctl name="SLIMBUS_0_RX Audio Mixer MultiMedia4" value="1" />
    <ctl name="SLIM RX0 MUX" value=" AIF_MIX1_PB" />
    <ctl name="SLIM RX1 MUX" value="AIF_MIX1_PB" />
    <ctl name="RX INT7_1 MIX1 INP0" value="RX0" />
    <ctl name="RX INT8_1 MIX1 INP0" value="RX1" />
    <ctl name="AIF4_VI Mixer SPKR_VI_1" value="1" />
    <ctl name="AIF4_VI Mixer SPKR_VI_2" value="1" />
    <ctl name="RX INT7 INTERP RX INT7" value="MIX2" />
    <ctl name="RX INT8 INTERP RX INT8 SEC" value="MIX" />
    <ctl name="SLIMBUS_0_RX Audio Mixer MultiMedia5" value="1" />
</path>
```

Regular boot:
```
adb shell "echo 1271 > /sys/class/gpio/export"
cat /sys/class/gpio/gpio1271/value
1
```

After boot:
```
adb shell "tinymix 'PM8996_Ear_Enable_States' '0'"
adb shell "echo 1271 > /sys/class/gpio/export"
cat /sys/class/gpio/gpio1271/value
```

Another boot:
```
adb shell "tinymix 'PM8996_Ear_Enable_States' '1'"
adb shell "tinymix 'PM8996_Ear_Enable_States' '0'"
adb shell "echo 1271 > /sys/class/gpio/export"
cat /sys/class/gpio/gpio1271/value
```

Another boot:
```
adb shell "tinymix 'PM8996_Ear_Enable_States' '0'"
adb shell "tinymix 'PM8996_Ear_Enable_States' '1'"
adb shell "echo 1271 > /sys/class/gpio/export"
cat /sys/class/gpio/gpio1271/value
```

Felix
 
ok this was fairly quick
Removing PM8996_Ear_Enable_States means handset no longer toggles the second speaker
Obeida Shamoun
 
ok now just need to trigger it on and off using tinymix
Felix
 
removing both RX0 Digital Volume and PM8996_Ear_Enable_States means speaker is no longer activated
Obeida Shamoun
 
let's see what happens
Felix
 
yup
no, doesn't do anything
grr
Obeida Shamoun
 
ok we need to rule out somehow that it's ctls
can you manually change ctl values in the same order as in handset, then change them back?
Felix
 
there is an audible interruption in music playback when I change call volume
so I really suspect sth else
but @oshmoun I'll give the manual method a try
Obeida Shamoun
 
hold up, does the second speaker work after a call?
Felix
 
but wait no, I did that yesterday already
Obeida Shamoun
 
a call where you don't change volume at all
Felix
 
you mean only after placing a call without volume change?
ah yeah
will try

Obeida Shamoun
 
starting to suspect it's about having playback that only utilizes the one speaker
Felix
 
dangit, I made a mistake obi
forgot to disable the ctl for Ear-enable-states again
when I first switch it to 1 and then back to 0 it enables the second speaker
so there we have our issue
Felix
 
Ok, "fixed" the issue in a hack-ish way
let speaker enable ear-enable-states
and compress-offload-playback disable it again
that way the gpio is triggered(by speaker, which is applied first) but the path is disabled again
Obeida Shamoun
 
ok great
so it was a ctl
let's see what it does in kernel
Felix
 
https://github.com/sonyxperiadev/kernel/blob/aosp/LE.UM.2.3.2.r1.4/techpack/audio/asoc/msm8996.c#L1156
Obeida Shamoun
 
ok so need to flip the gpio on then off and then it works fine?
Felix
 
yup
Obeida Shamoun
 
then I think the issue is that its initial state is on
Felix
 
also tried setting the ear-enable-states to enabled in the base config, but then you get no sound
no the initial state is off
Obeida Shamoun
 
how do you know?
checked dt?
Felix
 
https://github.com/sonyxperiadev/device-sony-kagura/blob/master/rootdir/vendor/etc/mixer_paths.xml#L504
Obeida Shamoun
 
actually, you can tell easier by just exporting the gpio
not sure that gets applied
since the kernel will report that it is already in that state
you should just export the gpio in question and check its value directly :smile:
Felix
 
how do I activate all the pr_debug stuff in a file agian?
Obeida Shamoun
 
-DDEBUG in Makefile I think
Felix
 
no actually the statement in the sound function will no help me
also jesus I don't want every debug feature on
Obeida Shamoun
 
hehe just replace the ones you want
Felix
 
how do I get the gpio state before the function toggles it?
Obeida Shamoun
 
/d/gpio/export
but you need to find out the gpio number
should be possible with some pr_err
then you do echo <gpio> > /d/gpio/export
then you can just cat /d/gpio/<gpio>

I think that's how it went
if not, then pretty similar to that
Marijn
 
@Felix file_name.c_CFLAGS := -DDEBUG if just for that file, not for everything in the makefile
But that's probably still too much given the extreme size of the audio "drivers"
Felix
 
obi your suspicion was right
it seems it was indeed enabled
```
[  128.627355] q6asm_send_cal: cal_block is NULL
[  128.854593] pm8996_ear_enable_put: setting ear_en_gpio=1271 to 0
[  128.994857] afe_get_cal_topology_id: [AFE_TOPOLOGY_CAL] not initialized for this port 16384
[  128.995401] afe_get_cal_topology_id: [AFE_TOPOLOGY_CAL] not initialized for this port 16384
[  128.995614] send_afe_cal_type cal_block not found!!
```
put the log statement to log the gpio number at both cases https://github.com/sonyxperiadev/kernel/blob/f06393a81970c0df609f844d3fce54557f0c268d/techpack/audio/asoc/msm8996.c#L1197-L1201
but only one pops up
might've screwed up format strings again though since that gpio(1271) seems a tad high

```diff
diff --git a/techpack/audio/asoc/msm8996.c b/techpack/audio/asoc/msm8996.c
index 9e93edf1b6e5..a5737635db42 100644
--- a/techpack/audio/asoc/msm8996.c
+++ b/techpack/audio/asoc/msm8996.c
@@ -1194,10 +1194,13 @@ static int pm8996_ear_enable_put(struct snd_kcontrol *kcontrol,
         }
         switch (ucontrol->value.integer.value[0]) {
         case 1:
+            /* static inline void gpio_set_value(unsigned int gpio, int value) */
+            pr_err("%s: setting ear_en_gpio=%d to 1\n", __func__, pdata->ear_en_gpio);
             gpio_set_value(pdata->ear_en_gpio, 1);
             break;
         case 0:
         default:
+            pr_err("%s: setting ear_en_gpio=%d to 0\n", __func__, pdata->ear_en_gpio);
             gpio_set_value(pdata->ear_en_gpio, 0);
             break;
         }
```

compared the output of /d/gpio though, some differences https://linediff.com/?id=5c8fee8c687f4be5228b4567
(gpios-before.txt vs gpios-after.txt)

Obeida Shamoun
 
nope it's not necessarily high @Felix
depends on the base
Felix
 
gpios 10, 13 and 127 have changed
description for kagura's 127 gpio is only "NC", whatever that means
Obeida Shamoun
 
I don't get it
127 or 1271?
Felix
 
dangit
125
not 127
Obeida Shamoun
 
but, the dmesg
I'm confused :no_mouth:
Felix
 
dmesg says 1271

Obeida Shamoun
 
then that is the gpio number to export
where did you find the other numbers?
Felix
 
in the linediff of /d/gpio before and after
link's a few lines up
Obeida Shamoun
 
ah oki
Felix
 
125 is TS_INT_N, 10 is MDP_VSYNC_P, 13 is CAM0_MCLK
though I'm even more confused now, did another run and now:
```
[  122.820442] pm8996_ear_enable_put: setting ear_en_gpio=1271 to 1
[  128.854593] pm8996_ear_enable_put: setting ear_en_gpio=1271 to 0
```
Obeida Shamoun
 
why is that weird?
Felix
 
because before I only saw the one who set it to 0
but I was manually scrolling through dmesg so maybe that's why
Felix
 
kinda lost now though, why would the gpio be specified in another base?
this https://developer.qualcomm.com/download/db410c/gpio-pin-assignment.pdf says TS_INT_N is sth for the touchscreen
MDP and vsync stuff must be video
CAM0 sounds like camera, so also odd
Obeida Shamoun
 
pretty sure what you need is really only 1271
just export that, disable it, then try playback again
or, to be extra sure
export it, use tinymix to change the gpio value, then confirm that it did change
Felix
 
obi there is only /d/gpio as a file, there are no files under it
no /d/gpio/export

Obeida Shamoun
 
errrm
aaaaah
/sys/class/gpio/export (edited) 
and you'll also find the bases there
Felix
 
there's no 1271 in there
ah now after the export there is, nice
Felix
 
if I export 1271, the settings fails with pm8996_ear_enable_put: request ear_en_gpio failed, ret:-16
Obeida Shamoun
 
yeah it seems to lock it
-16 is -EBUSY
unexport @Felix
Felix
 
how?
Obeida Shamoun
 
next to export :wink:
Felix
 
whoops :P
Obeida Shamoun
 
comeoncomeoncomeon I can't take it anymore :stuck_out_tongue:
Felix
 
still fails
Obeida Shamoun
 
that is weird
Obeida Shamoun
 
ok then just check it once then reboot @Felix
not the time to figure out how this properly works :stuck_out_tongue:
so check it after boot, then reboot
then check it after enabling ear_state through tinymix, then reboot
then check it after enabling then disabling ear_state through tinymix, then reboot
then reboot once more just for the heck of it :smile:
Felix
 
initial value is 1
Obeida Shamoun
 
gotcha
now to check if there is an explicit set somewhere
I reaaaaaally want it to be a change done by Angelo
@Felix can you try just initializing the variable?
I think it's just that simple
Felix
 
in the kernel?
Obeida Shamoun
 
yup
there is no initial value
so it is random
PLEASE let it be this
this would be the exact same issue as TTC
only in another driver
Felix
 
how do I figure out the mapping to the dtb gpios?
Obeida Shamoun
 
isn't the gpio declared directly in the wcd9335 node?
qcom,ear-en-gpios = <&pm8994_gpios 13 0>;
that's about it
but this is weird, how come this issue didn't appear on 4.4?
wait let's diff
Felix
 
4.4 has qcom,ear-en-gpios = <&pm8994_gpios 13 0>; as well
Obeida Shamoun
 
yeah ofc
was talking about the initialization of pm8996_ear_enable_states :sweat_smile:
it's not initialized on 4.4 as well
so it's weird the issue doesn't happen there, assuming it is indeed that
but let's see what your testing shows
also, did you happen to check suspend times after confirming the speaker works again?
Felix
 
noupe
but I'm pretty sure they're still bad
Obeida Shamoun
 
you can try diffing /d/gpiobetween 4.4 and 4.9
hopefully that points out some exciting differences to check
Felix
 
gonna have to dust off the old omni builds :P
Obeida Shamoun
 
but there is one thing to consider here
this absolutely can not stand
it is utterly unacceptable that tone gets something fixed

Felix
 
anyway, before I forget, where should I initialize the gpio in the dtb? in the tone-common msm_gpio\_\* fields?
what's the parameter to use?
still don't know which real gpio it corresponds to also
Obeida Shamoun
 
I don't think you can set an initial state for gpio
would need to convert to pinctrl to do that
although, the issue could be that the is some pinctrl entry that somehow changes the same gpio
Felix
 
pinctrl is a sodp thing?
Obeida Shamoun
 
nope
https://www.kernel.org/doc/Documentation/pinctrl.txt
i don't understand any of it, but you might if you're an electronics nutcase like Angelo
Felix
 
https://linediff.com/?id=5c900673687f4b9d298b4567
4.4(left) vs 4.9(right, untouched)
(gpios-4.4.txt vs gpios-before.txt)

Obeida Shamoun
 
@Felix not sure there's anything to discern from the diff, just too much stuff
Obeida Shamoun
 
@Felix might have found it
msm8996-tone-common.dtsi
a new node was added on 4.9
somc_pinctrl_pmic
and yup
Felix
 
I see, yeah

Obeida Shamoun
 
```
    /* GPIO 13 - HPH_EN0 - CMOS GPIO Output HIGH, SRC1.8V */
    hph_en0 {
        pm8994_gpio_13: hph0_enable {
            pins = "gpio13";
            function = "normal";
            output-high;
            drive-push-pull;
            bias-disable;
            qcom,drive-strength = <PMIC_GPIO_STRENGTH_LOW>;
            power-source = <PM8994_VPP_1P8>;
        };
    };
```

the state is high
from msm8996-mtp.dtsi
aaaah
wait
in msm8996-tone-common.dtsi

```
/* GPIO_13: EAR_EN */
&pm8994_gpio_13 {
    /* Inherited from MTP */
/*
    output-low;
    drive-push-pull;
    bias-disable;
    qcom,drive-strength = <PMIC_GPIO_STRENGTH_LOW>;
    power-source = <PM8994_VPP_1P8>;
*/
};
```
overwritten, but not really
wanna scold Angelo, here's your chance :smile:
Felix
 
so currently the gpio is just empty
Obeida Shamoun
 
nope, just the same as it always was
this works as an overlay (edited) 
Felix
 
@oshmoun tone is uncursed
the gpio fixed it
5s is next
Obeida Shamoun
 
@Felix great!
Felix
 
just the sound is really shitty
Obeida Shamoun
 
wat
Felix
 
on >50% speaker volume
you can hear it rattling
Obeida Shamoun
 
that is not good
but that was always there right?
can't imagine it is related to the change done just now
Felix
 
I dunno, I just really noticed now
and it's def only the top speaker that makes bad sounds
ok did a before/after without the gpio change and it was apparently bad all along
Obeida Shamoun
 
yeah that makes sense
wait do you still have acdb disabled @Felix?
