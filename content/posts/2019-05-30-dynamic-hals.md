---
title: "Android Dynamic HALs"
description: ""
date: 2019-05-30T21:29:16+02:00
draft: true
---

https://source.android.com/devices/architecture/hal/dynamic-lifecycle

```
# /vendor/etc/init/android.hardware.nfc@1.2-service.rc
service vendor.nfc_hal_service /vendor/bin/hw/android.hardware.nfc@1.2-service
    interface android.hardware.nfc@1.2::INfc default
    class hal
    oneshot
    disabled
    user nfc
    group nfc
```

**Important:** If using `PRODUCT_ENFORCE_VINTF_MANIFEST` and
`VINTF_ENFORCE_NO_UNUSED_HALS`, you need to specify the exact version of your
HALs in your manifest, else you cannot use `disabled` in `init.rc` files!

```
kagura:/ # lshal
| All binderized services (registered with hwservicemanager)
VINTF R Interface                                                                                 Thread Use Server Clients
FM    Y android.frameworks.displayservice@1.0::IDisplayService/default                            0/1        582    457
DC,FM Y android.frameworks.schedulerservice@1.0::ISchedulingPolicyService/default                 0/4        1031   457
DC,FM Y android.frameworks.sensorservice@1.0::ISensorManager/default                              0/4        1031   457
DM,FC Y android.hardware.atrace@1.0::IAtraceDevice/default                                        0/1        508    457
DM,FC Y android.hardware.audio.effect@4.0::IEffectsFactory/default                                0/4        557    579 457
DM,FC Y android.hardware.audio@4.0::IDevicesFactory/default                                       0/4        557    579 457
DM,FC Y android.hardware.biometrics.fingerprint@2.1::IBiometricsFingerprint/default               0/1        712    1031 457
DM,FC Y android.hardware.bluetooth@1.0::IBluetoothHci/default                                     0/3        558    457
DM,FC Y android.hardware.camera.provider@2.4::ICameraProvider/legacy/0                            0/5        559    674 457
DM,FC Y android.hardware.cas@1.0::IMediaCasService/default                                        0/2        560    457
DM    Y android.hardware.configstore@1.0::ISurfaceFlingerConfigs/default                          0/3        561    1771 1258 1464 1031 582 457
DM,FC Y android.hardware.configstore@1.1::ISurfaceFlingerConfigs/default                          0/3        561    1771 1258 1464 1031 582 457
DM,FC Y android.hardware.drm@1.0::ICryptoFactory/clearkey                                         0/3        563    457
DM,FC Y android.hardware.drm@1.0::ICryptoFactory/default                                          0/3        562    457
DM,FC Y android.hardware.drm@1.0::IDrmFactory/clearkey                                            0/3        563    457
DM,FC Y android.hardware.drm@1.0::IDrmFactory/default                                             0/3        562    457
DM,FC Y android.hardware.drm@1.1::ICryptoFactory/clearkey                                         0/3        563    457
DM,FC Y android.hardware.drm@1.1::IDrmFactory/clearkey                                            0/3        563    457
DM,FC Y android.hardware.gatekeeper@1.0::IGatekeeper/default                                      0/1        509    708 457
DM,FC Y android.hardware.gnss@1.0::IGnss/default                                                  0/1        564    1031 457
DM,FC Y android.hardware.gnss@1.1::IGnss/default                                                  0/1        564    1031 457
DM,FC Y android.hardware.graphics.allocator@2.0::IAllocator/default                               0/3        566    1031 1771 1258 582 457
DM,FC Y android.hardware.graphics.composer@2.1::IComposer/default                                 0/4        567    582 457
DM,FC Y android.hardware.health@2.0::IHealth/default                                              1/1        568    1031 684 457
DM,FC Y android.hardware.keymaster@3.0::IKeymasterDevice/default                                  0/1        510    678 457
DM,FC Y android.hardware.light@2.0::ILight/default                                                0/1        1105   1031 457
DM,FC Y android.hardware.media.omx@1.0::IOmx/default                                              0/5        689    457
DM,FC Y android.hardware.media.omx@1.0::IOmxStore/default                                         0/5        689    682 457
DM,FC Y android.hardware.memtrack@1.0::IMemtrack/default                                          0/1        569    1031 457
DM    Y android.hardware.nfc@1.0::INfc/default                                                    0/1        1824   1710 457
DM,FC Y android.hardware.nfc@1.1::INfc/default                                                    0/1        1824   1710 457
DM,FC Y android.hardware.nfc@1.2::INfc/default                                                    0/1        1824   1710 457
DM,FC Y android.hardware.power@1.0::IPower/default                                                0/1        570    1031 457
DM,FC Y android.hardware.power@1.1::IPower/default                                                0/1        570    1031 457
DM,FC Y android.hardware.power@1.2::IPower/default                                                0/1        570    1031 457
DM,FC Y android.hardware.power@1.3::IPower/default                                                0/1        570    1031 457
X     Y android.hardware.radio.config@1.0::IRadioConfig/default                                   0/2        707    457
DM    Y android.hardware.radio.deprecated@1.0::IOemHook/slot1                                     0/2        707    457
X     Y android.hardware.radio@1.0::IRadio/default                                                0/1        571    457
DM    Y android.hardware.radio@1.0::IRadio/slot1                                                  0/2        707    457
X     Y android.hardware.radio@1.0::ISap/default                                                  0/1        572    457
DM    Y android.hardware.radio@1.0::ISap/slot1                                                    0/2        707    457
X     Y android.hardware.radio@1.1::IRadio/default                                                0/1        571    457
DM    Y android.hardware.radio@1.1::IRadio/slot1                                                  0/2        707    457
X     Y android.hardware.radio@1.1::ISap/default                                                  0/1        572    457
DM    Y android.hardware.radio@1.1::ISap/slot1                                                    0/2        707    457
X     Y android.hardware.radio@1.2::IRadio/default                                                0/1        571    457
X     Y android.hardware.radio@1.2::ISap/default                                                  0/1        572    457
FC    Y android.hardware.secure_element@1.0::ISecureElement/SIM1                                  0/2        707    457
DM,FC Y android.hardware.sensors@1.0::ISensors/default                                            1/2        573    1031 457
DM,FC Y android.hardware.soundtrigger@2.0::ISoundTriggerHw/default                                0/4        557    579 457
DM,FC Y android.hardware.soundtrigger@2.1::ISoundTriggerHw/default                                0/4        557    579 457
FC    Y android.hardware.tetheroffload.config@1.0::IOffloadConfig/default                         0/1        690    457
FC    Y android.hardware.tetheroffload.control@1.0::IOffloadControl/default                       0/1        690    457
DM,FC Y android.hardware.thermal@1.0::IThermal/default                                            0/1        574    1031 457
DM,FC Y android.hardware.usb@1.0::IUsb/default                                                    0/1        575    1031 457
DM,FC Y android.hardware.vibrator@1.0::IVibrator/default                                          0/1        576    1031 457
DM,FC Y android.hardware.wifi.supplicant@1.0::ISupplicant/default                                 1/1        1291   1031 457
DM,FC Y android.hardware.wifi.supplicant@1.1::ISupplicant/default                                 1/1        1291   1031 457
DM,FC Y android.hardware.wifi@1.0::IWifi/default                                                  0/1        577    1031 457
DM,FC Y android.hardware.wifi@1.1::IWifi/default                                                  0/1        577    1031 457
DM,FC Y android.hardware.wifi@1.2::IWifi/default                                                  0/1        577    1031 457
DC,FM Y android.hidl.allocator@1.0::IAllocator/ashmem                                             0/1        555    457
X     Y android.hidl.base@1.0::IBase/SIM1                                                         0/2        707    457
X     Y android.hidl.base@1.0::IBase/Uim0                                                         0/2        707    457
X     Y android.hidl.base@1.0::IBase/UimLpa0                                                      0/2        707    457
X     Y android.hidl.base@1.0::IBase/ashmem                                                       0/1        555    457
X     Y android.hidl.base@1.0::IBase/clearkey                                                     0/3        563    457
X     Y android.hidl.base@1.0::IBase/default                                                      0/1        1824   457
X     Y android.hidl.base@1.0::IBase/imsradio0                                                    0/2        707    457
X     Y android.hidl.base@1.0::IBase/legacy/0                                                     0/5        559    674 457
X     Y android.hidl.base@1.0::IBase/slot1                                                        0/2        707    457
X     Y android.hidl.base@1.0::IBase/uimRemoteClient0                                             0/2        707    457
X     Y android.hidl.base@1.0::IBase/uimRemoteServer0                                             0/2        707    457
DC,FM Y android.hidl.manager@1.0::IServiceManager/default                                         1/1        457    1031
FM    Y android.hidl.manager@1.1::IServiceManager/default                                         1/1        457    1031
FM    Y android.hidl.manager@1.2::IServiceManager/default                                         1/1        457    1031
DC,FM Y android.hidl.token@1.0::ITokenManager/default                                             1/1        457
FM    Y android.system.net.netd@1.0::INetd/default                                                0/1        529    706 457
FM    Y android.system.net.netd@1.1::INetd/default                                                0/1        529    706 457
FM    Y android.system.suspend@1.0::ISystemSuspend/default                                        0/1        556    573 1031 707 579 457
DC,FM Y android.system.wifi.keystore@1.0::IKeystore/default                                       0/1        678    457
DM    Y vendor.display.config@1.0::IDisplayConfig/default                                         0/4        567    457
DM    Y vendor.display.config@1.1::IDisplayConfig/default                                         0/4        567    457
DM    Y vendor.display.config@1.2::IDisplayConfig/default                                         0/4        567    457
DM    Y vendor.nxp.nxpnfc@1.0::INxpNfc/default                                                    0/1        1824   457
DM    Y vendor.qti.hardware.radio.am@1.0::IQcRilAudio/slot1                                       0/2        707    457
DM    Y vendor.qti.hardware.radio.ims@1.0::IImsRadio/imsradio0                                    0/2        707    457
DM    Y vendor.qti.hardware.radio.lpa@1.0::IUimLpa/UimLpa0                                        0/2        707    457
DM    Y vendor.qti.hardware.radio.qtiradio@1.0::IQtiRadio/slot1                                   0/2        707    457
DM    Y vendor.qti.hardware.radio.uim@1.0::IUim/Uim0                                              0/2        707    457
DM    Y vendor.qti.hardware.radio.uim@1.1::IUim/Uim0                                              0/2        707    457
DM    Y vendor.qti.hardware.radio.uim_remote_client@1.0::IUimRemoteServiceClient/uimRemoteClient0 0/2        707    457
DM    Y vendor.qti.hardware.radio.uim_remote_server@1.0::IUimRemoteServiceServer/uimRemoteServer0 0/2        707    457

| All interfaces that getService() has ever returned as a passthrough interface;
| PIDs / processes shown below might be inaccurate because the process
| might have relinquished the interface or might have died.
| The Server / Server CMD column can be ignored.
| The Clients / Clients CMD column shows all process that have ever dlopen'ed
| the library and successfully fetched the passthrough implementation.
VINTF R Interface                                                      Thread Use Server Clients
FC    ? android.hardware.audio.effect@4.0::IEffectsFactory/default     N/A        557    557
FC    ? android.hardware.audio@4.0::IDevicesFactory/default            N/A        557    557
FC    ? android.hardware.bluetooth@1.0::IBluetoothHci/default          N/A        558    558
FC    ? android.hardware.camera.provider@2.4::ICameraProvider/legacy/0 N/A        559    559
FC    ? android.hardware.drm@1.0::ICryptoFactory/default               N/A        562    562
FC    ? android.hardware.drm@1.0::IDrmFactory/default                  N/A        562    562
FC    ? android.hardware.gatekeeper@1.0::IGatekeeper/default           N/A        509    509
FC    ? android.hardware.gnss@1.1::IGnss/default                       N/A        564    564
FC    ? android.hardware.graphics.allocator@2.0::IAllocator/default    N/A        566    566
FC    ? android.hardware.graphics.composer@2.1::IComposer/default      N/A        567    567
DM,FC ? android.hardware.graphics.mapper@2.0::IMapper/default          N/A        N/A    567 582 1031 1258 1464 1771
FC    ? android.hardware.keymaster@3.0::IKeymasterDevice/default       N/A        510    510
FC    ? android.hardware.memtrack@1.0::IMemtrack/default               N/A        569    569
FC    ? android.hardware.sensors@1.0::ISensors/default                 N/A        573    573
FC    ? android.hardware.soundtrigger@2.1::ISoundTriggerHw/default     N/A        557    557
FC    ? android.hardware.thermal@1.0::IThermal/default                 N/A        574    574
FC    ? android.hardware.vibrator@1.0::IVibrator/default               N/A        576    576
DC,FM ? android.hidl.memory@1.0::IMapper/ashmem                        N/A        N/A    682 689 1031 1258 1710

| All available passthrough implementations (all -impl.so files).
| These may return subclasses through their respective HIDL_FETCH_I* functions.
VINTF R Interface                                                         Thread Use Server Clients
FC    ? android.hardware.audio.effect@4.0::I*/* (/vendor/lib/hw/)         N/A        N/A
FC    ? android.hardware.audio@4.0::I*/* (/vendor/lib/hw/)                N/A        N/A
FC    ? android.hardware.bluetooth@1.0::I*/* (/vendor/lib/hw/)            N/A        N/A
FC    ? android.hardware.bluetooth@1.0::I*/* (/vendor/lib64/hw/)          N/A        N/A
FC    ? android.hardware.camera.provider@2.4::I*/* (/vendor/lib/hw/)      N/A        N/A
FC    ? android.hardware.drm@1.0::I*/* (/vendor/lib/hw/)                  N/A        N/A
FC    ? android.hardware.gatekeeper@1.0::I*/* (/odm/lib/hw/) (-qti)       N/A        N/A
FC    ? android.hardware.gatekeeper@1.0::I*/* (/odm/lib64/hw/) (-qti)     N/A        N/A    509
FC    ? android.hardware.gnss@1.1::I*/* (/vendor/lib/hw/) (-qti)          N/A        N/A
FC    ? android.hardware.gnss@1.1::I*/* (/vendor/lib64/hw/) (-qti)        N/A        N/A
FC    ? android.hardware.graphics.allocator@2.0::I*/* (/vendor/lib64/hw/) N/A        N/A
FC    ? android.hardware.graphics.composer@2.1::I*/* (/vendor/lib64/hw/)  N/A        N/A
DM,FC ? android.hardware.graphics.mapper@2.0::I*/* (/vendor/lib/hw/)      N/A        N/A
DM,FC ? android.hardware.graphics.mapper@2.0::I*/* (/vendor/lib64/hw/)    N/A        N/A
FC    ? android.hardware.keymaster@3.0::I*/* (/odm/lib/hw/) (-qti)        N/A        N/A
FC    ? android.hardware.keymaster@3.0::I*/* (/odm/lib64/hw/) (-qti)      N/A        N/A    510
FC    ? android.hardware.memtrack@1.0::I*/* (/vendor/lib/hw/)             N/A        N/A
FC    ? android.hardware.memtrack@1.0::I*/* (/vendor/lib64/hw/)           N/A        N/A
DM,FC ? android.hardware.renderscript@1.0::I*/* (/vendor/lib/hw/)         N/A        N/A
DM,FC ? android.hardware.renderscript@1.0::I*/* (/vendor/lib64/hw/)       N/A        N/A
FC    ? android.hardware.sensors@1.0::I*/* (/vendor/lib64/hw/)            N/A        N/A
FC    ? android.hardware.soundtrigger@2.1::I*/* (/vendor/lib/hw/)         N/A        N/A
FC    ? android.hardware.thermal@1.0::I*/* (/vendor/lib/hw/)              N/A        N/A
FC    ? android.hardware.thermal@1.0::I*/* (/vendor/lib64/hw/)            N/A        N/A
FC    ? android.hardware.vibrator@1.0::I*/* (/vendor/lib/hw/)             N/A        N/A
FC    ? android.hardware.vibrator@1.0::I*/* (/vendor/lib64/hw/)           N/A        N/A
DC,FM ? android.hidl.memory@1.0::I*/* (/system/lib/hw/)                   N/A        N/A
DC,FM ? android.hidl.memory@1.0::I*/* (/system/lib/vndk-sp-R/hw/)         N/A        N/A    682 689
DC,FM ? android.hidl.memory@1.0::I*/* (/system/lib64/hw/)                 N/A        N/A
DC,FM ? android.hidl.memory@1.0::I*/* (/system/lib64/vndk-sp-R/hw/)       N/A        N/A    1031 1258 1710
```

See [system/hwservicemanager/ServiceManager.cpp][servicemanager] (simplified):
```
tryStartService(string fqName, string name) {
    SetProperty("ctl.interface_start", fqName + "/" + name);
```

```
setprop ctl.interface_start <fqname>/instance
```

To get the `fqname`, you can look at the generated output in
`/vendor/etc/vintf/manifest.xml`, or splice it together yourself:

- `android.hardware`
- HAL name(lowercase, e.g. `nfc`)
- `@1.0` (version)
- `::`
- interface (e.g. `INfc`)
- `/`
- instance (e.g. `default`)

Example:
```
setprop ctl.interface_start 'android.hardware.nfc@1.2::INfc/default
setprop ctl.interface_restart 'android.hardware.nfc@1.2::INfc/default
setprop ctl.interface_stop 'android.hardware.nfc@1.2::INfc/default
```
