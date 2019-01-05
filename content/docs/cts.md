+++
title = "CTS"
description = "How to run CTS"
date = 2018-12-14T22:35:16+01:00
weight = 20
draft = true
bref = ""
toc = true
+++

# Setup

# Media tests

Download media files and push them to the device with the script

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

To run media tests:
https://source.android.com/devices/media/oem
Also save get_achievable_rates results

https://source.android.com/compatibility/cts/run

```
# run --plan CTS android.media.cts.MediaCodecCapabilitiesTest#testGetMaxSupportedInstances
run cts --module CtsMediaTestCases --test android.media.cts.MediaCodecCapabilitiesTest#testGetMaxSupportedInstances
```
