---
title: "Android Q: Build Logs and showcommands"
description: ""
date: 2020-01-10T21:53:35+01:00
draft: false
---

In previous Android versions, you could use `showcommands` as a build target
modifier. Now, you get a warning message:

```
! The argument `showcommands` is no longer supported.
! Instead, the verbose log is always written to a compressed file in the output dir:
!
!   gzip -cd out/verbose.log.gz | less -R
!
! Older versions are saved in verbose.log.#.gz files
```

You can just `zcat out/verbose.log.gz`, older versions will be saved as
`out/verbose.log.1.gz`, `verbose.log.2.gz` and so forth.
