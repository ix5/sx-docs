---
title: "Faster Development with RROs"
date: 2018-11-11T23:41:34+01:00
description: "Runtime Resource Overlays make customizing Android much easier and
more portable"
author: Felix
draft: false
---

<!--
# TODO:

- Is signing system apks necessary on our builds?
- Is signature checking enabled on omni etc? It surely is on lineage
- How to disable temporarily?
- Easier way to setup build tools
- Setting prioriy via OMS? How does it get calculated?
-->

## What are RROs?

- A way of overwriting a lot of Android internal stuff without having to
  recompile a whole ROM
- Take the form of an apk file in /system/vendor/overlay

## How does it work right now in Pie?

- Overlay manager service(OMS) introduced
- Priority?

## Testing it out within your AOSP build tree

Example: We want to change something in `frameworks/base/core/res`. But it's a
huge repo and compiling would take ages!  Solution: Use overlays, in this case
for `framework-res`.

- Used already by SODP! Check your `device-sony-common` and
  `device-sony-<devicename>` folders for overlay folders
- Includes all `DEVICE_PACKAGE_OVERLAYS` and `PRODUCT_PACKAGE_OVERLAYS`
  directories
- Fuses them into an apk named `<packagename>_auto_generated_rro.apk`  in
  `/system/vendor/overlay`
- For framework-res: `framework-res_auto_generated_rro.apk`
- `m -j $(nproc) framework-res_auto_generated_rro`
- `adb push out/target/product/<devicename>/system/vendor/overlay/framework-res__auto_generated_rro.apk /system/vendor/overlay/`
- Fast! ~30 seconds

## Going faster!

- But can we speed this up more? And why do we need to have a whole AOSP tree
  initialized when we just want to change a few xmls?
- Faster feedback from testers, empowering people with limited knowledge or
  resources to still participate
- Generate standalone RRO overlay
- Check example repo
- Push overlay.apk into /system/vendor/overlay
- Check installed overlays with `cmd overlay list`
- Enable own overlay with `cmd overlay enable [--user 0] <my-overlay>`
- May need to set priority via `cmd overlay set-priority lowest|highest <number> ...`

## Security

- Platform keys in build/target/product/security
- SODP keys in release-keys repo
- System apps need/get "signature" permission
