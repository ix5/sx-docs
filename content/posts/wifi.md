---
title: "WiFi notes"
date: 2019-01-01T08:18:00+01:00
draft: true
author: Felix
---

# Tethering 5Ghz

`getAvailable5gNonDFSChannels`

DFS = Dynamic Frequency selction

# Other
STA = Station mode = Client
AP = Access point = Router

wifi_concurrency_cfg.txt
```
ENABLE_STA_SAP_CONCURRENCY:1
SAP_INTERFACE_NAME:softap0
SAP_CHANNEL:6
```

# CHRE
> This daemon loads the Context Hub Runtime Environment (CHRE) dynamic modules
> onto the SLPI using FastRPC, and exposes a sockets interface for clients on
> the applications processor to interact CHRE

# More tethering
frameworks/opt/net/wifi/service/java/com/android/server/wifi/WificondControl.java
frameworks/opt/net/wifi/service/java/com/android/server/wifi/SoftApManager.java
frameworks/opt/net/wifi/service/java/com/android/server/wifi/util/ApConfigUtil.java


# Needed patches

external/libnl/lib/msg.c
```
-static void print_hdr(FILE *ofd, struct nl_msg *msg)
+// Needs to be non-static to be used elsewhere
+void print_hdr(FILE *ofd, struct nl_msg *msg)
````

external/wpa_supplicant_8
-> lots of different ones, see patch

frameworks/base
```
diff --git a/services/art-profile b/services/art-profile
index a372a9c2f2a..8f46d8d0367 100644
--- a/services/art-profile
+++ b/services/art-profile
@@ -702,6 +702,7 @@ HSPLandroid/net/wifi/IWificond;->disableSupplicant()Z
 HSPLandroid/net/wifi/IWificond;->enableSupplicant()Z
 HSPLandroid/net/wifi/IWificond;->getAvailable2gChannels()[I
 HSPLandroid/net/wifi/IWificond;->getAvailable5gNonDFSChannels()[I
+HSPLandroid/net/wifi/IWificond;->getAvailable5gWithDFSChannels()[I
 HSPLandroid/net/wifi/IWificond;->getAvailableDFSChannels()[I
 HSPLandroid/net/wifi/IWificond;->tearDownApInterface(Ljava/lang/String;)Z
 HSPLandroid/net/wifi/IWificond;->tearDownClientInterface(Ljava/lang/String;)Z
```

kernel: drivers/staging/wlan-qc/qca-wifi-host-cmn
```
diff --git a/wmi/src/wmi_unified_non_tlv.c b/wmi/src/wmi_unified_non_tlv.c
index a55fea25..35d3334b 100644
--- a/wmi/src/wmi_unified_non_tlv.c
+++ b/wmi/src/wmi_unified_non_tlv.c
@@ -2682,6 +2682,14 @@ send_pdev_set_regdomain_cmd_non_tlv(wmi_unified_t wmi_handle,
        wmi_buf_t buf;
 
        int len = sizeof(wmi_pdev_set_regdomain_cmd);
+       /* TODO */
+       qdf_print("%s: sending regdomain: domain=%d, 2g=%d, 5g=%d, dfs_domain=%s\n",
+                               __func__,
+                               param->currentRDinuse,
+                               param->currentRD2G,
+                               param->currentRD5G,
+                               param->dfsDomain
+                               );
 
        buf = wmi_buf_alloc(wmi_handle, len);
        if (!buf) {
```


device/sony/tone
```
diff --git a/PlatformConfig.mk b/PlatformConfig.mk
index 24c0bfb..0148bf6 100644
--- a/PlatformConfig.mk
+++ b/PlatformConfig.mk
@@ -44,12 +44,13 @@ BOARD_KERNEL_CMDLINE += androidboot.bootdevice=7464900.sdhci
 TARGET_RECOVERY_FSTAB ?= $(PLATFORM_COMMON_PATH)/rootdir/vendor/etc/fstab.tone
 
 # Wi-Fi definitions for Broadcom solution
-BOARD_WLAN_DEVICE           := bcmdhd
+BOARD_WLAN_DEVICE           := qcwcn
+BOARD_HAS_QCOM_WLAN := true
 BOARD_WPA_SUPPLICANT_DRIVER := NL80211
 WPA_SUPPLICANT_VERSION      := VER_0_8_X
-BOARD_WPA_SUPPLICANT_PRIVATE_LIB := lib_driver_cmd_bcmdhd
+BOARD_WPA_SUPPLICANT_PRIVATE_LIB := lib_driver_cmd_qcwcn
 BOARD_HOSTAPD_DRIVER        := NL80211
-BOARD_HOSTAPD_PRIVATE_LIB   := lib_driver_cmd_bcmdhd
+BOARD_HOSTAPD_PRIVATE_LIB   := lib_driver_cmd_qcwcn
 # define firmware paths if not using brcmfmac driver
 ifneq ($(WIFI_DRIVER_BUILT),brcmfmac)
 WIFI_DRIVER_FW_PATH_PARAM   := "/sys/module/bcmdhd/parameters/firmware_path"
diff --git a/rootdir/vendor/etc/wifi/p2p_supplicant_overlay.conf b/rootdir/vendor/etc/wifi/p2p_supplicant_overlay.conf
index e44c477..f58d42b 100644
--- a/rootdir/vendor/etc/wifi/p2p_supplicant_overlay.conf
+++ b/rootdir/vendor/etc/wifi/p2p_supplicant_overlay.conf
@@ -2,4 +2,6 @@ disable_scan_offload=1
 wowlan_triggers=any
 p2p_no_go_freq=5170-5740
 p2p_search_delay=0
-no_ctrl_interface=
\ No newline at end of file
+no_ctrl_interface=
+p2p_no_group_iface=1
+p2p_go_vht=1
diff --git a/rootdir/vendor/etc/wifi/wpa_supplicant_overlay.conf b/rootdir/vendor/etc/wifi/wpa_supplicant_overlay.conf
index 740792b..5099e4e 100644
--- a/rootdir/vendor/etc/wifi/wpa_supplicant_overlay.conf
+++ b/rootdir/vendor/etc/wifi/wpa_supplicant_overlay.conf
@@ -2,4 +2,13 @@ disable_scan_offload=1
 wowlan_triggers=any
 p2p_disabled=1
 filter_rssi=-75
-no_ctrl_interface=
\ No newline at end of file
+no_ctrl_interface=
+
+eapol_version=2
+ap_scan=1
+country=DE
+tdls_external_control=1
+bss_max_count=512
+
+# Seems not a good idea, with system+vendor being read-only
+#update_config=1
```

# BRCMFMAC
device/sony/kagura
```
diff --git a/device.mk b/device.mk
index eff2375..1c908d2 100644
--- a/device.mk
+++ b/device.mk
@@ -18,6 +18,9 @@ DEVICE_PATH := device/sony/kagura/rootdir
 DEVICE_PACKAGE_OVERLAYS += \
     device/sony/kagura/overlay
 
+# declare brcmfmac as the wifi driver
+#WIFI_DRIVER_BUILT := brcmfmac
+
 # Device Specific Permissions
 PRODUCT_COPY_FILES := \
     frameworks/native/data/etc/handheld_core_hardware.xml:$(TARGET_COPY_OUT_VENDOR)/etc/permissions/handheld_core_hardware.xml \
@@ -70,6 +73,10 @@ PRODUCT_COPY_FILES += \
 # WIFI FW patch
 PRODUCT_COPY_FILES += \
     $(DEVICE_PATH)/vendor/firmware/bcmdhd.cal:$(TARGET_COPY_OUT_VENDOR)/firmware/bcmdhd.cal
+ifeq ($(WIFI_DRIVER_BUILT),brcmfmac)
+PRODUCT_COPY_FILES += \
+    $(DEVICE_PATH)/vendor/firmware/bcmdhd.cal:$(TARGET_COPY_OUT_VENDOR)/firmware/brcm/brcmfmac43455-sdio.txt
+endif
 
 # Device Init
 PRODUCT_PACKAGES += \
 ```

# Misc

`system/connectivity/wificond/net/netlink_utils.h`
```
struct BandInfo {
  BandInfo() = default;
  BandInfo(std::vector<uint32_t>& band_2g_,
           std::vector<uint32_t>& band_5g_,
           std::vector<uint32_t>& band_dfs_)
      : band_2g(band_2g_),
        band_5g(band_5g_),
        band_dfs(band_dfs_) {}
  // Frequencies for 2.4 GHz band.
  std::vector<uint32_t> band_2g;
  // Frequencies for 5 GHz band without DFS.
  std::vector<uint32_t> band_5g;
  // Frequencies for DFS.
  std::vector<uint32_t> band_dfs;
};
```

frameworks/base/wifi/java/android/net/wifi/WifiScanner.java
```
public class WifiScanner {
    /** no band specified; use channel list instead */
    public static final int WIFI_BAND_UNSPECIFIED = 0;      /* not specified */

    /** 2.4 GHz band */
    public static final int WIFI_BAND_24_GHZ = 1;           /* 2.4 GHz band */
    /** 5 GHz band excluding DFS channels */
    public static final int WIFI_BAND_5_GHZ = 2;            /* 5 GHz band without DFS channels */
    /** DFS channels from 5 GHz band only */
    public static final int WIFI_BAND_5_GHZ_DFS_ONLY  = 4;  /* 5 GHz band with DFS channels */
    /** 5 GHz band including DFS channels */
    public static final int WIFI_BAND_5_GHZ_WITH_DFS  = 6;  /* 5 GHz band with DFS channels */
    /** Both 2.4 GHz band and 5 GHz band; no DFS channels */
    public static final int WIFI_BAND_BOTH = 3;             /* both bands without DFS channels */
    /** Both 2.4 GHz band and 5 GHz band; with DFS channels */
    public static final int WIFI_BAND_BOTH_WITH_DFS = 7;    /* both bands with DFS channels */
```


external/iw/util.c
```
static const char *commands[NL80211_CMD_MAX + 1] = {
/*
 * sed 's/^\tNL80211_CMD_//;t n;d;:n s%^\([^=]*\),.*%\t[NL80211_CMD_\1] = \"\L\1\",%;t;d' nl80211.h
 */
	[NL80211_CMD_UNSPEC] = "unspec",
	[NL80211_CMD_GET_WIPHY] = "get_wiphy",
    [...]
```

linux/nl80211.h
```
/**
 * enum nl80211_dfs_regions - regulatory DFS regions
 *
 * @NL80211_DFS_UNSET: Country has no DFS master region specified
 * @NL80211_DFS_FCC: Country follows DFS master rules from FCC
 * @NL80211_DFS_ETSI: Country follows DFS master rules from ETSI
 * -> a.k.a Europe
 * @NL80211_DFS_JP: Country follows DFS master rules from JP/MKK/Telec
 */
enum nl80211_dfs_regions {
	NL80211_DFS_UNSET	= 0,
	NL80211_DFS_FCC		= 1,
	NL80211_DFS_ETSI	= 2,
	NL80211_DFS_JP		= 3,
};
```

kernel netlink.h
```
# Netlink Messages and Attributes Interface
#
# Message Format:
#    <--- nlmsg_total_size(payload)  --->
#    <-- nlmsg_msg_size(payload) ->
#   +----------+- - -+-------------+- - -+-------- - -
#   | nlmsghdr | Pad |   Payload   | Pad | nlmsghdr
#   +----------+- - -+-------------+- - -+-------- - -
#   nlmsg_data(nlh)---^                   ^
#   nlmsg_next(nlh)-----------------------+
#
# Payload Format:
#    <---------------------- nlmsg_len(nlh) --------------------->
#    <------ hdrlen ------>       <- nlmsg_attrlen(nlh, hdrlen) ->
#   +----------------------+- - -+--------------------------------+
#   |     Family Header    | Pad |           Attributes           |
#   +----------------------+- - -+--------------------------------+
#   nlmsg_attrdata(nlh, hdrlen)---^

static inline void *nla_data(const struct nlattr *nla) {
	return (char *) nla + NLA_HDRLEN; }
```
