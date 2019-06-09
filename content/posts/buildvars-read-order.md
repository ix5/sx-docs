---
title: "Android Build Variable Read Order"
description: ""
date: 2019-03-18T09:33:13+01:00
draft: false
---

**Update:**: Just use `make product-graph`.
Then convert the generated `.dot` file to pdf:
```
dot -Tpdf -Nshape=box -o out/products.pdf out/products.dot
```
...or svg:
```
dot -Tsvg -Nshape=box -o out/products.svg out/products.dot
```

<!--
Build output:
Product graph DOT: out/products.dot for device/sony/kagura/aosp_f8331.mk
argv: ['build/make/tools/filter-product-graph.py', 'device/sony/kagura/aosp_f8331.mk']
[100% 8/8] Product graph .dot file: out/products.dot
Command to convert to pdf: dot -Tpdf -Nshape=box -o out/products.pdf out/products.dot
Command to convert to svg: dot -Tsvg -Nshape=box -o out/products.svg out/products.dot
-->


## Sony Open Devices Project: Read order of build variables

- `kagura/vendorsetup.sh` (only for `build/envsetup.sh`)
  Sets up the lunch combo, which then sets the environment variables
  `TARGET_PRODUCT`, `TARGET_BUILD_VARIANT`, `TARGET_PLATFORM_VERSION`,
  `TARGET_BUILD_TYPE=release`
- `kagura/AndroidProducts.mk`
- `kagura/aosp_f8331.mk`
- `kagura/aosp_f8332.mk`
- `kagura/device.mk`
- `tone/platform.mk`
- `common/common.mk`
- (`customization/customization.mk`)
- `common/common-init.mk`
- `common/common-odm.mk`
- `common/common-packages.mk`
- `common/common-perm.mk`
- `common/common-prop.mk`
- `common/common-treble.mk`

- `kagura/BoardConfig.mk`
- `tone/PlatformConfig.mk`
- `common/CommonConfig.mk`
- (customization/Customization.mk)
- (`common-headers/KernelHeaders.mk`)
- (`common-kernel/KernelConfig.mk`)
- `sepolicy/sepolicy.mk`

<!-- TODO: Which gets read first by the build system? -->

## Internal Order

Short:

- `build/core/main.mk`
- `build/core/config.mk`
- `build/core/envsetup.mk`
- `build/core/product_config.mk`
- `BoardConfig.mk`
- `build/core/product.mk`
- `build/core/device.mk`

`build/core/main.mk`
```
BUILD_SYSTEM := $(TOPDIR)build/make/core
include $(BUILD_SYSTEM)/config.mk
include $(BUILD_SYSTEM)/definitions.mk
```
Then, include all subdir makefiles:
```
subdir_makefiles := $(SOONG_ANDROID_MK) $(file <$(OUT_DIR)/.module_paths/Android.mk.list)
$(foreach mk,$(subdir_makefiles),$(info [$(call inc_and_print,subdir_makefiles_inc)/$(subdir_makefiles_total)] including $(mk) ...)$(eval include $(mk)))
```

```
include $(BUILD_SYSTEM)/Makefile
```

`build/core/config.mk`
```
include $(BUILD_SYSTEM)/envsetup.mk
```
<!-- include $(BUILD_SYSTEM)/combo/select.mk -->

`build/core/envsetup.mk`
```
include $(BUILD_SYSTEM)/product_config.mk
board_config_mk := \
	$(strip $(sort $(wildcard \
		$(SRC_TARGET_DIR)/board/$(TARGET_DEVICE)/BoardConfig.mk \
		$(shell test -d device && find -L device -maxdepth 4 -path '*/$(TARGET_DEVICE)/BoardConfig.mk') \
		$(shell test -d vendor && find -L vendor -maxdepth 4 -path '*/$(TARGET_DEVICE)/BoardConfig.mk') \
	)))
include $(board_config_mk)
```
That means: `$board_config_mk` = `device/$VENDOR/$DEVICE/BoardConfig.mk`

`build/core/product_config.mk`
```
include $(BUILD_SYSTEM)/product.mk
include $(BUILD_SYSTEM)/device.mk
```

**TODO:** Nice charts with relations

[elinux]: https://elinux.org/Android_Device
