---
title: "Google git-repo: forall"
description: ""
date: 2020-02-05T00:10:09+01:00
draft: false
---

Google's [repo tool][gerrit-git-repo] tool can run a command in all of its
tracked repositories.

Use `-p` to also show the current repository when running.

Example usage: See all the changes a security patch bump introduced:
```
repo forall -p -c \
  "/usr/bin/git --no-pager \
    diff android-10.0.0_r18..android-10.0.0_r25 \
    2>/dev/null"
```

[gerrit-git-repo]: https://gerrit.googlesource.com/git-repo
