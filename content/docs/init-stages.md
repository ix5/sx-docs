---
title: "Android Init Stages"
description: "Overview"
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

| Class             | Comment        |
| ----------------- | -------------- |
| `core`            | Never shut down after restarting |
| `main`            | Shut down and then restart after the disk password is entered |
| `late_start`      | Does not start until after /data has been decrypted and mounted |

| Trigger           | Classes        |
| ----------------- | -------------- |
| `on charger`      | `charger`      |
| `on late-fs`      | `early-hal`    |
| `on nonencrypted` | `hal`, `core`  |
| `on boot`         | `hal`, `core`  |

## Keywords (selection)
| Keyword       | Action                 |
| ------------- | ---------------------- |
| `start`       | Starts a service, even when it's disabled |
| `class core`  | Makes a service part of a group.          |
| `class_start` | Starts a group. Once this group is started, the service is started only if it is enabled (services can be disabled by having `disabled` in the service spec, or `disable <service>`) |
| `enable`      | Allows a service to be started as part of a `class_start`. If the class is already started, this'll schedule the service to be started |
| `disable`     | Prevent a service from being started as part of `class_start`. This does not stop the service if it's already running |

## Changes with encryption
With the introduction of system-wide encryption into Android, the init process
gains a few more tricks:
- vold.decrypt triggers
- `class early_hal` for crucial services needed for decryption, e.g. `keymaster`
  and `gatekeeper` (TODO: Add link to AOSP `init.rc` for `class_start`)
- restarting framework, reboot, minimal UI stuff
- `nonencrypted` trigger, which, despite its name, always gets triggered and
  which pulls in `class_start main` and `class_start late_start`

```
on property:vold.decrypt=trigger_restart_min_framework
    class_start main

on property:vold.decrypt=trigger_restart_framework
    stop surfaceflinger
    start surfaceflinger
    class_start main
    class_start late_start

on property:vold.decrypt=trigger_shutdown_framework
    class_reset late_start
    class_reset main
```
Gone in master/Q: [Stop using trigger_reset_main][trigger_reset]

[fde]: https://source.android.com/security/encryption/full-disk#starting_an_encrypted_device_with_default_encryption
[fbe]: https://source.android.com/security/encryption/file-based
[trigger_reset]: https://android-review.googlesource.com/c/platform/system/vold/+/952637
