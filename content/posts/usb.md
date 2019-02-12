---
title: "Quick: USB kernel things"
description: "Quick USB testing post"
date: 2019-02-08T13:00:30+01:00
draft: false
---

While pondering how to get USB3/PCI-E(via QMP PHY) to work on mainline
Linux(instead of our own Qcom/Codeaurora-based 4.9), we ran a quick test to
check which `phy`s are critical to get `peripheral` mode working(`host` is for
another day, OTG comes later).

First, check which usb controller is involved:
```
kagura:/ $ getprop sys.usb.controller
6a00000.dwc3
```

That gives us `6a00000` via [DWC3][dwc3-ti].

To find out which `phy`(≙ [Physical layer][phy-osi]) is needed for baseline USB
connectivity, we turned them off one by one: First `<&qusb_phy0>`, then
`<&ssphy>`.

[arch/arm64/boot/dts/qcom/msm8996.dtsi][dtsi]
```
&soc {
    [...]
    usb3: ssusb@6a00000{
        [...]
        dwc3@6a00000 {
            compatible = "snps,dwc3";
            // TEST: Set:
            //dr_mode = "peripheral";

            reg = <0x06a00000 0xc8d0>;
            interrupt-parent = <&intc>;
            interrupts = <0 131 0>;

            usb-phy = <&qusb_phy0>, <&ssphy>;
            //TEST: Set only one of the two:
            //usb-phy = <&qusb_phy0>;
            //usb-phy = <&ssphy>;

            tx-fifo-resize;
            snps,usb3-u1u2-disable;
            snps,nominal-elastic-buffer;
            snps,is-utmi-l1-suspend;
            snps,hird-threshold = /bits/ 8 <0x0>;
        };
```

Turns out that both are needed on Kernel 4.9 though.

In general, one can see that the Qcom DWC3 implementation is a hacked-together
beast that takes possession of the whole `PHY` layer, as opposed to the mainline
`DWC3` framework.

Compare:

- Our CAF 4.9 driver: [drivers/usb/dwc3/dwc3-msm.c][caf-dwc3]
- Mainline driver: [drivers/usb/dwc3/dwc3-qcom.c][mainline-dwc3]

[dwc3-ti]: http://processors.wiki.ti.com/index.php/Linux_Core_DWC3_User's_Guide
[phy-osi]: https://en.wikipedia.org/wiki/Physical_layer
[dtsi]: https://github.com/sonyxperiadev/kernel/blob/e17b3925fe2a3579a29cc8278138025d8ac72642/arch/arm64/boot/dts/qcom/msm8996.dtsi#L2100-L2111
[caf-dwc3]: https://github.com/sonyxperiadev/kernel/blob/e17b3925fe2a3579a29cc8278138025d8ac72642/drivers/usb/dwc3/dwc3-msm.c
[mainline-dwc3]: https://github.com/torvalds/linux/blob/9925e6ebe5c2da9601037cca73462782900b9190/drivers/usb/dwc3/dwc3-qcom.c
