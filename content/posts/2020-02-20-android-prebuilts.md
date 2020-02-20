---
title: "Android Prebuilts"
description: ""
date: 2020-02-20T20:50:46+01:00
draft: false
---

`PREBUILT_SHARED_LIBRARY` is deprecated. Define `LOCAL_MODULE_CLASS` and call
`BUILD_PREBUILT` instead.

Current supported prebuilt types:  
[build/soong/androidmk/cmd/androidmk/android.go][androidmk-q]:
```
var prebuiltTypes = map[string]string{
    "SHARED_LIBRARIES": "cc_prebuilt_library_shared",
    "STATIC_LIBRARIES": "cc_prebuilt_library_static",
    "EXECUTABLES":      "cc_prebuilt_binary",
    "JAVA_LIBRARIES":   "java_import",
    "ETC":              "prebuilt_etc",
}
```
Supported on current master, as of 2020-02-20:  
[build/soong/androidmk/androidmk/android.go][androidmk-q]:
```
var prebuiltTypes = map[string]string{
    "SHARED_LIBRARIES": "cc_prebuilt_library_shared",
    "STATIC_LIBRARIES": "cc_prebuilt_library_static",
    "EXECUTABLES":      "cc_prebuilt_binary",
    "JAVA_LIBRARIES":   "java_import",
    "APPS":             "android_app_import",
    "ETC":              "prebuilt_etc",
}
```

## Binary
```
include $(CLEAR_VARS)
LOCAL_MODULE := myexec
LOCAL_SRC_FILES := myexec
LOCAL_MODULE_CLASS := EXECUTABLES
LOCAL_MODULE_TAGS := optional
# Optional:
LOCAL_MULTILIB := 64
LOCAL_PROPRIETARY_MODULE := true
LOCAL_SHARED_LIBRARIES := libmyexample
include $(BUILD_PREBUILT)
```
and in `blueprint` format:
```
cc_prebuilt_binary {
    name: "myexec",
    srcs: ["myexec"],
    // Optional:
    vendor: true,
    relative_install_path: "hw",
    // Soong supports the same extras as for regular cc_binary:
    shared_libs: [
        "libmyexample",
    ],
    //vintf_fragments: ["vendor.acme.myexec@1.0-service.xml"],
    //init_rc: ["vendor.acme.myexec@1.0-service.rc"],
}
```

## (Shared) library
```
include $(CLEAR_VARS)
LOCAL_MODULE := libmyexample
LOCAL_SRC_FILES := $(LOCAL_MODULE).so
# For shared:
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
# For static:
#LOCAL_MODULE_CLASS := STATIC_LIBRARIES
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := .so
# Optional:
LOCAL_MULTILIB := 64
LOCAL_PROPRIETARY_MODULE := true
include $(BUILD_PREBUILT)
```
and in `blueprint` format:
```
cc_prebuilt_library_shared {
    name: "libmyexample",
    srcs: ["libmyexample.so"],
    // Optional:
    compile_multilib: "64",
    proprietary: true,
    strip: {
        none: true,
    },
}
// For static:
//cc_prebuilt_library_static {
```

# App
```
include $(CLEAR_VARS)
LOCAL_MODULE := MyApp
LOCAL_SRC_FILES := MyApp.apk
LOCAL_CERTIFICATE := platform
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := .apk
# Optional:
LOCAL_PRIVILEGED_MODULE := true
LOCAL_PROPRIETARY_MODULE := true
include $(BUILD_PREBUILT)
```
`blueprint` support for `android_app_import` is in current master (as of
2020-02-20), will be available in Android R:
```
android_app_import {
    name: "MyApp",
    // Make sure the build system doesn't try to resign the APK
    dex_preopt: {
        enabled: false,
    },
    apk: "MyApp.apk",
    presigned: true,
}
```

# Converting
As always, you can use [the androidmk tool][androidmk-post] to convert from
`make` to `blueprint` syntax - not the other way around, though. Take a look
into the source code of `build/make` and `build/soong` to find out equivalents
if you must.

[androidmk-q]: https://android.googlesource.com/platform/build/soong/+/refs/tags/android-10.0.0_r29/androidmk/cmd/androidmk/android.go#931
[androidmk-master]: https://android.googlesource.com/platform/build/soong/+/988414c2cf6bfb868df7d402e0bf825d6fd44cc8/androidmk/androidmk/android.go#934
[androidmk-post]: {{< ref "blueprints.md" >}}
