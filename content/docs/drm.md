+++
title = "Sony DRM"
description = ""
date = 2018-10-19T22:26:47+02:00
weight = 20
draft = true
bref = ""
toc = true
+++

With Sony devices, lot of functionality is proprietary technology.
Why that is is a topic for another debate, just keep in mind Sony's mobile
division has been shrinking markedly since 2014, by a factor of 5, while Sony's
core business is in its home market of Japan, camera technology and
entertainment.

The way this proprietary technology is "protected" is by utilizing different
forms of nasty mechanisms. The main vector is the "Trim Area(TA)" partition,
a partition about 2MB in size which holds device-specific DRM keys.
Without this partition your device will not boot, making it a hard brick.

By unlocking your device, you will forfeit the DRM keys which are saved on that
TA partition. You can still use Android and even re-flash a stock ROM, but you
will not be able to use the features which require DRM keys.  An overview of
lost functionality is available from Sony:
"[Current platform functionality - Maintained devices](https://developer.sony.com/develop/open-devices/get-started/supported-devices-and-functionality/current-platform-functionality-maintained/)".

Most significant are the following losses(using Xperia XZ as an example):

- TRILUMINOS™ display for mobile, X-Reality® for mobile picture engine, Dynamic Contrast Enhancement
- exFAT support
- Qnovo Adaptive Charging, Battery Care
- High-Resolution Audio (LPCM, FLAC, ALAC, DSD), DSEE HX, LDAC, Digital Noise Cancellation, Clear Audio+
- S-Force Front Surround
- Media codecs: Xvid/MP4/H.265 player
- Camera technology:
  - Drop in resolution
  - Triple image sensing technology, Predictive Hybrid Autofocus, 1/2.3” Exmor
    RS™ for mobile image sensor, 5x Clear Image Zoom, The BIONZ® for mobile
    image-processing engine
  - Front camera: Exmor RS™ for mobile image sensor
  - HDR photo, SteadyShot™ with Intelligent Active Mode (5-axis stablization), Fast Capture
  - Low-light photo: Drop from up to ISO 12800 to ISO 800
  - Video recording from 4K down to 1080p
- GLONASS
- Sensors: Game rotation vector, Geomagnetic rotation vector, Significant motion
  detector, Fingerprint
- STAMINA mode
- Wi-Fi Miracast, DLNA Certified®

On earlier devices, someone found bugs in the Android system to exploit,
which gave one access to the contents of the TA partition with the bootloader
locked.
