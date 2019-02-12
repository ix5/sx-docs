---
title: "Gotcha: sepolicy for system-as-root devices"
date: 2019-02-12T22:12:00+01:00
draft: false
author: oshmoun
---

For testing a new SELinux policy, it used to be enough to update only the
`sepolicy` binary file in `out/target/product/<device>/root/sepolicy` and
re-pack the boot image. That way, one avoids having to rebuild the whole boot
image, which may or may not include rebuilding the kernel, which could take
time.

On [system-as-root devices][sys-as-root] - all A/B devices thus far - things are
a bit different. `sepolicy` is still located under `/`(if the policy is not
split, more on that later), but it actually lives on the system partition(under
`/system_root/`).
That means that updating `sepolicy` on such devices means manually copying the
new policy to `/sepolicy`, assuming one has already mounted `system_root` in rw
mode.

<div class="message warning">
  <h5>Note</h5>
Pushing an updated `boot.img` to the device will not update the used
<code>sepolicy</code>. The file inside the boot partition is an unused duplicate
on system-as-root devices!
</div>

On `FULL_TREBLE` devices, `PRODUCT_SEPOLICY_SPLIT` is true, which means the
sepolicy will no longer be built as a unified binary, but rather pushed as
`.cil` plain-text files under `/system/etc/selinux/` and `/vendor/etc/selinux/`.
There is no longer a single `sepolicy` binary to update in the ramdisk, making
GSIs possible.

[sys-as-root]: https://source.android.com/devices/bootloader/system-as-root
