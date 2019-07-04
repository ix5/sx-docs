---
title: "Building Android master for SODP"
description: "Navigating rough waters"
date: 2019-06-02T14:00:00+02:00
draft: true
---

- Buildvars
- APEX: flatten or not?
- Dynamic partitions
- SELinux version
- Finding non-existent `PRODUCT_PACKAGES`

- Manifest
- `local_manifests` branches
- `repo_update.sh`
- `q_repo_update.sh` (vendor-qcom-opensource-location, PRs to merge and patches etc.)
 (basically list all the Q-COMPAT open PRs)

- TODO: Set prebuilt kernel
- TODO: Set TARGET_COMPILE_WITH_MSM_KERNEL to BUILD_KERNEL
- TODO: New HALs: health.storage, debug(dumpstate, atrace)
- TODO: Patches to device repos to switch to new vendorsetup
