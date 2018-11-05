+++
title = "Security"
description = "Security, verified boot and encryption"
date = 2018-10-19T22:15:22+02:00
weight = 20
draft = false
bref = "Security, verified boot and encryption"
toc = true
+++

Android has a lot of security features built in. However, when testing and
debugging those are really inconvenient to developers. As an end user, you
should be using a ROM which has all the security features enabled, with the
latest security patches.

If tweaking and experimenting is important to you, you can disable some of those
security features; but be warned, they're there for good reasons.

<mark>For developers:</mark> Please do not jeopardize your users' security by lazily turning
off security features. In the early stages of porting it is ok to be lax for
debugging reasons, but you should find the root causes of e.g. sepolicy denials
instead of blankly giving out access to anything.

## Verified boot with dm-verity
For more information see [Implementing dm-verity](https://source.android.com/security/verifiedboot/dm-verity).

<mark>For developers:</mark> Disable dm-verity by ripping out the
`PRODUCT_SYSTEM_VERITY_PARTITION` block and disabling the `call` to
`build/target/product/verity.mk`.

## Encryption
The /data and /userdata(a.k.a. the "internal SD card") partitions can be
encrypted("`encryptable`"). On recent Android versions, those partitions are
encrypted by default and can not be turned off("`forceencrypt`").

<mark>For developers:</mark> Since some tools like the TWRP custom recovery do not have
proper support for encrypted partitions, it may be advisable to revert this hard
"must" back to "can" to give the choice to the users.
Edit `fstab.<devicename>` and find a line which looks like the following:
```
/dev/block/bootdevice/by-name/userdata  /data  ext4  [...]  [...],forceencrypt=footer,quota
```
Change `forceencrypt` to `encryptable`.

## Root and Magisk
**Root:** "Root" means you have full access to the system you are running. Most
of the time, you do not want this, rather you want a specific program to run a
specific privileged action only. If everything on your system has root access,
your system will be taken over by viruses in no time.

That is why you'd want a program to manage root access and only give it out to
programs you trust. **Magisk Manager** is such a program. You need to enable
root the "Magisk way" by flashing the Magisk zip, which sets up root access on
your system so that only the Magisk Manager app can manage it.

**SafetyNet:** Simplified, SafetyNet is a way of telling applications that
"everything is on stock, no root" on your device. Apps can check if SafetyNet is
"triggered" and may refuse to start if it is.

**Hiding root:** Some apps, such as banking apps and Netflix, demand that you do
not run a custom ROM or have root access enabled on your system.
They can be tricked into thinking you are running a stock ROM with various
methods, some of which can be found in the Magisk repositories.

## SELinux

**Enforcing vs permissive:** You can set SELinux to permissive, which means it
will only log policy violations instead of blocking them. Again, ok for
development, not ok to risk your users' safety by lazily setting it to
`permissive`.  
<mark>For developers:</mark> A good guide to quickly fix issues is to read `adb
logcat` and watch out for lines that look like this:
```
avc: denied { action } for pid=...
```
Then read this guide:
[How to write sepolicy to fix a denial](https://gist.github.com/MSF-Jarvis/ec52b48eb2df1688b7cbe32bcd39ee5f)

**Priv-app permissions**: Privileged apps residing in `/system/priv-app` can
have access to system functions which normal apps(e.g. from the Play Store) can
not access. They need to declare which elevated privileges they want, and the
system developer maker needs to declare which permissions should be granted to
these apps.

Most of the time issues with privapp-permissions arise with GApps, because
Google's apps demand ever increasing power over the system. You should ask the
developers of your GApps variant to fix these issues instead of asking your ROM
maker to disable privapp-permissions.

<mark>For developers:</mark> Disable privapp-permissions-enforcing by ripping out the following block:
```
PRODUCT_PROPERTY_OVERRIDES += ro.control_privapp_permissions=enforce
```
