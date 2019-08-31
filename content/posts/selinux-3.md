---
title: "SElinux on Android: Part 3"
description: "Treble and onward: Security changes"
date: 2018-12-22T18:07:04+01:00
draft: true
---

## Stuff:
- sepolicy split
  - `PRODUCT_x` vars
  - move into `/system/etc/selinux/`
  - move into `/vendor/etc/selinux/`
  - now 2 policies(instead of one binary `sepolicy` file in root dir,
    definitions from one are no longer accessible from the other!
  - build system change: vendor_service_contexts to vndservice_contexts
- `service_contexts`, `hwservice_contexts`, `vndservice_contexts`
- how public and vendor/private dirs are fused now
- move from binary to `.cil` format, coping with .cil?
- macros for treble support(`not_full_treble()`) and why to avoid them
- system-as-root locations
- more restrictive neverallows
- example: sonyxperiadev sepolicy repo from one folder to split vendor,
  public, private when going from oreo to pie
- `hal_camera_default` etc labels and re-using aosp ones
- init -> init + vendor_init split

```
# Platform witout a vendor partition
TARGET_COPY_OUT_VENDOR := system/vendor
# Force split
# (not working yet without real split vendor part)
#BOARD_PROPERTY_OVERRIDES_SPLIT_ENABLED := true

# Test split sepolicy on older devices(kinda pointless, but good to test):
# -> will set target_full_treble to true so not_full_treble() macros are
# invalid now
PRODUCT_SEPOLICY_SPLIT_OVERRIDE := true

# Change to permissive to test for failures:
BOARD_USE_ENFORCING_SELINUX := false
```
