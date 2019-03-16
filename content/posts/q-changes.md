---
title: "Android Q Changes"
description: "Mainly to the build system"
date: 2019-03-16T08:18:41+01:00
slug: android-q-changes
draft: false
---

### sepolicy
`sepolicy` has new `neverallows`.

### ccache
`ccache` is no longer supported by default, the prebuilt `ccache` exec has been
removed.

See [build/core/ccache.mk][ccache-q]:

> We no longer provide a `ccache` prebuilt.
> 
> Ours was old, and had a number of issues that triggered non-reproducible
> results and other failures. Newer ccache versions may fix some of those
> issues, but at the large scale of our build servers, we weren't seeing
> significant performance gains from using `ccache` -- you end up needing very
> good locality and/or very large caches if you're building many different
> configurations.
> 
> Local no-change full rebuilds were showing better results, but why not just
> use incremental builds at that point?
> 
> So if you still want to use `ccache`, continue setting `USE_CCACHE`, but also
> set the `CCACHE_EXEC` environment variable to the path to your ccache
> executable.

[ccache-q]: https://android.googlesource.com/platform/build/+/refs/tags/android-q-preview-1/core/ccache.mk#17
