---
title: "Tone Battery & Storage Features"
slug: "tone-battery-and-storage-features"
description: ""
date: 2019-07-04T07:08:19+02:00
draft: false
---

While researching a bit about the battery management of MSM8996 for implementing
storage statistics for a [health HAL][health-hal], a few notes:

## Battery dump
```
kagura:/ # ls /sys/class/power_supply/battery/
allow_hvdcp3
battery_charging_enabled
battery_type
capacity
charge_control_limit
charge_control_limit_max
charge_full
charge_full_design
charge_type
charging_enabled
chgerr_sts
constant_charge_current_max
current_now
cycle_count
device
dp_dm
enable_shutdown_at_low_battery
flash_active
flash_current_max
flash_trigger
fv_cfg
fv_cmp_cfg
health
input_current_limited
input_current_max
input_current_now
input_current_settled
input_current_state
int_cld
lrc_enable
lrc_not_startup
lrc_socmax
lrc_socmin
max_pulse_allowed
power
present
rerun_aicl
resistance_id
restricted_charging
safety_timer_enabled
smart_charging_activation
smart_charging_interruption
smart_charging_status
status
subsystem
system_temp_level
technology
temp
type
uevent
voltage_max
voltage_max_design
voltage_now
```

## BMS dump

```
kagura:/ # ls /sys/class/power_supply/bms/
batt_aging
battery_type
capacity
capacity_raw
charge_full
charge_full_design
charge_now
charge_now_error
charge_now_raw
current_now
cycle_count
cycle_count_id
device
esr_count
hi_power
power
resistance
resistance_id
subsystem
temp
temp_cool
temp_warm
type
uevent
update_now
voltage_max_design
voltage_min
voltage_now
voltage_ocv
```

## Storage devices

```
kagura:/ # readlink /sys/block/mmcblk0
../devices/platform/soc/7464900.sdhci/mmc_host/mmc0/mmc0:0001/block/mmcblk0
kagura:/ # realpath /sys/block/mmcblk0
/sys/devices/platform/soc/7464900.sdhci/mmc_host/mmc0/mmc0:0001/block/mmcblk0
```

## Storage device features

```
kagura:/ # ls /sys/devices/platform/soc/7464900.sdhci/mmc_host/mmc0/mmc0\:0001/
block/
cid
csd
date
-> 07/2016
driver
dsr
enhanced_area_offset
enhanced_area_size
enhanced_rpmb_supported
erase_size
ffu_capable
fwrev
hwrev
life_time
-> 0x02 0x02
manfid
name
ocr
oemid
-> 0x0100
power
pre_eol_info
-> 01
preferred_erase_size
prv
raw_rpmb_size_mult
rel_sectors
rev
-> 0x8
serial
-> 0xXXXXXXXX
subsystem
type
uevent
```

Pretty neat, gives us all we need: Lifetime-A(`0x02`), Lifetime-B(`0x02`), and
pre-EOL info(`01`).

## Block devices

```
kagura:/ # ls /sys/devices/platform/soc/7464900.sdhci/mmc_host/mmc0/mmc0\:0001/block/mmcblk0/
alignment_offset
discard_alignment
mmcblk0p10
mmcblk0p16
mmcblk0p21
mmcblk0p27
mmcblk0p32
mmcblk0p38
mmcblk0p43
mmcblk0p49
mmcblk0p54
mmcblk0rpmb
removable
uevent
badblocks
ext_range
mmcblk0p11
mmcblk0p17
mmcblk0p22
mmcblk0p28
mmcblk0p33
mmcblk0p39
mmcblk0p44
mmcblk0p5
mmcblk0p55
no_pack_for_random
ro
bdi
force_ro
mmcblk0p12
mmcblk0p18
mmcblk0p23
mmcblk0p29
mmcblk0p34
mmcblk0p4
mmcblk0p45
mmcblk0p50
mmcblk0p6
num_wr_reqs_to_start_packing
size
capability
holders
mmcblk0p13
mmcblk0p19
mmcblk0p24
mmcblk0p3
mmcblk0p35
mmcblk0p40
mmcblk0p46
mmcblk0p51
mmcblk0p7
power
slaves
dev
inflight
mmcblk0p14
mmcblk0p2
mmcblk0p25
mmcblk0p30
mmcblk0p36
mmcblk0p41
mmcblk0p47
mmcblk0p52
mmcblk0p8
queue
stat
device
mmcblk0p1
mmcblk0p15
mmcblk0p20
mmcblk0p26
mmcblk0p31
mmcblk0p37
mmcblk0p42
mmcblk0p48
mmcblk0p53
mmcblk0p9
range
subsystem
```

```
kagura:/ # cat /sys/devices/platform/soc/7464900.sdhci/mmc_host/mmc0/mmc0\:0001/block/mmcblk0/stat
    2227      191   154756     3016     4937     6565   221592    32364        0    13429    35336
```

---

## UFS/MMC specifications

According to a [spec sheet from Kingston][kingston] [^1], see chapter 5.17
"Device Life time" and 5.18 "Pre EOL Information".

### Device Life Time Set Type A

> This field provides an estimated indication of the device life time which is
> reflected by the average wear out of memory of Type B relative to its maximum
> estimated device life time

| Value  | Description                       |
| ------ | --------------------------------- |
| 0x00   | Not Defined                       |
| 0x01   | 0% - 10% Device Life Time Used    |
| 0x02   | 10% - 20% Device Life Time Used   |
| 0x03   | 20% - 30% Device Life Time Used   |
| 0x04   | 30% - 40% Device Life Time Used   |
| 0x05   | 40% - 50% Device Life Time Used   |
| 0x06   | 50% - 60% Device Life Time Used   |
| 0x07   | 60% - 70% Device Life Time Used   |
| 0x08   | 70% - 80% Device Life Time Used   |
| 0x09   | 80% - 90% Device Life Time Used   |
| 0x0A   | 90% - 100% Device Life Time Used  |
| 0x0B   | Exceeded Maximum Device Life Time |
| Others | Reserved                          |

### Device Life Time Set Type B

> This field provides an estimated indication of the device life time which is
> reflected by the average wear out of memory of Type A relative to its maximum
> estimated device life time

| Value  | Description                       |
| ------ | --------------------------------- |
| 0x00   | Not Defined                       |
| 0x01   | 0% - 10% Device Life Time Used    |
| 0x02   | 10% - 20% Device Life Time Used   |
| 0x03   | 20% - 30% Device Life Time Used   |
| 0x04   | 30% - 40% Device Life Time Used   |
| 0x05   | 40% - 50% Device Life Time Used   |
| 0x06   | 50% - 60% Device Life Time Used   |
| 0x07   | 60% - 70% Device Life Time Used   |
| 0x08   | 70% - 80% Device Life Time Used   |
| 0x09   | 80% - 90% Device Life Time Used   |
| 0x0A   | 90% - 100% Device Life Time Used  |
| 0x0B   | Exceeded Maximum Device Life Time |
| Others | Reserved                          |

### Pre EOL Information

> This field provides information about device life time reflected by average
> reserved blocks.

| Value       | Pre-EOL Info. | Description                     |
| ----------- | ------------- | ------------------------------- |
| 0x00        | Not Defined   | Not Defined                     |
| 0x01        | Normal        | Normal                          |
| 0x02        | Warning       | Consumed 80% of Reserved Blocks |
| 0x03        | Urgent        |                                 |
| 0x04 – 0xFF | Reserved      |                                 |

[health-hal]: https://github.com/ix5/vendor-sony-oss-health/
[kingston]: https://www.datasheets.com/datasheet/EMMC08G-W100-A06U-Kingston%20Technology-62609778

[^1]: Spec sheet "Embedded Multi-Media Card" - EMMC08G-W100-A06U, Kingston Technology Company, Inc. - 2015-1-22
