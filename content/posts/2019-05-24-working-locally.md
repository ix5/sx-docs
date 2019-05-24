---
title: "SODP: Working Locally"
description: ""
date: 2019-05-24T11:48:01+02:00
draft: false
---

To reset your Android source tree to the branches specified in your manifests,
use `repo sync --current-branch -l -j 8 --force-sync`.  
The `-l` means to not connect to the network.

To only fetch the newest references without modifying the checked-out tree, use
`-n` instead of `-l`.

We also made a change to `repo_update.sh` that enables you to work offline after
having fetched the relevant commits once:
[Look for commits locally before fetching][repo_update].

[repo_update]: https://github.com/sonyxperiadev/repo_update/pull/96
