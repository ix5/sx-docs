---
title: "Android Init Stages"
description: ""
date: 2019-02-25T16:49:06+01:00
weight: 20
draft: false
bref: ""
toc: false
---

| Init stage     | Contents                                                                 |
| -------------- | ------------------------------------------------------------------------ |
| `early-init`   | Very first stage of initialization. Used for SELinux and OOM settings    |
| `init`         | Creates file systems, mount points, and writes kernel variables          |
| `early-fs`     | Run just before filesystems are ready to be mounted                      |
| `fs`           | Specifies which partitions to load                                       |
| `post-fs`      | Commands to run after filesystems (other than `/data`) are mounted       |
| `post-fs-data` | `/data` decrypted (if necessary) and mounted                             |
| `early-boot`   | Run after property service has been initialized, but before booting rest |
| `boot`         | Normal boot commands                                                     |
| `charger`      | Commands used when device is in charger mode                             |

*Adapted from Jonathan Levin's excellent
[Android Internals: A Confectioner's Cookbook](http://www.newandroidbook.com/)*, chapter 4.
