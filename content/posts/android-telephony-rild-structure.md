---
title: "Android Telephony Rild Structure"
date: 2019-02-02T23:52:30+01:00
draft: true
author: Felix
---

Closed-source blobs, open source blobs
References in PRODUCT_PACKAGES
Things in the manifest
Properties

```
(vendor).ril.libpath=
```

IOemHook is registered in harware/ril/libril/ril_service.cpp.

The addition of the default radio.config@1.0-service in #563 doesn't make much
sense. There are references to IRadioConfig and it's ::registerAsService in
libril-qc-qmi-1.so, but do not seem to be called. Which complements the fact
that it's not registered (as seen in service list) either. The functions in the
default service are no-ops anyway.

In any case, we are not specifying this service to be available in our
manifest.xml.

Same for android.hardware.radio@1.2-sap-service : ISap is registered in
hardware/ril (depending on the available sims even), so there's no point to add
this no-op service (the functions are stubs).

We are specifying the wrong radio versions. hardware/ril/libril registers IRadio
and ISap 1.1, but we declare 1.2.

service list (and/or vndservice list) are broken. The aforementioned services
are definitely successfully registered (added log statements to print the return
status), but do not show up in the list.

Which means this PR is good as it is. I'd still like to continue on #563, but do
the following differently:

Change our manifest to reflect IRadio and ISap 1.1.

Leave out the no-op services.

Check if libril-qc-qmi-1.so is really registering IRadioConfig. If so, we should
specify it in the manifest for correctness.

Keep an fqname for IOemHook as it's still specified and used.

Remove android.hardware.radio@1.2-radio-service: this default service is a no-op
apart from two functions. Besides, we are already registering IRadio from libril
as mentioned before.
