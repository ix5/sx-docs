---
title: "Sony Devices"
description: "Overview of SODP devices"
date: 2019-02-10T21:21:51+01:00
weight: 20
draft: false
bref: "Overview of SODP devices"
toc: true
---

### SDM660 devices (ganges board, 2019)

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia 10          | Kirin     | i3113, i4113     |
| Xperia 10 Plus     | Mermaid   | i3213, i4213     |

### SDM845 devices (tama board, 2018)

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia XZ2         | Akari     | h8216, h8266     |
| Xperia XZ2 Compact | Apollo    | h8314, h8324     |
| Xperia XZ3         | Akatsuki  | h8416, h9436     |

### SDM630 devices (nile board, 2018)

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia XA2         | Pioneer   | h3113, h4113     |
| Xperia XA2 Ultra   | Discovery | h3213, h4213     |
| Xperia XA2 Plus    | Voyager   | h3413, h4413     |

### MSM8998 devices (yoshino board, 2017)

| Device name        | Code name | Product (SS, DS) |
| ------------------ | --------- | ---------------- |
| Xperia XZ Premium  | Maple     | g8131, g8142     |
| XPeria XZ1         | Poplar    | g8341, g8342     |
| Xperia XZ1 Compact | Lilac     | g8441            |

### MSM8996 devices (tone board, 2016)

| Device name          | Code name | Product (SS, DS) |
| -------------------- | --------- | ---------------- |
| Xperia X Performance | Dora      | f8131, f8132     |
| Xperia XZ            | Kagura    | f8331, f8332     |
| Xperia XZs           | Keyaki    | g8231, g8232     |

### MSM8956 devices (loire board, 2016)

| Device name      | Code name | Product (SS, DS) |
| ---------------- | --------- | ---------------- |
| Xperia X         | Suzu      | f5121, f5122     |
| Xperia X Compact | Kugo      | f5321            |
| Xperia Touch     | Blanc     | g1109            |

---

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

<!-- ## Leds, thermals, sensors -->

<!-- ## Camera -->

<!-- ## Proprietary modules -->

<!-- ## SoC/Qualcomm stuff -->

<!-- ## Firmware -->
<!-- TERM=xterm mono UnSIN.exe file.sin -->
