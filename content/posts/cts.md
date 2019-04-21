---
title: "Quick: Media CTS"
description: "How to run Media Compatibility Test Suite"
date: 2019-03-01T08:18:00+01:00
draft: false
bref: "How to run Media CTS"
author: Felix
---

<!-- # Setup -->
CTS stands for Compatibility Test suite and is a tool normally distributed to
vendors of Android devices.

The CTS media tests can help benchmark your device to help you adjust the
expected framerates, e.g. for video recording for a given codec.

## Media tests

Download [CTS media files][ctsmedia] and push them to the device with the
provided script.

Need to adjust `CtsMediaTestCases.config` to find your media files under
`path/to/android-cts-media` or else it will complain about missing media files:
```
<configuration description="Config for CTS Media test cases">
    [...]
    <target_preparer class="com.android.compatibility.common.tradefed.targetprep.MediaPreparer">
        <option name="images-only" value="false" />
        <option name="local-media-path" value="/path/to/android-cts-media" />
        <option name="skip-media-download" value="true" />
    </target_preparer>
    [...]
</configuration>
```

To run [media tests](https://source.android.com/devices/media/oem), use this
command, the documentation on the source.android.com page is out of date:

```
run cts \
  --module CtsMediaTestCases \
  --test android.media.cts.MediaCodecCapabilitiesTest#testGetMaxSupportedInstances
```
<!-- run -\-plan CTS android.media.cts.MediaCodecCapabilitiesTest#testGetMaxSupportedInstances -->

Also make sure to save the results of `get_achievable_rates` for later.

## Wrapping up
More general info about running CTS is available at the
[androidsource page][ctsrun].

[ctsmedia]: https://source.android.com/compatibility/cts/downloads#cts-media-files
[ctsrun]: https://source.android.com/compatibility/cts/run
