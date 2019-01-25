---
title: "Android blueprint files"
date: 2019-01-01T08:18:00+01:00
draft: false
author: Felix
---

`androidmk` is a tool to convert Makefile-style `Android.mk` files to
`Android.bp` "blueprint" files, which look like json and are faster to parse.  
YMMV, but they also seem more readable and largely prevent you from doing crazy
if-else and convoluted stuff as you'd be drawn to if using makefiles.  
As a downside, they're an "Android-ism" and make it harder for people to port
your software to a project that is not Android.

Use `make -j 8 blueprint_tools` inside the AOSP tree to build the `androidmk` go
binary.

You can then copy the binary from `out/soong/host/linux-x86/bin/androidmk` to
your `~/.local/bin/` to use it outside the AOSP tree. Beware that the tool is
constantly being updated, so you might need to re-build it from time to time.

See this question about [androidmk on StackOverflow][so-androidmk].

[so-androidmk]: https://stackoverflow.com/questions/51207766/where-can-i-find-androidmk-tool-to-convert-android-mk-to-android-bp
