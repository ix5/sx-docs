---
title: "Android Q: Clang got an Upgrade"
description: ""
date: 2019-12-23T00:21:03+01:00
draft: false
---

In the current Android `master` branch, the default clang version got upgraded
from 9.0.3 (Q) to 10.0.1.

The current version is defined in [build/soong/cc/config/global.go][soong]:
```go
[...]
// prebuilts/clang default settings.
ClangDefaultBase         = "prebuilts/clang/host"
ClangDefaultVersion      = "clang-r370808"
ClangDefaultShortVersion = "10.0.1"
```

[soong]: https://android.googlesource.com/platform/build/soong/+/7e143af6ee16efad245922acd7baecbd980d2f98/cc/config/global.go#127
