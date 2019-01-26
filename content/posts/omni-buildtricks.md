---
title: "Omni build tips"
date: 2019-01-01T09:18:00+01:00
bref: "Some tips for OmniROM developers"
draft: false
author: Felix
---

## Faster rebuilds

No need to re-read and re-evaluate all the `Android.mk` files, just leave the
ROM fingerprints/timestamps as they are for development.

E.g. the ROM_FINGERPRINT is dynamically set(based on `$(shell date ...)`), so it
triggers a full re-read of build variables:
```
ROM_FINGERPRINT := OmniROM/$(PLATFORM_VERSION)/$(TARGET_PRODUCT_SHORT)/$(shell date +%Y%m%d.%H:%M)
```

Change all these values so they are no longer tied to the current date:
```
# vendor/omni/config/version.mk
ROM_VERSION := current
ROM_FINGERPRINT := OmniROM/current
PRODUCT_SYSTEM_DEFAULT_PROPERTIES += \
    ro.omni.fingerprint=current
```
