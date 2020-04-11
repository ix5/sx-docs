---
title: "IRQs and Wakeups"
description: "a.k.a. obi learns to care about docs"
date: 2020-04-11T23:19:19+02:00
draft: true
---

Trying to hunt down the source of some battery drain on tama devices that are
not akatsuki. The difference between akatsuki and the reset of the tama family
is that it uses keymaster 4, whereas apollo and akari use km3.

Some other thoughts led us to align the GIC (*Global Interrupt Controller*)
driver with the upstream version: [Update GIC to 8.2 tag][obi_gic].

More tracing:
```
adb shell "echo 1 > /sys/module/msm_show_resume_irq/parameters/debug_mask"
```

Unrelated nile/ganges debugging: Calls seem to come in from modem via glink:
```
[  546.912583] GICv3: gic_show_resume_irq: 20 triggered glink-native
[  546.912583] GICv3: gic_show_resume_irq: 295 triggered glink-native
```

### Chat conversation:

**Person B:** for all it cares, the cpus are exiting sleep state
that's all I'm seeing
**Person A:**: Let's get to the basics: What wakes up a sleeping AP?
**Person A:**: Only two things can wake up an AP
**Person C:** @PersonB To know what driver to signal?
**Person C:**By means of a wake-able interrupt, ofc
**Person A:**: Of which, one is related to the other
**Person A:**: 1. Architectural timers
**Person A:**: 2. External interrupts
Where 1 triggers an internal interrupt
So it's a irq line
There's no need to explain
So
**Person B:** ok lemme explain more
**Person B:** https://github.com/sonyxperiadev/kernel/blob/aosp/LA.UM.7.1.r1/kernel/power/suspend.c#L445-L460
so, I think we can agree this is where the suspend happens?
**Person A:**: have you tried to place prints in wakeup functions
in the GIC driver?=
have you tried to place prints in wakeup functions in the PDC driver?
and if they didnt' show anything useful
have you tried to save the interrupt status on suspend
and check it with the one on resume
**Person B:** https://github.com/sonyxperiadev/kernel/blob/aosp/LA.UM.7.1.r1/kernel/power/suspend.c#L447-L453
it's only going into here
**Person A:**: ?
**Person B:** not the abort
**Person A:**: this is the generic layer @PersonB
what wakes up the AP is something very tied to it
not a generic layer
**Person B:** that's not my point here
**Person A:**: does `wakeup_pending` return something?
is there any pending wakeup when suspend occurs?
**Person B:** my point is, I logged this and it simply doesn't enter the second case
**Person A:**: so it doesn't enter line 448?
**Person B:** it thinks it's going to sleep, but get woken up momentarily
**Person D:** the standard kernel facilities seem quite useless when debugging this
**Person B:** it does go to 448
**Person D:** we gotta dig deeper into drivers etc like @PersonA said
**Person A:**:
place logging in PDC and GIC-v3
**Person B:** https://github.com/sonyxperiadev/kernel/blob/aosp/LA.UM.7.1.r1/drivers/irqchip/qcom/pdc.c
https://github.com/sonyxperiadev/kernel/blob/aosp/LA.UM.7.1.r1/drivers/irqchip/irq-gic-v3.c
@PersonD there you go, this is where to place logging
**Person B:**
```
echo "irq != 5 && irq != 7 && irq != 20 && irq != 32 && irq != 39 && irq != 52 && irq != 61 && irq != 62 && irq != 184 && irq != 216 && irq != 230 && irq != 232 && irq != 286 && irq != 293 && irq != 295" > /d/tracing/events/irq/irq_handler_entry/filter
```

### Unrelated chat conversation

**Person A:**
```
diff --git a/arch/arm64/boot/dts/qcom/sdm660.dtsi b/arch/arm64/boot/dts/qcom/sdm660.dtsi
index 8e997c522874..d4d014877ab5 100644
--- a/arch/arm64/boot/dts/qcom/sdm660.dtsi
+++ b/arch/arm64/boot/dts/qcom/sdm660.dtsi
@@ -484,7 +484,7 @@
                compatible = "qcom,mpm-gpio-sdm660", "qcom,mpm-gpio";
                interrupt-controller;
                interrupt-parent = <&intc>;
-               #interrupt-cells = <3>;
+               #interrupt-cells = <2>;
        };        timer {
```

The size of the interrupts array in children
**Person B:**
```
		interrupts-extended = <&wakegic GIC_SPI 171
			IRQ_TYPE_EDGE_RISING>;
```
**Person A:**
ah cool. so even 2 is wrong I suppose
**Person B:**
There's 3 cells there to specify the interrupt, besides the desired state
**Person A:**
ooh i understood wrong
**Person B:**
It's some sort of meta-property, I don't know where it's used
**Person B:**
Nor if it is enforced
**Person B:**
```
	interrupt-map = <0 &intc 0 0 125 0
			1 &intc 0 0 221 0
```
**Person B:**
I may be a little wrong in pasting code from a node with `interrupt-parent=<&intc>;`
**Person B:**
Since the interrupt is referred to in that map for example

[obi_gic]: https://github.com/oshmoun/kernel/commit/28caf42bacdd57971df80f1844cfcccd267cb0c9
