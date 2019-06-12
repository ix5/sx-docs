---
title: "Applying Single CAF Commits"
description: ""
date: 2019-06-04T14:25:45+02:00
draft: true
---

Normally, git either fetches all remote branches or only a specific branch with
`git fetch <remote> <branch>`, but most git servers cannot send a single commit.

Say you want to apply only a commit from CodeAurora Forum(“CAF”) but you do not
know the branch it is on.
E.g. [source.codeaurora.org/quic/la/kernel/msm-4.9/commit?id=3186af9f3ece8d5d07a3d62cc1a64b448eab375b](https://source.codeaurora.org/quic/la/kernel/msm-4.9/commit?id=3186af9f3ece8d5d07a3d62cc1a64b448eab375b)

You can get a `.patch`-formatted version of any commit by exchanging
`/commit?id=` for `/patch?id=`.

You can download that patch and apply it via git's apply-mail function:
`curl 'https://source.codeaurora.org/quic/la/kernel/msm-4.9/patch/?id=3186af9f3ece8d5d07a3d62cc1a64b448eab375b' | git am`.
<!--
```
CAF=https://source.codeaurora.org/quic/la/kernel/msm-4.9
PATCH=3186af9f3ece8d5d07a3d62cc1a64b448eab375b
curl "https://$CAF/patch/?id=$PATCH" \
  | git am
```
-->
