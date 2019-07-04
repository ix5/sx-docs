---
title: "Persistent Logs"
description: ""
date: 2019-05-23T14:25:53+02:00
draft: false
---

- https://android.googlesource.com/platform/system/core/+/refs/tags/android-9.0.0_r39/logd/README.property
- https://android.googlesource.com/platform/system/core/+/refs/tags/android-9.0.0_r39/logcat/logpersist
- https://android.googlesource.com/platform/system/core/+/refs/tags/android-9.0.0_r39/logcat/logcatd.rc

```
setprop persist.logd.logpersistd logcatd
```

Enables `logpersist` service
