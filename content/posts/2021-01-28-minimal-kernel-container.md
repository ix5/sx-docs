---
title: "Creating a Minimal Container for Kernel Builds"
date: 2021-01-28T18:37:28+01:00
author: Felix
slug: "minimal-build-container"
draft: true
---

What we need:

- clang
- cross compiler arm32
- gcc fallback
- mkbootimg + small lib
- mkdtimg
- python, git, ...

The beauty of a dedicated container is that it does not need to be able to run
any "regular" server tasks. We can strip away everything not strictly required
for the build process from the resulting image.
