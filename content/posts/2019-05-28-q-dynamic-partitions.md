---
title: "Android Q: Dynamic Partitions"
description: ""
date: 2019-05-29T16:15:09+02:00
slug: "android-q-dynamic-partitions"
draft: false
---

Talk by Google's David Anderson at Linux Plumber's Conf, see
[LPC Schedule][schedule] for more, [video at YouTube][youtube].  
PDF: [Dynamic Partitions][pdf].

---

**Update:** See [Dynamic Partitions: Take Two](/info/post/dynamic-partitions-take-two/).

---

Buildvars:
```
PRODUCT_USE_DYNAMIC_PARTITIONS
PRODUCT_USE_DYNAMIC_PARTITION_SIZE
PRODUCT_RETROFIT_DYNAMIC_PARTITIONS
PRODUCT_BUILD_SUPER_PARTITION
```

Need to unset `BOARD_BUILD_SYSTEM_ROOT_IMAGE`, `TARGET_NO_RECOVERY` is
unaffected.

[AOSP build/make: Unset system-as-root for mainline][unset]:
```diff
-BOARD_BUILD_SYSTEM_ROOT_IMAGE := true
```

> This flag is not allowed to be true when dynamic partitions are
> enabled, which mainline devices are expected to do.

---

Of course, this stuff is hopelessly broken for legacy devices. An attempt to get
it to work:

`build/make`:
```diff
diff --git a/tools/releasetools/build_super_image.py b/tools/releasetools/build_super_image.py
index f63453d8e..b13b906d0 100755
--- a/tools/releasetools/build_super_image.py
+++ b/tools/releasetools/build_super_image.py
@@ -182,6 +182,9 @@ def BuildSuperImage(inp, out):
 
   if isinstance(inp, str):
     if os.path.isdir(inp):
+      if os.path.isdir(out):
+        logger.info("out is a dir: %s, setting to super_empty" % str(out))
+        out = "%s/super_empty.img" % out
       logger.info("Building super image from extracted target files...")
       return BuildSuperImageFromExtractedTargetFiles(inp, out)
```

`system/core`:
```diff
index a0ccc10f7..6b2436e59 100644
--- a/fs_mgr/libfiemap_writer/split_fiemap_writer.cpp
+++ b/fs_mgr/libfiemap_writer/split_fiemap_writer.cpp
@@ -95,7 +95,22 @@ std::unique_ptr<SplitFiemap> SplitFiemap::Create(const std::string& file_path, u
         // To make sure the alignment doesn't create too much inconsistency, we
         // account the *actual* size, not the requested size.
         total_bytes_written += writer->size();
-        remaining_bytes -= writer->size();
+        LOG(WARNING) << "SplitFiemap:Create() remaining_bytes = "
+                     << remaining_bytes<< ", writer->size() = "
+                     << writer -> size();
+        // 09-29 23:54:24.512  2523  2525 W gsid: SplitFiemap:Create()
+        //      remaining_bytes = 1589934592, writer->size() = 1589936128
+        // 1589934592 − 1589936128 = −1536
+        if (remaining_bytes < writer->size()) {
+            LOG(WARNING) << "SplitFiemap:Create() remaining_bytes"
+                         << "< writer->size, setting remaining_bytes to 0";
+            remaining_bytes = 0;
+        } else {
+            remaining_bytes -= writer->size();
+            LOG(WARNING) << "SplitFiemap:Create()"
+                         << " remaining_bytes, minus writer->size = "
+                         << remaining_bytes;
+        }
 
         out->AddFile(std::move(writer));
     }
```

[schedule]: https://www.linuxplumbersconf.org/event/2/timetable/?view=nicecompact
[youtube]: https://www.youtube.com/watch?v=xMtDDEj-02c&feature=youtu.be&list=PLVsQ_xZBEyN2tq96cAph0Dcy6gJVq_Wqw&t=4685s
[unset]: https://android-review.googlesource.com/c/platform/build/+/932362
[pdf]: https://linuxplumbersconf.org/event/2/contributions/225/attachments/49/56/06._Dynamic_Partitions_-_LPC_Android_MC_v2.pdf
