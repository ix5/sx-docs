+++
title = "SElinux on Android: Part 2"
description = "Treble and onward: Security changes"
date = 2018-12-22T18:07:04+01:00
draft = true
+++

## Stuff:
- sepolicy split
  - `PRODUCT_x` vars
  - move into /system/etc/selinux/
  - move into /vendor/etc/selinux/
  - now 2 policies(instead of one binary `sepolicy` file in root dir,
    definitions from one are no longer accessible from the other!
  - build system change: vendor_service_contexts to vndservice_contexts
- how public and vendor/private dirs are fused now
- move from binary to .cil format
- ld.config.28.txt changes
- macros for treble support and why to avoid them

```
# Platform witout a vendor partition
TARGET_COPY_OUT_VENDOR := system/vendor
# Force split
# (not working yet without real split vendor part)
#BOARD_PROPERTY_OVERRIDES_SPLIT_ENABLED := true

# Test split sepolicy on older devices:
PRODUCT_SEPOLICY_SPLIT_OVERRIDE := true

# Test full treble
PRODUCT_FULL_TREBLE_OVERRIDE := true

# Change to permissive to test
BOARD_USE_ENFORCING_SELINUX := false

# (not working without real split vendor part because symlinks and ld.config)
BOARD_VNDK_VERSION := current
PRODUCT_SHIPPING_API_LEVEL ?= 26
PRODUCT_USE_VNDK_OVERRIDE := true
# Include vndk/vndk-sp/ll-ndk modules
PRODUCT_PACKAGES += vndk_package
```
