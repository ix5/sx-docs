---
title: "Kagura Stock /persist Labeling"
date: 2019-05-16T10:42:05+02:00
draft: true
---

# On stock

`/persist` block device:
```
lrwxrwxrwx 1 root   root   21 1970-05-02 23:33 persist -> /dev/block/mmcblk0p44
```

Recovery:
```
# mkdir /persist
# mount /dev/block/mmcblk0p44 /persist
~ # ls -laZ /persist/
drwxrwx--x   18      4096 Aug  5 18:15 u:object_r:persist_file:s0       .
drwxrwxrwt   29      1100 Aug  5 18:20 u:object_r:rootfs:s0             ..
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_alarm_file:s0 alarm
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_bluetooth_file:s0 bluetooth
drwx------    2      4096 Aug  5 18:15 u:object_r:persist_bms_file:s0   bms
drwx------    4      4096 Aug  5 18:15 u:object_r:persist_drm_file:s0   data
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_display_file:s0 display
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_drm_file:s0   drm
drwxrwx---    3      4096 Aug  5 18:15 u:object_r:rfs_shared_hlos_file:s0 hlos_rfs
drwx------    2      4096 Jan  1  1970 u:object_r:persist_file:s0       lost+found
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_misc_file:s0  misc
drwxrwx---    2      4096 Sep 10  2018 u:object_r:simpin_cache_data_file:s0 pc
drwx------    2      4096 Aug  5 18:15 u:object_r:persist_qti_fp_file:s0 qti_fp
drwx------    6      4096 Aug  5 18:15 u:object_r:rfs_file:s0           rfs
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_secnvm_file:s0 secnvm
drwxr-xr-x    3      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 sensors
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_time_file:s0  time
drwxrwx---    2      4096 Aug  5 18:15 u:object_r:persist_vpp_file:s0   vpp
```

```
~ # ls -laZ /persist/sensors/
drwxr-xr-x    3      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 .
drwxrwx--x   18      4096 Aug  5 18:15 u:object_r:persist_file:s0       ..
drwx------    3      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 registry
-rw-r--r--    1         2 Oct 16  2018 u:object_r:sensors_persist_file:s0 sensors_settings
-rwx------    1     25468 Aug  5 18:15 u:object_r:sensors_persist_file:s0 sns.reg
-rw-------    1        12 Aug  5 18:15 u:object_r:sensors_persist_file:s0 version.txt
~ # ls -laZ /persist/sensors/registry/
drwx------    3      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 .
drwxr-xr-x    3      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 ..
drwx------    2      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 registry
~ # ls -laZ /persist/sensors/registry/registry/
drwx------    2      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 .
drwx------    3      4096 Aug  5 18:15 u:object_r:sensors_persist_file:s0 ..
~ # toybox getfattr -d /persist/
# file: /persist/
security.restorecon_last="*�Q�Cr��.e#�C:��"
security.selinux="u:object_r:persist_file:s0"
```

# On AOSP
(Without running `toybox setfattr -x security.restorecon_last /mnt/vendor/persist/` in init)
```
kagura:/ # toybox getfattr -d /persist/
# file: /persist/
security.restorecon_last="#�7@=��Bit{���?�X�"
security.selinux="u:object_r:persist_file:s0"

kagura:/ # ls -laZ /persist/sensors/
total 52
drwxr-xr-x  3 system system u:object_r:persist_sensors_file:s0  4096 1970-08-05 18:15 .
drwxrwx--x 19 root   system u:object_r:persist_file:s0          4096 1970-08-05 18:35 ..
drwx------  3 system system u:object_r:persist_sensors_file:s0  4096 1970-08-05 18:15 registry
-rw-r--r--  1 system system u:object_r:persist_sensors_file:s0     2 2018-10-16 15:24 sensors_settings
-rwx------  1 system system u:object_r:persist_sensors_file:s0 25352 1970-08-05 18:35 sns.reg
-rw-------  1 system system u:object_r:persist_sensors_file:s0    12 1970-08-05 18:15 version.txt
```

from Marijn:
```
mermaid:/ # cat /system/etc/selinux/plat_file_contexts /vendor/etc/selinux/vendor_file_contexts | sha1sum
0a0dd50f05ef6d235cebae7e0ed96fd78060a975  -
mermaid:/ # toybox getfattr -d /mnt/vendor/persist/ | xxd
00000000: 2320 6669 6c65 3a20 2f6d 6e74 2f76 656e  # file: /mnt/ven
00000010: 646f 722f 7065 7273 6973 742f 0a73 6563  dor/persist/.sec
00000020: 7572 6974 792e 7265 7374 6f72 6563 6f6e  urity.restorecon
00000030: 5f6c 6173 743d 220a 0dd5 0f05 ef6d 235c  _last="......m#\
00000040: ebae 7e0e d96f d780 60a9 7522 0a73 6563  ..~..o..`.u".sec
00000050: 7572 6974 792e 7365 6c69 6e75 783d 2275  urity.selinux="u
00000060: 3a6f 626a 6563 745f 723a 7065 7273 6973  :object_r:persis
00000070: 745f 6669 6c65 3a73 3022 0a0a            t_file:s0"..
```
