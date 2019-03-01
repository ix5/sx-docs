---
title: "Android intro"
description: ""
date: 2019-02-09T22:14:54+02:00
weight: 10
draft: true
bref: ""
toc: false
---

## Some base numbers for android development
The gist of it is: It all starts with `repo`. Repo is Google's tool to manage a
large number of git repositories. Written as a python script, repo knows which
git repository is saved to which path in the file system, knows which state each
repo is in, which remotes it tracks; it can update all tracked repos at
once(multi-threaded too).

`repo` is driven by manifest files. They are simple `xml` files describing path,
remote, branch/revision.

Over time, the repo tool got a bit smarter and can now resolve more complicated
situations.

The .git/alternates, symlinks into `.repo/project*`
