---
title: "GPS HAL Adaptation"
description: "Overhauling, soong, QMI"
date: 2020-03-23T12:12:40+01:00
draft: false
---

<!--

#############################################
# TODO
#############################################

Use header exports instead of dedicated header libs, also keep existing
#include paths instead of requiring modifications
-> Also update now obsolete commit ids to appropriate ones

#############################################


(Mostly done I think)

Methodology: What is needed? Has anyone else done this work (for us) already?
What can we ditch? What is proprietary? What is useless fluff?

Using 8.1 CAF tags, replacing proprietary QMI components, using `libloc_loader`,
changing build guards, conversion to blueprint modules, fixing header copying,
deducing (not reverse-engineering!!!) QMI headers from kernel (Jens)

Reorganizing: No longer using `LOCAL_COPY_HEADERS`, defining separate
`cc_library_headers`

(introduce androidmk again, link article, and state its limits)

soong/blueprint nice things
- filegroup
- defaults

meh
- namespaces: imports, having to add them in device tree mk,
  upcoming: visibility

-->

## Intro
The Qualcomm GPS/GNSS functionality in Android is spread out over lots of files
and repositories. Most important are the `hardware/qcom/gps` and
`vendor/qcom/opensource/location` directories, plus a lot of proprietary
libraries.

We wanted to upgrade our [forked opensource location HAL][xperia-oss-loc] to a
more recent CAF tag since we had to resort to retrofitting patches to keep up
with the `hardware/qcom/gps` parts, see for instance [this PR][oss-loc-pr].

This turned out into a larger ordeal, since more recent Qualcomm/CAF versions of
the OSS location repo depended on extensive changes to the `gps` HAL as well, so
we had to pull in those changes and adapt the repo into our SODP build
environment as well.

## Picking a CAF tag
Let‘s take as given that we need to bump our OSS location and gps HALs.
We want the `sm8150` future-oriented version of our HALs, so we head to the
CodeAurora Forum git repos and fetch the latest 8.1 tags:
[platform/hardware/qcom/gps][qcom-gps-caf-8.1] and
[platform/vendor/qcom-opensource/location][oss-loc-caf-8.1] using tag
`LA.UM.8.1.r1-14500-sm8150.0`. You can search for tags at the repo’s
[/refs][oss-loc-refs] node.

## Requirements

Starting out, the package list in [common-packages.mk][pkgsmk] looked like this:
```
# hardware/qcom/gps
PRODUCT_PACKAGES += \
    libloc_core \
    libgps.utils \
    liblocation_api \
    libloc_pla \
    libgnss
# vendor/qcom/opensource/location
PRODUCT_PACKAGES += \
    libloc_api_v02 \
    libloc_ds_api \
    libgnsspps
```

The `init`-startable location services are included via
[common-treble.mk][treblemk]:
```
# GNSS
PRODUCT_PACKAGES += \
    android.hardware.gnss@1.1-impl-qti \
    android.hardware.gnss@1.1-service-qti
```

This results in the following dependency chain:
`android.hardware.gnss@1.1-service-qti` loads
`android.hardware.gnss@1.1-impl-qti`, which loads:
- `libloc_core`
- `libgps.utils`
- `liblocation_api`
- `libloc_pla`
- `libgnss`

`libloc_core` loads the following in [ContextBase.cpp][qcom-gps-contextbase]
(shortened):
```
const char* libname = "libloc_api_v02.so";
// If gps.conf has GNSS_DEPLOYMENT = 1 (QCSR SS5 enabled):
libname = "libsynergy_loc_api.so"
if ((handle = dlopen(libname, RTLD_NOW)) != NULL) {
  getLocApi_t* getter = (getLocApi_t*) dlsym(handle, "getLocApi");
}
```
That means that we need to provide `libloc_api_v02.so`, and in case we want to
provide the OSS "synergy API", `libsynergy_loc_api.so`.

I could not find any reference to `libgnsspps`, and `libloc_ds_api` seems unused
on 8.1 as well.

One peculiarity for SODP is that we cannot rely on proprietary symbols and libs
at build time since all our non-`odm` code is public. For that reason,
[Obeida Shamoun][oshmoun] wrote a tool called [libloc_loader][deblobify] which
`dlopen`s the required libraries from the `/odm` partition. The required
libraries are defined in [libloc_loader.h][loader-libs]:
- `libdsi_netctrl.so`
- `libqmi_cci.so`
- `libqmi_common_so.so`
- `libqmiservices.so`

We can ditch `libdsi_netctrl` and `libqmiservices` and their respective
[loader functions][loader-dsi] since their symbols are no longer required on the
8.1 OSS location repo.

So, let us review our requirements:
- `android.hardware.gnss@1.1-impl-qti` (should be bumped to 2.0)
- `android.hardware.gnss@1.1-service-qti` (should be bumped to 2.0)
- `libloc_core`
- `libgps.utils`
- `liblocation_api`
- `libloc_pla`
- `libgnss`
- `libloc_api_v02.so`
- `libsynergy_loc_api.so`
- `libqmi_cci.so` (`odm`)
- `libqmi_common_so.so` (`odm`)

## Adaptation phase
To adapt the new 8.1 HALs, we need to re-apply some commits that we had applied
to the previous CAF base and cull usage of some proprietary libs.
Just rebase the [previous SODP adaptation commits][sodp-commits] on top of the
8.1 OSS location repo.

These bringup tweaks are fairly tame and self-explanatory:
- gps: [Remove liblbs_core][pavel-lbs]
- gps: [Remove conflicting /etc configs][gps-etc]
- oss location: [synergy_loc_api: Remove proprietary symbols][synergy]
- oss location: [loc_api: Move inclusion of gps_extended.h][gps_extended]
- oss location: [loc_api: Include missing loc_cfg header][loc_cfg]

## QMI and proprietary headers
Since Qualcomm wants to keep their chip and core firmware and software internals
secret, they devise all sorts of idiosyncratic schemes and data formats to
interface with their chips. One of those formats is a sort of messaging
protocol called `QMI`, a form of IPC. `QMI` stands for "Qualcomm MSM Interface".
For more information, see the entry on
[QMI on the osmocom Wiki][qmi-osmo][^osmo].

Even though the location repo is filed under `opensource`, it relies on symbol
definitions in header files that are only shipped with a Qualcomm
"Board Support Package" - `BSP` for short.

Let's gloss over _how_ exactly [Jens Andersen][enjens] managed to extract the
needed headers by _"studying the code and the functions defined in the kernel
headers"_ and keep in mind that less is more when interacting with and working
around proprietary interfaces. The needed symbols are part of the PR
[Add headers needed for compiling location provider][jens-qmi]. I am sorry that
I cannot be more specific with this topic, it‘s a delicate matter.

With the 8.1 location repo, we need to adapt a few headers slightly:
[loc_api: Update QMI header symbols][qmi_headers] and
[loc_api: Fix error type enums][enums]. [Marijn][marijns] and me were puzzled as
to how the doubly-included `common_qmi_idl_type_table_object_v01` was even
compiling with the legacy location repo, but we agreed that the definition
should be moved to `libloc_loader.c`.

## Modern makefile practices
The heading of this section is a tad contradictory, since the `Android.mk`-based
build system is very slowly being phased out. Still, one has to adapt to the
times, and Google is planning to or already has deprecated a good lot of
functions in Q and will make them errors on the R release.

We shall we begin by looking at a typical old-style multi-library entanglement
of medium complexity. Usually libraries will have interdependencies and reliance
on shared header files.

File structure:
```
├── module_a
│   │── libfoo.c
│   │── libfoo.h
│   └── Android.mk
└── module_b
    │── libbar.c
    │── libbar.h
    └── Android.mk
```

`module_a/Android.mk`:
```make
include $(CLEAR_VARS)
LOCAL_MODULE           := libmodule_a
LOCAL_SHARED_LIBRARIES := libutils
LOCAL_SRC_FILES        := libfoo.c
LOCAL_COPY_HEADERS     := libfoo.h
LOCAL_COPY_HEADERS_TO  := module_a/
LOCAL_C_INCLUDES       := $(TARGET_OUT_HEADERS)/data/inc
include $(BUILD_SHARED_LIBRARY)
```
`libmodule_a` will have access to the headers in
`$(TARGET_OUT_HEADERS)/data/inc`. The headers var resolves to:
```
$ get_build_var TARGET_OUT_HEADERS
# out/target/product/<device>/obj/include
```
So `out/target/product/<device>/obj/include/data/inc/my_inc.h` will be
`#include`-able as `my_inc.h` for `module_a/libfoo.c`:
```c
#include "my_inc.h"
[...]
```

Similarly, `libfoo.h` will be accessible to `libbar.c` (or any other lib for
that matter) because it was explicitly copied into a subfolder of the `out`
dir’s `include` directory via `LOCAL_COPY_HEADERS_TO`.

`module_b/Android.mk`:
```make
include $(CLEAR_VARS)
LOCAL_MODULE           := libmodule_b
LOCAL_SHARED_LIBRARIES := libutils libmodule_a
LOCAL_SRC_FILES        := libbar.c
include $(BUILD_SHARED_LIBRARY)
```

`module_b/libbar.c`:
```c
#include <module_a/libfoo.h>
[...]
```

You will note that the `make` system allows specifying absolute paths (i.e. not
relative to the module dir and also outside the module dir).

Android Q is starting to discourage such Wild West practices and kindly asks you
to use "header libraries" as modules instead.

`module_a/Android.mk`
```make
LOCAL_PATH := $(call my-dir)
[...]
include $(CLEAR_VARS)
LOCAL_MODULE := libmodule_a_headers
LOCAL_EXPORT_C_INCLUDE_DIRS := $(LOCAL_PATH)/
include $(BUILD_HEADER_LIBRARY)
```

`module_b/Android.mk`:
```make
[...]
LOCAL_HEADER_LIBRARIES := libmodule_a_headers
```

At the same time, you should revise your makefiles and see whether you can
remove some overzealous logic. Makefiles are slow to parse and can slow down
your iteration speed a lot.

We do the same for `libloc_loader` and move its header into a separate module:
[loc_api: Clean libloc_loader, headers as module][libloc_header].

## Soong and blueprints
Now we also need to slightly modify some behaviour because `soong` does not
allow that specific mode of operating any more. We can prepare this in `make`
already so that the switch to soong will not be as drastic.
[utils+gnss: Invert debuggable logic][gnss-invert]: Rather than checking whether
the build type is a `user` build, soong with its `product_variables: debuggable`
only allows checking whether it is not.

Now we can go ahead and start converting the gps and location modules to the
soong `Android.bp` language. Using [blueprints][bp] is much faster to parse. A
downside is that any dependency of a module defined in a blueprint file must
also be defined in a blueprint file, because all blueprint modules are evaluated
and validated before `make` modules are read.

You can use the [androidmk tool][androidmk-post] to bear the brunt of the
conversion work, but complex `make` structures will not translate one-to-one
into blueprint format.

`../`-style relative `LOCAL_C_INCLUDES` outside the module dir like in
[gps/2.0/Android.mk][gps-rel-include] are no longer allowed. We need to define
those as header library modules. For make, you can refer to the
`BUILD_HEADER_LIBRARY` example from above, in soong it looks like this:
[android: Define viz+measurement@1.0 header libs][measurement_header]
[^export_headers].

[Source files from outside the module directory][gps-relative] are similarly no
longer allowed.  Soong has the concept of a `filegroup` which we can use
instead.

File structure:
```
├── foo
│   ├── foo.c (shared)
│   └── Android.bp
├── bar
│   ├── bar.c
│   └── Android.bp
└── baz
    ├── baz.c
    └── Android.bp
```

`foo/Android.bp`
```
filegroup {
    name: "foo_src",
    srcs: ["foo.c"]
}
```

You can reference that group of files via `:foo_src` (note the colon) from
anywhere else, e.g. `bar/Android.bp`
```
cc_library_shared {
    name: "bar",
    srcs: [
        "bar.c",
        ":foo_src",
    ],
}
```

You can no longer rely on traditional conditional statements in soong; this is a
no-go:
```
ifeq ($(GNSS_HIDL_LEGACY_MEASURMENTS),true)
LOCAL_CFLAGS += -DGNSS_HIDL_LEGACY_MEASURMENTS
endif
```
You will need to write your own go logic in Go for makevar-dependent build
logic. See for example how SODP differentiates between `gralloc` versions in the
display HAL: [bootstrap_go_module][intf-bootstrap],
[gralloc_defaults.go][gralloc-go], [device tree config][soong-config-dt] via
`SOONG_CONFIG_<X>`.

For project-wide inherited variables, Qualcomm uses
[target_specific_features.mk][features] and sets `LOCAL_CFLAGS` to
`$(GNSS_CFLAGS)`. We can use the `defaults` system of soong to supplant that
behaviour:
```
cc_defaults {
    name: "foo_defaults",
    // Contents of $(GNSS_CFLAGS) from target_specific_features follow:
    cflags: [
        "-Werror",
        "-Wno-error=unused-parameter",
        [...]
    ],
}
[...]
cc_library_shared {
    name: "libfoo",
    defaults: [
        "foo_defaults",
    ],
}
```
Defaults are usable even from outside the current module. But note that source
files stay relative to the inheriting module, so if you have `foo/foo.c`,
define `foo_defaults` with `srcs: ["foo.c"]` and try to access `foo_defaults`
from `bar` - which also contains `foo.c`, you will have `bar/foo.c` selected as
`srcs`! Defaults are like header files in that way.
<!-- TOOD: Verify truthiness of this... -->

Taken together, the conversion of our GPS and location repos to soong looks like
this: [location: Convert to Android.bp][location-bp] and
[gps: Convert to Android.bp][gps-bp]. Note that we need to empty the
`Android.mk` file or give it a build guard so that we do not run into duplicate
module definitions.

## Namespaces
Now a particular annoyance: `android.hardware.gnss@2.0-service-qti` relies on
[libqti_vndfwk_detect.so][fwkdetect], which is
[hidden behind a soong namespace][fwk-namesp]. Even though the namespace is
added to `PRODUCT_SOONG_NAMESPACES` in our device trees, `libqti_vndfwk_detect`
still is inaccessible from other soong modules which have not explicitly
imported the `core-utils` namespace.

Note: `make` modules can access `libqti_vndfwk_detect` because they only "care"
about whether `core-utils` is in the board’s `PRODUCT_SOONG_NAMESPACES`.
<!-- TOOD: Verify truthiness of this... -->

We define a namespace for `hardware/qcom/gps` and let it `import` the
`core-utils` namespace: [gps: Android.bp: Define soong_namespace][gps-namesp].
That in turn necessitaces namespacing the OSS location repo:
[location: Android.bp: Define soong_namespace][location-namesp].

All in all, more work than I’d have liked, but that is the "correct" way to do
it, according to Google.

## Device trees
Those changes in the hardware repos need to be reflected in the device trees as
well. [Add GPS+location soong namespaces][dt-common-namesp] and
[Update GNSS package list][dt-common-pkg].

## Sepolicy
The uprevisioned service binary needs an SELinux label:
`sepolicy/vendor/file_contexts`:
```
-/(system/vendor|vendor)/bin/hw/android\.hardware\.gnss@1\.1-service-qti   u:object_r:hal_gnss_qti_exec:s0
+/(system/vendor|vendor)/bin/hw/android\.hardware\.gnss@2\.0-service-qti   u:object_r:hal_gnss_qti_exec:s0
```

## Finishing
So, what‘s left? Cleaning up and crafting proper commits, documentation,
testing, trying to send as much as possible upstream.

Converting your projects to soong can make them "cleaner" and faster to parse,
but since Google is obsessed with namespacing and versioning things - well,
overengineering and refactoring constantly is what it is - implementing
blueprints often ends up being a bit more complicated than it ought to.

[^osmo]: The osmocom Wiki page on QMI might be outdated at this point. QMI is a proprietary and not publicly documented interface.
[^export_headers]: For later, it might be interesting to look into `export_header_lib_headers` and `export_shared_lib_headers`/`export_static_lib_headers`.

[pkgsmk]: https://github.com/sonyxperiadev/device-sony-common/blob/456037de1ff6823d4d1c2e243d439ec3e9a8cfe8/common-packages.mk#L54-L66
[treblemk]: https://github.com/sonyxperiadev/device-sony-common/blob/456037de1ff6823d4d1c2e243d439ec3e9a8cfe8/common-treble.mk#L70-L73
[xperia-oss-loc]: https://github.com/sonyxperiadev/vendor-qcom-opensource-location/tree/q-mr0
[oss-loc-pr]: https://github.com/sonyxperiadev/vendor-qcom-opensource-location/pull/22
[qcom-gps-caf-8.1]: https://source.codeaurora.org/quic/la/platform/hardware/qcom/gps/tree/?h=LA.UM.8.1.r1-14500-sm8150.0
[oss-loc-caf-8.1]: https://source.codeaurora.org/quic/la/platform/vendor/qcom-opensource/location/tree/?h=LA.UM.8.1.r1-14500-sm8150.0
[oss-loc-refs]: https://source.codeaurora.org/quic/la/platform/vendor/qcom-opensource/location/refs/
[qcom-gps-contextbase]: https://git.ix5.org/felix/hardware-qcom-gps/src/commit/17d7115c55f42dbe410ed5e4eada4e5faedf01ac/core/ContextBase.cpp#L43-L44
[oshmoun]: https://github.com/oshmoun
[deblobify]: https://github.com/sonyxperiadev/vendor-qcom-opensource-location/pull/8
[loader-libs]: https://github.com/sonyxperiadev/vendor-qcom-opensource-location/blob/d0e4656d09fddb923df440c9b6025c58b9e52135/loc_api/libloc_loader/libloc_loader.h#L4-L7
[loader-dsi]: https://github.com/sonyxperiadev/vendor-qcom-opensource-location/blob/d0e4656d09fddb923df440c9b6025c58b9e52135/loc_api/libloc_loader/libloc_loader.c#L14
[pavel-lbs]: https://git.ix5.org/felix/hardware-qcom-gps/commit/4756243ee494aa647201523c52bf6b3fcd3ae7ba
[gps-etc]: https://git.ix5.org/felix/hardware-qcom-gps/commit/e9cd68d4b5455b20b80e6eb541afdaecc80297b6
[sodp-commits]: https://github.com/sonyxperiadev/vendor-qcom-opensource-location/commits/cb0f300a11348266eba03f4d6cd3a29a82f5a586
[synergy]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/8fa540e9da4e953e0bb9743e58bc84ae8c09eab7
[gps_extended]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/7fd1200797adfc3b34082c20ca0e64f13908e894
[enums]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/d6142220859e23a7933be2cd3fb6cca0bac90c10
[loc_cfg]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/04b2d11df23510c682fc0a72466ca7bf9eb89e28
[qmi_headers]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/a795a20ffafe413fff5596a3e8d3274128133385
[libloc_header]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/22b2815d54f036f90d629320b109aa0f1f976bdc
[qmi-osmo]: https://osmocom.org/projects/quectel-modems/wiki/QMI
[jens-qmi]: https://github.com/sonyxperiadev/vendor-qcom-opensource-location/pull/2
[enjens]: https://github.com/EnJens
[marijns]: https://github.com/MarijnS95
[gnss-invert]: https://git.ix5.org/felix/hardware-qcom-gps/commit/a5c3dfb98572aaf6a1ce3b2b529f4bc0b8261c09
[bp]: https://android.googlesource.com/platform/build/soong/+/refs/tags/android-10.0.0_r32#android_bp-file-format
[androidmk-post]: {{< ref "blueprints.md" >}}
[gps-relative]: https://git.ix5.org/felix/hardware-qcom-gps/src/branch/caf-8.1-sodp-port/android/2.0/Android.mk#L20-L21
[gps-rel-include]: https://git.ix5.org/felix/hardware-qcom-gps/src/branch/caf-8.1-sodp-port/android/2.0/Android.mk#L37-L38
[measurement_header]: https://git.ix5.org/felix/hardware-qcom-gps/commit/3ddeba35d73fe0b5f73b990e1c6ac1ea428fff7a
[intf-bootstrap]: https://github.com/sonyxperiadev/vendor-qcom-opensource-commonsys-intf-display/blob/fa5a97670dc1a0939aa287cf4b1e5f25c8f82815/Android.bp#L1-L19
[gralloc-go]: https://github.com/sonyxperiadev/vendor-qcom-opensource-commonsys-intf-display/blob/fa5a97670dc1a0939aa287cf4b1e5f25c8f82815/gralloc_defaults.go
[soong-config-dt]: https://github.com/sonyxperiadev/device-sony-common/blob/456037de1ff6823d4d1c2e243d439ec3e9a8cfe8/CommonConfig.mk#L77-L81
[features]: https://git.ix5.org/felix/hardware-qcom-gps/src/commit/da31f93c4fe8e1fb8e8d0fc0b75c57e1e18b816f/build/target_specific_features.mk
[location-bp]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/fdad662beaa29974854fe2515d6fd4bd0f0d3049
[gps-bp]: https://git.ix5.org/felix/hardware-qcom-gps/commit/281b87dee91cc10de5ee81898dd8dcfa3a1b27e6
[fwkdetect]: https://github.com/sonyxperiadev/vendor-qcom-opensource-core-utils/blob/52439ba6f6e540a4d112f44e11e0abd236510c35/fwk-detect/Android.bp#L2
[fwk-namesp]: https://github.com/sonyxperiadev/vendor-qcom-opensource-core-utils/pull/2
[gps-namesp]: https://git.ix5.org/felix/hardware-qcom-gps/commit/da31f93c4fe8e1fb8e8d0fc0b75c57e1e18b816f
[location-namesp]: https://git.ix5.org/felix/vendor-qcom-opensource-location/commit/6b64acf58bd7385af9813183cc64218ce6282b57
[dt-common-namesp]: https://git.ix5.org/felix/device-sony-common/commit/3a745222647aca381464a50c2465b17290595458
[dt-common-pkg]: https://git.ix5.org/felix/device-sony-common/commit/d48a4b1eb926a704f226fc3a4288ac2453e88614
