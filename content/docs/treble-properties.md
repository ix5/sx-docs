---
title: "Treble Properties"
description: "VNDK, HALs, permissions and more"
date: 2019-03-02T18:42:43+01:00
draft: false
toc: true
---

## Full treble?
To make a device treble-compatible, just set `PRODUCT_FULL_TREBLE_OVERRIDE :=
true` and be done? But what does that even mean?

For treble requirements, see [build/make/core/config.mk][build-config]:
```
PRODUCT_TREBLE_LINKER_NAMESPACES
PRODUCT_SEPOLICY_SPLIT
PRODUCT_ENFORCE_VINTF_MANIFEST
PRODUCT_NOTICE_SPLIT
TARGET_COPY_OUT_VENDOR := vendor
```
(`PRODUCT_NOTICE_SPLIT` is always true and cannot be altered)

Further requirements for devices *launching* on Oreo and up:
```
BOARD_PROPERTY_OVERRIDES_SPLIT_ENABLED := true
BOARD_BUILD_SYSTEM_ROOT_IMAGE := true
BOARD_USES_RECOVERY_AS_BOOT := true
BOARD_AVB_ENABLE := true
```

## VNDK - Vendor Native Development Kit
Defining `BOARD_VNDK_VERSION` means the generated build will restrict access to
libraries to only those explicitly made available to vendor programs.
Usually you'd set it to `current`, meaning the latest VNDK version will be used.

You can test building with VNDK but disable it at runtime:
```
BOARD_VNDK_RUNTIME_DISABLE := false
```

There is much more to to it, read about
[the VNDK at source.android.com][vndk-source].

## Linker namespaces

Example full-treble config:
```
BOARD_VNDK_VERSION := current
# BOARD_VNDK_VERSION obsoletes TREBLE_LINKER_NAMESPACES
PRODUCT_TREBLE_LINKER_NAMESPACES := false
```

c.f. [system/core/rootdir/Android.mk][systemcore-androidmk]:
```
_enforce_vndk_at_runtime := false
ifdef BOARD_VNDK_VERSION
  ifneq ($(BOARD_VNDK_RUNTIME_DISABLE),true)
    _enforce_vndk_at_runtime := true
  endif
endif
[...]
ld_config_template := $(LOCAL_PATH)/etc/ld.config.txt

_enforce_vndk_lite_at_runtime := false
ifeq ($(_enforce_vndk_at_runtime),false)
  ifeq ($(PRODUCT_TREBLE_LINKER_NAMESPACES)|$(SANITIZE_TARGET),true|)
    _enforce_vndk_lite_at_runtime := true
  endif
endif
[...]
ld_config_template := $(LOCAL_PATH)/etc/ld.config.vndk_lite.txt

# for legacy non-treblized devices
LOCAL_SRC_FILES := etc/ld.config.legacy.txt
```

So, to conclude linker paths stuff:

- Setting `BOARD_VNDK_VERSION` enforces usage of the latest
  `ld.config.txt`(`ld.config.28.txt` for Pie), unless
  `BOARD_VNDK_RUNTIME_DISABLE` is set. This means `_enforce_vndk_at_runtime` is
  set internally.
- If `_enforce_vndk_at_runtime` is not set, `PRODUCT_TREBLE_LINKER_NAMESPACES`
  enforces usage of `ld.config.vndk_lite.txt`, which is less restrictive.
- Not setting any of these props means `ld.config.legacy.txt` is used, which is
  meant for pre-treble devices and very permissive.

More on [linker namespaces at source.android.com][linker-source].

## vintf - Vendor Interface Manifest
After creating the device manifest, defining `DEVICE_MANIFEST_FILE` and setting
`PRODUCT_ENFORCE_VINTF_MANIFEST_OVERRIDE := true` as explained in
[vintf: develop for a new device][vintf-develop]. This will alert you if you
a mismatch between HALs defined in the *Device Manifest(DM)* and those defined
in your *Device Compatibility Matrix.*

You can also be alerted if you have defined HALs in your manifest but not told
the framework about them in your *Compatibility Matrix* with
`VINTF_ENFORCE_NO_UNUSED_HALS := true`. This invokes `assemble_vintf` in strict
mode when checking at build time.

c.f. [AssembleVintf.cpp][assemblevintf](simplified here):
```
// If -c is provided, check it.
bool checkDualFile(const HalManifest& manifest, const CompatibilityMatrix& matrix) {
    if (getBooleanFlag("PRODUCT_ENFORCE_VINTF_MANIFEST")) {
        if (!manifest.checkCompatibility(matrix, &error)) {
            std::cerr << "Not compatible: " << error << std::endl;

    // Check HALs in device manifest that are not in framework matrix.
    if (getBooleanFlag("VINTF_ENFORCE_NO_UNUSED_HALS")) {
        auto unused = manifest.checkUnusedHals(matrix);
        if (!unused.empty()) {
            std::cerr << "Error: The following instances are in the device manifest but "
                      << "not specified in framework compatibility matrix: " << std::endl
                      << "    " << android::base::Join(unused, "\n    ") << std::endl
```

The other place `PRODUCT_ENFORCE_VINTF_MANIFEST` has an effect is with the
Android [HIDL service manager][servicemanagement] (configured via
[transport/Android.bp][libhidl-transport]).
If `ENFORCE_VINTF_MANIFEST` is defined, `bool vintfLegacy = false;`
applies.
```
getRawServiceInternal(descriptor, instance, bool retry, bool getStub) {
    [...]
    #ifdef ENFORCE_VINTF_MANIFEST
    bool vintfLegacy = false;
    #else
    bool vintfLegacy = (transport == Transport::EMPTY);
    #endif
    bool vintfHwbinder = (transport == Transport::HWBINDER);
    bool vintfPassthru = (transport == Transport::PASSTHROUGH);
    for (int tries = 0; !getStub && (vintfHwbinder || vintfLegacy); tries++) {
        if (getStub || vintfPassthru || vintfLegacy) {
            Return<sp<IBase>> ret = sm->get(descriptor, instance);
            [...]
            waiter->done(); // exit loop
        }
    }
    if (getStub || vintfPassthru || vintfLegacy) {
        sp<IServiceManager> pm = getPassthroughServiceManager();
    }
```
`ENFORCE_VINTF_MANIFEST` closes a loophole in `getRawServiceInternal` which
means `libhidltransport` will no longer fetch services which are not `hwbinder`
or `passthrough`, i.e. `EMPTY` transports will no longer be fetched.

To list all HALs and HAL services activity on-device, use the `lshal` command.

More on [vintf at source.android.com][vintf-source].

## Split SELinux policy
Pre-treble, the policy files for SElinux were situated on the ramdisk of the
`boot` partition[^1]. They consisted of an `sepolicy` binary and `file_contexts`
files in the root directory ramdisk.

With `PRODUCT_SEPOLICY_SPLIT_OVERRIDE` set, the `sepolicy` binary will be split
up into *platform*(system) and *vendor* policy files. They policy is also no
longer a binary file but rather uses the new `.cil` plain-text format. 

The *platform* `.cil` policy and context files will be pushed to
`/system/etc/selinux/`, while the *vendor* `.cil` policy and context files will
be pushed to `/vendor/etc/selinux/`.

Another change in behaviour when enabling the split is the effect of
[not_full_treble() macros][not-full-treble]: With `SEPOLICY_SPLIT` set,
[target_full_treble is set to true][sepolicy-defs] and the macros are now
simply ignored, i.e.  all the policy that you inclosed in
`not_full_treble(allow ...)` statements is ignored.

More on [implementing SELinux on source.android.com][sepolicy-source].

[^1]: Exception: When using `system-as-root`, the `sepolicy` binary was not on the boot ramdisk but rather on the root of the `system` partition.  For more, see [Gotcha: sepolicy for system-as-root devices][sepolicy-ab].

[build-config]: https://android.googlesource.com/platform/build/+/12fe6f01e355619c9584390a4561dd734f6cf003/core/config.mk#788
[vndk-source]: https://source.android.com/devices/architecture/vndk#vndk-versioning
[systemcore-androidmk]: https://android.googlesource.com/platform/system/core/+/93d837f3a90acec007647f21ed4573f044fa6f1e/rootdir/Android.mk#220
[linker-source]: https://source.android.com/devices/architecture/vndk/linker-namespace
[vintf-develop]: https://source.android.com/devices/architecture/vintf/dm#develop-new-devices
[assemblevintf]: https://android.googlesource.com/platform/system/libvintf/+/9adf115f40240d1f8bfd0266c2445f7a9b3e0262/AssembleVintf.cpp#266
[libhidl-transport]: https://android.googlesource.com/platform/system/libhidl/+/22a3454d4f78075e261aa64991f75c724a8bd99d/transport/Android.bp#76
[servicemanagement]: https://android.googlesource.com/platform/system/libhidl/+/7637124c991c36700956159e57f11c755f94f60a/transport/ServiceManagement.cpp#689
[vintf-source]: https://source.android.com/devices/architecture/vintf
[sepolicy-source]: https://source.android.com/security/selinux/build#android-o
[not-full-treble]: https://android.googlesource.com/platform/system/sepolicy/+/0fa3d2766f4d9d84dd01d2e2d75d366734cfcc5f/public/te_macros#462
[sepolicy-defs]: https://android.googlesource.com/platform/system/sepolicy/+/053cb34130b763d93e2181062ebe1b5f8bf3ad9c/definitions.mk#11
[sepolicy-ab]: ./../post/gotcha-sepolicy-for-system-as-root-devices/
