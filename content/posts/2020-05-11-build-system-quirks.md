---
title: "Quick: Misc Build system quirks"
description: ""
date: 2020-05-11T17:47:08+02:00
draft: false
---

## Module partition tags

Short overview over the `Android.bp` buildvars for modules, since they make no
sense and I can never remember them.

See [soong/android/module.go][module].

**proprietary:** Whether this is a proprietary vendor module, and should be
installed into /vendor  
Use `soc_specific` instead for better meaning.

**vendor:** Whether this module is specific to an SoC (System-On-a-Chip).  
When set to true, it is installed into `/vendor` (or `/system/vendor` if
vendor partition does not exist).  
Use `soc_specific` instead for better meaning.

**soc_specific:** Whether this module is specific to an SoC (System-On-a-Chip).  
When set to true, it is installed into `/vendor` (or `/system/vendor` if vendor
partition does not exist).

**device_specific:** Whether this module is specific to a device, not only for
SoC, but also for off-chip peripherals.  
When set to true, it is installed into `/odm` (or `/vendor/odm` if odm
partition does not exist, or `/system/vendor/odm` if both odm and vendor
partitions do not exist).  
This implies `soc_specific:true`.

**product_specific:** Whether this module is specific to a software
configuration of a product (e.g. country, network operator, etc).  
When set to true, it is installed into `/product` (or `/system/product` if
product partition does not exist).

## Dependencies on other modules

Normally, linking to a module automatically adds it to the build. But if you
have a non-library module, you can still require its presence from another
module:
```
prebuilt_etc {
    name: "android.software.live_wallpaper.xml",
    src: "android.software.live_wallpaper.xml",
}

android_app {
    name: "LiveWallpapersPicker",
    srcs: ["src/**/*.java"],
    required: ["android.software.live_wallpaper.xml"],
    // [...]
}
```
For the full example, see [wallpapers: Android.bp][wallpapers].

[module]: https://android.googlesource.com/platform/build/soong/+/refs/tags/android-10.0.0_r35/android/module.go#229
[wallpapers]: https://android.googlesource.com/platform/packages/wallpapers/LivePicker/+/refs/tags/android-10.0.0_r36/Android.bp#26
