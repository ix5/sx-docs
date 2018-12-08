+++
title = "SELinux"
description = ""
date = 2018-12-19T22:15:22+02:00
weight = 20
draft = true
bref = ""
toc = true
+++

Directories:

- `system/sepolicy`
- `device/sony/sepolicy`
- `device/sony/<platform>/sepolicy_platform`
- `system/bt/vendor_libs/linux/sepolicy`
- `system/extras/boottime_tools/bootio/sepolicy`
- ...

`file_contexts`
`genfs_contexts`

Sepolicy macros

Testing it out: either `mka sepolicy` or `make bootimage`

```
PRODUCT_FULL_TREBLE_OVERRIDE := false
BOARD_USE_ENFORCING_SELINUX := true
```

Treble and changes from Oreo to Pie
`binder_in_service_violators` etc

`binder`, `hwbinder`, `vndbinder`

Interaction with closed-source "secure" stuff, `rild`

Apps, services, files, binaries

How to read denials

Where to look for reference (marlin, crosshatch, crosshatch-sepolicy)
And permalinks to indiviual lines of policy

Things that can be left as-is
