---
title: "Android Init Stages"
description: ""
date: 2019-02-25T16:49:06+01:00
weight: 20
draft: false
bref: ""
toc: false
---

## Stages

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

## Classes
Programs can be started based on init stage triggers:
```
service my_service /bin/mysvc
    user system
    group system
    disabled

on boot
    start my_service
```
But it is more common to assign a `class` to a service, based on which it is
started:
```
service my_service /bin/mysvc
    class core
    user system
    group system
```
The service will be started based on `class_start core` which happens `on boot`.

TODO: Overview of classes

## Changes with encryption
With the introduction of system-wide encryption into Android, the init process
gains a few more tricks:
- vold.decrypt triggers
- `class early_hal` for crucial services needed for decryption, e.g. `keymaster`
  and `gatekeeper` (TODO: Add link to AOSP init.rc for class_start)
- restarting framework, reboot, minimal UI stuff

[fde]: https://source.android.com/security/encryption/full-disk#starting_an_encrypted_device_with_default_encryption
[fbe]: https://source.android.com/security/encryption/file-based
