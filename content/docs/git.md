+++
title = "Git tricks"
description = ""
date = 2018-11-19T22:26:47+02:00
weight = 20
draft = true
bref = ""
toc = false
+++

<!-- ## Un-fuck git -->
## How to fix issues with git and repo
In case the Android tree has inconsistencies.

- e.g. `.repo/projects/prebuilts/build-tools.git/shallow` should point to a commit
- e.g. `.repo/project-objects/android_prebuilts_build-tools.git/objects/info/alternates`
  should point to the reference tree's `.repo/projects/prebuilts/build-tools.git/objects`

If those conditions are not valid for your tree, you can delete the project and
project-objects sub-folders for your affected project and re-run repo. Often
times, errors during syncing will be caused by broken files in
`.repo/project-objects`.

<!-- TODO: Add more actual tricks, stashing, repo status, how to check out
gerrit stuff, track remotes, quickly test a PR, merge from other repos, or
cherry-pick -->
