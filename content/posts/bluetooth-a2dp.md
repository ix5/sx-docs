---
title: "Bluetooth A2DP offloading notes"
date: 2019-03-12T08:18:00+01:00
draft: true
author: Felix
---

## RPA

RPA = Resolvable private address? Something with offloading?

> First question: If a peripheral is paired (and bonded) to multiple centrals
> using resolvable private addressing (aka unique IRK’s per central), does the
> peripheral have to cycle through all RPAs when trying to connect to any of
> them?

> And if so, does TI RTOS provide support for cycling through all paired IRKs
> when advertising or is it the responsibility of the application to cycle
> through the IRKs?

> My bet is that it’s the application’s responsibility. Then the next question
> is can it be done by simply changing the advertising data on some interval
> basis during runtime (using GAPROLE_ADVERT_DATA for example)?

https://e2e.ti.com/support/wireless-connectivity/bluetooth/f/538/t/682273?CC2640R2F-Multiple-Centrals-Using-RPA

## A2DP

a2dp offloading means offloading audio(AAC?) encoding to sth.. dedicated? Some
type of (A)DSP?

qcom audio + bluetooth HAL
`libbthost_?`

```
#AUDIO_FEATURE_ENABLED_A2DP_OFFLOAD := true
```

plus adding props and device manifest entries, updating FCM/DCM
