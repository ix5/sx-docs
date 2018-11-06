+++
title = "Structure"
description = "How to lay out your own build project"
date = 2018-10-19T22:15:22+02:00
weight = 20
draft = false
bref = "How to lay out your source strucure"
toc = true
+++

## AOSP layout

When you set up your Android directory with `repo init`, what happens is the
Google `repo` tool creates a hidden `.repo` folder, which by default has a
`manifest.xml` file.

### manifest.xml and local_manifests
`manifest.xml` is actually a symbolic link to `.repo/manifests/default.xml`. It
contains a description of the directories needed to build android, and where to get
them. Those directories are all just git repositories that get cloned from
[android.googlesource.com](https://android.googlesource.com).
A `<remote>` is just a git remote base url with a default branch specified. A
`<project>` sets `name` and `path` as well as `remote` and `revision`(git
branch).  
Let's take this example:
```
<remote name="sony" fetch="https://github.com/sonyxperiadev/" />
<project path="hardware/qcom/gps" name="hardware-qcom-gps-sdm845" groups="device" remote="sony" revision="aosp/android-9.0.0_r11" />
```
This would just `git clone` the git repo from https://github.com/sonyxperiadev/hardware-qcom-gps-sdm845
into `hardware/qcom/gps`. Every time you run `repo sync` it would fetch the
latest version of the branch `aosp/android-9.0.0_r11` from GitHub.

Instead of editing `manifest.xml`, you can instead put your modifications
inside `.repo/local_manifests/`. This way, if the AOSP default manifest changes,
you do not have to re-apply your changes manually.

To get started with your own `local_manifests`, take a look at
[github.com/sonyxperiadev/local_manifests](https://github.com/sonyxperiadev/local_manifests). If you want to remove a default project, use `<remove-project>`.

### Modules
Every directory with an `Android.mk` or `Android.bp` file in it will get considered.

Most interesting to new developers might be the `frameworks`, `hardware`,
`kernel` and `packages` folders.

## Getting started

**You'll need:**

- `local_manifests` from Sony
- `device-sony-common` from Sony
- `device-sony-<board>` from Sony where `<board>` is your phone platform, e.g.
  `tone` for Xperia XZ
- `device-sony-<model-codename>` from Sony where `<model-codename>` is the
  codename of your device, e.g. `kagura` for Xperia XZ
- `repo` from Google
- `repo_update.sh` script from Sony
- (Optional) Your own script to apply your own patches

### Caveats

- Use the appropriate branch for your things at all times!
  It is tempting to test out patches from `master`, but be consistent.
- Cherry-pick carefully
- If anything goes wrong, use `repo forall -vc "git reset --hard"`
- If you must fork a repository, never commit your changes to the `master`
  branch, create a custom branch instead. This way, you can still pull from the
  "upstream" repo(i.e. the repo you forked it from) to `master` and then merge
  from `master` into your own custom branch. Otherwise you will run into merge
  conflicts very quickly.

### Tips

- It is way easier to write a patch and submit it than forking a
  repository and keeping it in sync with the upstream repo. If the maintainers do not accept your patch but you still
  need it, keep a branch in your forked repository for all your changes and only
  cherry-pick them with your own patch script. Take a look at
  [repo_update.sh](https://github.com/sonyxperiadev/repo_update)
  from sonyxperiadev for inspiration.
- Use overlays instead of modifying android_frameworks_base
- If you want to theme, it may be easier to create a magisk module than to
  recompile your whole build. Also look into Runtime Resource Overlays(RROs).
