---
title: "Android's toybox"
description: "More capable than you expected"
date: 2019-04-26T18:29:10+02:00
draft: false
author: Felix
---

Android ships a stripped down version of standard Linux commands and tools via
[toybox][toybox]. `toybox` is comparable to [busybox][busybox-wiki], it consists
of a single executable in `/vendor/bin/toybox` which provides all the
functionality.

For example, to view extended file attributes - `xattrs`:
```
kagura:/persist # toybox getfattr -d /persist/
# file: .
security.restorecon_last="�+4����	\�	�X�7tu�"
security.selinux="u:object_r:persist_file:s0"
```

To wipe a particular `xattr` to force `restorecon_recursive` to run on next
boot:
```
kagura:/persist # toybox setfattr -x security.restorecon_last /persist/
kagura:/persist # toybox getfattr -d /persist/
# file: .
security.selinux="u:object_r:persist_file:s0"
```

As of Android 9.0 Pie, `toybox` includes the following commands:

- `acpi`
- `base64`
- `basename`
- `blkid`
- `blockdev`
- `cal`
- `cat`
- `chattr`
- `chcon`
- `chgrp`
- `chmod`
- `chown`
- `chroot`
- `chrt`
- `cksum`
- `clear`
- `cmp`
- `comm`
- `cp`
- `cpio`
- `cut`
- `date`
- `dd`
- `df`
- `diff`
- `dirname`
- `dmesg`
- `dos2unix`
- `du`
- `echo`
- `egrep`
- `env`
- `expand`
- `expr`
- `fallocate`
- `false`
- `fgrep`
- `file`
- `find`
- `flock`
- `fmt`
- `free`
- `freeramdisk`
- `fsfreeze`
- `getenforce`
- `getfattr`
- `grep`
- `groups`
- `gunzip`
- `gzip`
- `head`
- `help`
- `hostname`
- `hwclock`
- `id`
- `ifconfig`
- `inotifyd`
- `insmod`
- `install`
- `ionice`
- `iorenice`
- `iotop`
- `kill`
- `killall`
- `ln`
- `load_policy`
- `log`
- `logname`
- `losetup`
- `ls`
- `lsattr`
- `lsmod`
- `lsof`
- `lspci`
- `lsusb`
- `makedevs`
- `md5sum`
- `microcom`
- `mkdir`
- `mkfifo`
- `mknod`
- `mkswap`
- `mktemp`
- `modinfo`
- `modprobe`
- `more`
- `mount`
- `mountpoint`
- `mv`
- `nbd-client`
- `nc`
- `netcat`
- `netstat`
- `nice`
- `nl`
- `nohup`
- `od`
- `partprobe`
- `paste`
- `patch`
- `pgrep`
- `pidof`
- `pivot_root`
- `pkill`
- `pmap`
- `printenv`
- `printf`
- `ps`
- `pwd`
- `pwdx`
- `readlink`
- `realpath`
- `renice`
- `restorecon`
- `rev`
- `rfkill`
- `rm`
- `rmdir`
- `rmmod`
- `runcon`
- `sed`
- `sendevent`
- `seq`
- `setenforce`
- `setfattr`
- `setprop`
- `setsid`
- `sha1sum`
- `sha224sum`
- `sha256sum`
- `sha384sum`
- `sha512sum`
- `sleep`
- `sort`
- `split`
- `start`
- `stat`
- `stop`
- `strings`
- `stty`
- `swapoff`
- `swapon`
- `sync`
- `sysctl`
- `tac`
- `tail`
- `tar`
- `taskset`
- `tee`
- `time`
- `timeout`
- `top`
- `touch`
- `tr`
- `traceroute`
- `traceroute6`
- `true`
- `truncate`
- `tty`
- `tunctl`
- `ulimit`
- `umount`
- `uname`
- `uniq`
- `unix2dos`
- `uptime`
- `usleep`
- `uudecode`
- `uuencode`
- `vconfig`
- `vmstat`

[toybox]: https://android.googlesource.com/platform/external/toybox/
[busybox-wiki]: https://en.wikipedia.org/wiki/BusyBox
