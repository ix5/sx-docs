---
title: "Sony/Qualcomm device subsystems"
slug: "sony-qualcomm-device-subsystems"
aliases: ["/post/sony/qualcomm-device-subsystems"]
description: "Small glossary"
date: 2018-12-22T18:07:04+01:00
draft: false
---

- `lpass` = Low-Power Audio SubSystem
- `mpss` = audio memory stuff for adsp
  (mpa and mpss are the addresses for venus/adsp hexagon stuff)
- `kgsl` = adreno, kernel graphics support layer
- `cdsp` is the term for adsp on newer snapdragon processors
  https://developer.qualcomm.com/docs/snpe/overview.html
  (sdm835 has adsp, 845 and 855 have cdsp)
- `SLPI` = Sensor Low Power Island
- `mpm` = Modem Power Manager
- `BIMC` = Bus Interface Memory Controller
- `QRTR` = Qualcomm's IPC router for QMI comms
- `vidc` = v4l en-/decoders, now called `venus`. Kernel driver.
  See [lwn][lwn-vidc], [venus][venus-kernel]
- `fpc` = Fast Power Collapse, else "Fingerprint controller"
- `tlmm` = TOP Level Mode Multiplexer, see [msm-pinctrl][tlmm]
- `MPM` = MSM sleep Power Manager
- `PIL` = Peripheral Image Loader
  > used for loading QDSP6v5 (Hexagon) firmware images for modem subsystems into
  > memory and preparing the subsystem's processor to execute code. It's also
  > responsible for shutting down the processor when it's not needed.
- `NoC` = Network-on-Chip
- Clock ending in `_a`: Always-on RPM set clock
- `IBS` = Instruction Based Sampling, could also be In-Band Switching 
- `HBM` = If in display context: High Brightness mode, else High Bandwidth
  Memory
- `SPMI` = System Power Management Interface - see [inclusion commit][spmi]
- `PMIC` = Power management integrated circuit

More information, collected unofficially: [Qualcomm Kernel][osmocom]

[lwn-vidc]: https://lwn.net/Articles/705831/
[venus-kernel]: https://github.com/torvalds/linux/blob/d8924c0d76aaa52e4811b5c64115d9a7f36cc73a/Documentation/devicetree/bindings/media/qcom%2Cvenus.txt
[tlmm]: https://android.googlesource.com/kernel/msm/+/android-wear-5.0.2_r0.1/Documentation/devicetree/bindings/pinctrl/msm-pinctrl.txt
[osmocom]: https://osmocom.org/projects/quectel-modems/wiki/Qualcomm_Kernel
[spmi]: https://github.com/torvalds/linux/commit/5a86bf343976b9c8ab2f240bc866451fa67e5573
