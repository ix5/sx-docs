---
title: "sepolicy for system_as_root devices"
date: 2019-02-12T22:12:00+01:00
draft: false
author: oshmoun
---

For selinux issues, it is usually enough to update `sepolicy` only.
That way, one avoids having to rebuild the whole boot image, which may or may not include rebuilding the kernel, which could take time.
On system_as_root devices, like all A/B devices thus far, things are a bit different. `sepolicy` is still under `/`, if the policy is not split, but it actually lives on the system partition.
That means that updating `sepolicy` on such devices means manually copying the new policy to `/sepolicy`, assuming one has already mounted system_root in rw mode.
