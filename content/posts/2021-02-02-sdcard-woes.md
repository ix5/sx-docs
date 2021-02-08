---
title: "SDCard Woes"
date: 2021-02-02T12:58:48+01:00
draft: true
---

Possible sources of sdcard multi-user errors.

---

https://source.android.com/devices/tech/admin/multi-user

fwb overlay:
```
    <!--  Maximum number of supported users -->
    <integer name="config_multiuserMaximumUsers">4</integer>

    <!--  Whether Multiuser UI should be shown -->
    <bool name="config_enableMultiUserUI">true</bool>
```

`frameworks/base/core/res/res/xml/storage_list.xml`:
```
<!-- See storage config details at http://source.android.com/tech/storage/ -->

<StorageList xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- removable is not set in nosdcard product -->
    <storage
        android:mountPoint="/storage/sdcard"
        android:storageDescription="@string/storage_usb"
        android:primary="true" />
</StorageList>
```

`bonito/device.mk`:
```
PRODUCT_CHARACTERISTICS := nosdcard
# Use Sdcardfs
PRODUCT_PRODUCT_PROPERTIES += \
    ro.sys.sdcardfs=1
```
`bonito/product.prop`:
```
# Simulate sdcard on /data/media
persist.fuse_sdcard=true
```


`marlin`/device-common.mk:
```
PRODUCT_CHARACTERISTICS := nosdcard
```

`marlin/vold.fstab`:
```
dev_mount sdcard /storage/sdcard1 auto /devices/msm_sdcc.2/mmc_host
```


`marlin/init.common.rc`:
```
on init
    [...]
    # Support legacy paths
    symlink /sdcard /mnt/sdcard
    symlink /sdcard /storage/sdcard0
```



`device-androidia`
```
<!-- The <device> element should contain one or more <storage> elements.
     Exactly one of these should have the attribute primary="true".
     This storage will be the primary external storage and should have path="/mnt/sdcard".
     Each storage should have both a path and description attribute set.
     The following boolean attributes are optional:

        primary:    this storage is the primary external storage
        removable:  this is removable storage (for example, a real SD card)
        emulated:   the storage is emulated via the FUSE sdcard daemon
        mtp-reserve: number of megabytes of storage MTP should reserve for free storage
                     (used for emulated storage that is shared with system's data partition)

      A storage should not have both emulated and removable set to true
-->

<StorageList xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- internal emulated storage -->
    <storage
        android:storageDescription="@string/storage_internal"
        android:primary="true"
        android:emulated="true"
        android:mtpReserve="100" />
</StorageList>
```

`cel_apl/fstab.recovery`:
```
# Source: device/intel/mixins/groups/storage/sdcard-mmc0-usb-sd/fstab.recovery
/dev/block/mmcblk0p1                    /sdcard          vfat    defaults                                   voldmanaged=sdcard:auto
```
