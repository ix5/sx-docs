+++
title = "Unlocking"
description = "Some notes on gaining control"
date = 2018-11-16T08:42:43+01:00
draft = true
+++

[Qualcomm's chain of trust](https://lineageos.org/engineering/Qualcomm-Firmware/)

MSM8996 tone kagura

- PBL -> XBL -> Aboot/ABL
- S1 Flash protocol?

- aboot, xbl, sbl signatures by sony or by carrier?
- unlocking status saved in pbl memory?
- ta partition, unlocking info stored there as well?
- OTP_LOCK_STATUS
- get phoneprops, where stored? how accessed?

- pwning edl/pbl https://alephsecurity.com/2018/01/22/qualcomm-edl-1
- firehose protocol
- are elf files signed by sony of qcom?
- leaked sony elfs?

- writing to aboot via firehose?
- leaked qcom tools?
- setool2?

- overwriting aboot with dirtycow exploit?
- is abootbak used if aboot is unbootable/not signed properly?
- does s1flasher expand the emmc_appsboot sin/elf to fill /aboot?
- how to see if a partition is filled up or not
