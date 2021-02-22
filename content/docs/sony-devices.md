---
title: "Sony Devices"
slug: "sony-devices"
description: "Overview of SODP devices"
date: 2019-02-10T21:21:51+01:00
weight: 20
draft: false
bref: "Overview of SODP devices"
toc: true
---

### Snapdragon 865 devices (edo board, 2020)
Internal SoC codename: `sm8250`

Platform: `edo`

Latest Open Devices Kernel: 4.19

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia 1 II        | pdx203    | xqat51, xqat52   |
| Xperia 5 II        | pdx206    | xqas52           |

**Peculiarities:** [Dynamic Android Partitions][dap], A/B,
has DTBO partition.

### Snapdragon 665 devices (seine board, 2020)
Internal SoC codename: `sdm660`/`sm6125`

Platform: `seine`

Latest Open Devices Kernel: 4.14

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia 10 II       | pdx201    | xqau51, xqau52   |

**Peculiarities:** First platform with [Dynamic Android Partitions][dap], A/B,
has DTBO partition.

### Snapdragon 855 devices (kumano board, 2019)
Internal SoC codename: `sm8150`

Platform: `kumano`

Latest Open Devices Kernel: 4.14

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia 1           | Griffin   | j8110, j9110     |
| Xperia 5           | Bahamut   | j8210, j9210     |

**Peculiarities:** A/B, has DTBO partition.

### Snapdragon 660 devices (ganges board, 2019)
Internal SoC codename: `sdm660`

Platform: `ganges`

Latest Open Devices Kernel: 4.14

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia 10          | Kirin     | i3113, i4113     |
| Xperia 10 Plus     | Mermaid   | i3213, i4213     |

**Peculiarities:** A/B.

### Snapdragon 845 devices (tama board, 2018)
Internal SoC codename: `sdm845`

Platform: `tama`

Latest Open Devices Kernel: 4.14

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia XZ2         | Akari     | h8216, h8266     |
| Xperia XZ2 Compact | Apollo    | h8314, h8324     |
| Xperia XZ3         | Akatsuki  | h8416, h9436     |

**Peculiarities:** A/B, has DTBO partition.

### Snapdragon 630 devices (nile board, 2018)
Internal SoC codename: `sdm660` (yes, not 630)

Platform: `nile`

Latest Open Devices Kernel: 4.14

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia XA2         | Pioneer   | h3113, h4113     |
| Xperia XA2 Ultra   | Discovery | h3213, h4213     |
| Xperia XA2 Plus    | Voyager   | h3413, h4413     |

**Peculiarities:** A/B.

### Snapdragon 835 devices (yoshino board, 2017)
Internal SoC codename: `msm8998`

Platform: `yoshino`

Latest Open Devices Kernel: 4.14

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia XZ Premium  | Maple     | g8141, g8142     |
| Xperia XZ1         | Poplar    | g8341, g8342     |
| Xperia XZ1 Compact | Lilac     | g8441            |

**Peculiarities:** Separate `/vendor` partition on poplar and lilac. No A/B.

### Snapdragon 820 devices (tone board, 2016)
Internal SoC codename: `msm8996`

Platform: `tone`

Latest Open Devices Kernel: 4.9

| Device name          | Code name | Product (SS, DS) |
| -------------------- | --------- | ---------------- |
| Xperia X Performance | Dora      | f8131, f8132     |
| Xperia XZ            | Kagura    | f8331, f8332     |
| Xperia XZs           | Keyaki    | g8231, g8232     |

**Peculiarities:** Legacy.

### Snapdragon 650 devices (loire board, 2016)
Internal SoC codename: `msm8956`, sometimes `msm8954`

Platform: `loire`

Latest Open Devices Kernel: 4.9

| Device name      | Code name | Product (SS, DS) |
| ---------------- | --------- | ---------------- |
| Xperia X         | Suzu      | f5121, f5122     |
| Xperia X Compact | Kugo      | f5321            |
| Xperia Touch     | Blanc     | g1109            |

**Peculiarities:** Legacy.

---

## Basebands
See `ro.baseband` or `ro.boot.baseband`.
- `mdm`: External Modem
- `msm`/`sdm`: Internal Modem
- `apq`: No Modem

# Internals

## Accessing special modes
**Fastboot**: Turn off device, then hold volume-up and connect cable. LED will
be blue.

**Download mode**: Turn off device, then hold volume-down and connect cable. LED
will be green.

**Recovery**: Turn off device, then hold volume-down and power button

**Forced reboot**: Hold volume-up and power until the first vibration

**Forced shut off** Hold volume-up and power until the second, longer
vibration(pulses three times)

## Device internals

### kagura
```
kagura:/ # cat /sys/devices/soc0/soc_id
246
kagura:/ # cat /sys/devices/soc0/revision
3.1
```

**Mount pstore:**
```
# Don't forget the - dash!
mount -t pstore - /sys/fs/pstore
```

**Get MCC/MNC:**
```
cat /data/vendor/radio/iccid*
```

<!-- ## Leds, thermals, sensors -->

<!-- ## Camera -->

<!-- ## Proprietary modules -->

<!-- ## SoC/Qualcomm stuff -->

<!-- ## Firmware -->
<!-- TERM=xterm mono UnSIN.exe file.sin -->

## Misc

**Mapping device - target:**
```
pdx203    | xqat51, xqat52
pdx206    | xqas52
pdx201    | xqau51, xqau52
griffin   | j8110, j9110
bahamut   | j8210, j9210
kirin     | i3113, i4113
mermaid   | i3213, i4213
akari     | h8216, h8266
apollo    | h8314, h8324
akatsuki  | h8416, h9436
pioneer   | h3113, h4113
discovery | h3213, h4213
voyager   | h3413, h4413
maple     | g8141, g8142
poplar    | g8341, g8342
lilac     | g8441
dora      | f8131, f8132
kagura    | f8331, f8332
keyaki    | g8231, g8232
suzu      | f5121, f5122
kugo      | f5321
blanc     | g1109
```

**List of all targets - both SS/DS:**
```
find . -iname 'aosp_*.mk' | cut -d '/' -f 3 | cut -d '.' -f 1
aosp_h8216
aosp_h8266
aosp_h8416
aosp_h9436
aosp_h8314
aosp_h8324
aosp_g1109
aosp_h3213
aosp_h4213
aosp_f8131
aosp_f8132
aosp_f8331
aosp_f8332
aosp_g8231
aosp_g8232
aosp_i3113
aosp_i4113
aosp_f5321
aosp_g8441
aosp_g8141
aosp_g8142
aosp_i3213
aosp_i4213
aosp_h3113
aosp_h4113
aosp_g8341
aosp_g8342
aosp_f5121
aosp_f5122
aosp_h3413
aosp_h4413
aosp_j8210
aosp_j9210
aosp_j8110
aosp_j9110
aosp_xqau51
aosp_xqau52
aosp_f8331
aosp_f8332
aosp_xqat51
aosp_xqat52
aosp_xqas52
```

**List of all targets - only one per device:**
```
xqat51
xqas52
xqau51
j8110
j8210
i3113
i3213
h8216
h8314
h8416
h3113
h3213
h3413
g8141
g8341
g8441
f8131
f8331
g8231
f5121
f5321
g1109
```
Single-SIM preferred for this list.

[dap]: https://source.android.com/devices/tech/ota/dynamic_partitions
