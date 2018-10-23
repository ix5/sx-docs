+++
title = "Structure"
description = "How to lay out your own build infrastructure"
date = 2018-10-19T22:15:22+02:00
weight = 20
draft = false
bref = "How to lay out your source strucure"
toc = true
+++

**You'll need:**

- `local_manifests` from Sony
- `device-sony-common` from Sony
- `device-sony-<board>` from Sony where `<board>` is your phone platform, e.g.
  `tone` for Xperia XZ
- `device-sony-<model-codename>` from Sony where `<model-codename>` is the
  codename of your device, e.g. `kagura` for Xperia XZ
- `repo` from Google
- Your own script to apply your own patches

### Caveats

- Use the appropriate branch for your things at all times!
  It is tempting to test out patches from `master`, but be consistent.
- Cherry-pick carefully
- If anything goes wrong, use `repo forall -vc "git reset --hard"`

### Tips

- Use overlays instead of modifying android_frameworks_base
- If you want to theme, it may be easier to create a magisk module than to
  recompile your whole build. Also look into Runtime Resource Overlays(RROs).
