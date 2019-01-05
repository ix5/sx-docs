+++
title = "SElinux on Android: Part 1"
description = "Basics: How SELinux works on Android and how it fits into Android's security approach"
date = 2018-12-22T18:07:04+01:00
draft = true
+++

## Stuff:
- why? -> apps spying, /proc/ leaks, ...
- properties, labels, permissions, capabilities
- file layout, `BOARD_SEPOLICY_x` fusion vars
- neverallows
- ld.config.txt
- seccomp
- reading denials, audit2allow
- working with aosp policy, private/public/vendor
