---
title: "Android Q: Compiling Kernels in-tree"
description: ""
date: 2019-08-17T22:52:49+02:00
draft: false
slug: android-q-compiling-kernels-in-tree
---

## TL;DR - Kernel fixes
For an example of how to update your kernel makefile for Q, see
[Sony Kernel: Update makefiles for Android Q and clang][kernel]


## Q Build system changes
The main changes in Android Q are:

- New `$PATH` restrictions
- GCC 4.9 deprecation
- Removal of ccache
- `PHONY` target enforcement

For more information, see
[AOSP: Build System Changes for Android.mk Writers][changesmd].

#### PHONY targets
For more information, see [.PHONY rule enforcement][phony].

Real output files cannot be marked `PHONY` any more, the build system will
complain if `PHONY` targets have slashes in them.
```sh
warning: PHONY target "out/.../foo" looks like a real file (contains a "/")
```

Just remove the attribute if
an output file isn't really `PHONY`.
```diff
-.PHONY: $(PRODUCT_OUT)/kernel
-$(PRODUCT_OUT)/kernel: $(KERNEL_BIN) | $(ACP)
-        cp $(KERNEL_BIN) $(PRODUCT_OUT)/kernel
+$(PRODUCT_OUT)/kernel: $(KERNEL_BIN)
+    $(ACP) $(KERNEL_BIN) $(PRODUCT_OUT)/kernel
```

Also, you cannot depend on intermediates being `PHONY` any more if your final
target is non-`PHONY`, but that's standard `make` for you.

#### Output directory restrictions
These will come to bite you if you rely on build
targets with absolute filenames.

**Example:**
```make
KERNEL_OUT: $(PWD)/$(OUT)/obj/KERNEL_OBJ
$(KERNEL_OUT)/drivers:
	make -C $(KERNEL_SRC) -O $(KERNEL_OUT) drivers
```
Throws
```sh
warning: writing to readonly directory: "/home/[...]/out/[...]/KERNEL_OBJ/drivers"
```
That is because the make target `$(KERNEL_OUT)/drivers` does not begin with
`out/target/...` but rather is an absolute path; the build system is not smart
enough to recognize that the path _should_ still be allowed.

Use a relative-path `KERNEL_OUT` for a make target and an
absolute-path`KERNEL_OUT_ABS` for invoking the "real" `make` command:
```make
KERNEL_OUT: $(OUT)/obj/KERNEL_OBJ
KERNEL_OUT_ABS: $(PWD)/$(OUT)/obj/KERNEL_OBJ
$(KERNEL_OUT)/drivers:
	make -C $(KERNEL_SRC) -O $(KERNEL_OUT_ABS) drivers
```

#### GCC is broken for host compilation
Even with correct `PATH` supplied for `cc1`, the prebuilt GCC 4.9 still fails at
compiling the host scripts:
```sh
  HOSTCC  scripts/basic/fixdep
  [...]/kernel/scripts/basic/fixdep.c:105:23:
    fatal error: sys/types.h: No such file or directory
     #include <sys/types.h>
```
It's better to use clang for `HOSTCC`:
```
# Android.mk
CLANG_HOST_TOOLCHAIN := $(PWD)/prebuilts/clang/host/linux-x86/clang-r365631/bin
CLANG_HOSTCC := $(CLANG_HOST_TOOLCHAIN)/clang
[...]
KERNEL_CROSS_COMPILE += HOSTCC="$(CLANG_HOSTCC)"
[...]
my-target:
	make $(MAKE_FLAGS) -C $(KERNEL_SRC_ABS) O=$(KERNEL_OUT_ABS) \
		ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE)
```
`Makefile`:
```diff
-HOSTCC       = gcc
-HOSTCXX      = g++
+HOSTCC       = $(HOSTCC)
+HOSTCXX      = $(HOSTCXX)
```

#### Absolute paths for invoking build tools
```make
CLANG_HOST_TOOLCHAIN := $(PWD)/prebuilts/clang/host/linux-x86/clang-r365631/bin
KERNEL_HOSTCC := $(CLANG_HOST_TOOLCHAIN)/clang
KERNEL_TOOLCHAIN := $(PWD)/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin
KERNEL_TOOLCHAIN_PREFIX := aarch64-linux-android-
KERNEL_TOOLCHAIN_PATH := $(KERNEL_TOOLCHAIN)/$(KERNEL_TOOLCHAIN_PREFIX)
KERNEL_CROSS_COMPILE :=
KERNEL_CROSS_COMPILE += CC="$(CLANG_CC)"
KERNEL_CROSS_COMPILE += CLANG_TRIPLE="aarch64-linux-gnu"
KERNEL_CROSS_COMPILE += HOSTCC="$(KERNEL_HOSTCC)"
KERNEL_CROSS_COMPILE += CROSS_COMPILE="$(KERNEL_TOOLCHAIN_PATH)"
KERNEL_CROSS_COMPILE += CROSS_COMPILE_ARM32="$(KERNEL_TOOLCHAIN_32BITS_PATH)"
```

#### No relying on tools being in PATH
You cannot rely on `make`, `perl`, `gcc` etc. being in `$PATH`.  
For more information, see [Android Q Changes: PATH restrictions][q-changes-path].

Perl is no longer allowed as a `PATH` tool:
```make
# Android.mk
KERNEL_PERL := /usr/bin/perl
[...]
KERNEL_CROSS_COMPILE += PERL=$(KERNEL_PERL)
```
`Makefile`:
```diff
-PERL= perl
+PERL= $(PERL)
```
`lib/Makefile`:
```
-      cmd_build_OID_registry = perl $(srctree)/$(src)/build_OID_registry $< $@
+      cmd_build_OID_registry = $(PERL) $(srctree)/$(src)/build_OID_registry $< $@
```

`make` is no longer an allowed `PATH` tool, and you need to supply an extra
`PATH` for host binutils:
```make
GCC_PREBUILTS := $(PWD)/prebuilts/gcc/linux-x86/host
KERNEL_HOST_TOOLCHAIN := \
    $(GCC_PREBUILTS)/x86_64-linux-glibc2.17-4.8/x86_64-linux/bin
KERNEL_HOST_TOOLCHAIN_LIBEXEC := \
    $(GCC_PREBUILTS)/libexec/gcc/x86_64-linux/4.8.3
KERNEL_PREBUILT_MAKE := $(PWD)/prebuilts/build-tools/linux-x86/bin/make
# clang/GCC (glibc) host toolchain needs to be prepended to $PATH for certain
# host bootstrap tools to be built. Also, binutils such as `ld` and `ar` are
# needed for now.
KERNEL_MAKE_EXTRA_PATH := $(KERNEL_HOST_TOOLCHAIN)
ifneq ($(TARGET_KERNEL_USE_CLANG),true)
  KERNEL_MAKE_EXTRA_PATH := \
      "$(KERNEL_HOST_TOOLCHAIN):$(KERNEL_HOST_TOOLCHAIN_LIBEXEC)"
endif
KERNEL_MAKE := \
	PATH="$(KERNEL_MAKE_EXTRA_PATH):$$PATH" \
	$(KERNEL_PREBUILT_MAKE)
```

<!--
```
#export PATH=$PATH:/home/builder/omni/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.17-4.8/libexec/gcc/x86_64-linux/4.8.3/

export COLLECT_GCC_OPTIONS="'-E' '-v' '-mtune=generic' '-march=x86-64' cc1 -E -quiet -v -imultiarch x86_64-linux-gnu -iprefix /home/builder/omni/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.17-4.8/lib/gcc/x86_64-linux/4.8.3 - -mtune=generic -march=x86-64"

export PATH=$(pwd)/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.17-4.8/x86_64-linux/bin:$PATH
export PATH=$(pwd)/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.17-4.8/libexec/gcc/x86_64-linux/4.8.3:$PATH

#export PATH=$PATH:/home/builder/omni/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/libexec/gcc/aarch64-linux-android/4.9.x/

#prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/libexec/gcc/arm-linux-androideabi/4.9.x/cc1
#prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/libexec/gcc/aarch64-linux-android/4.9.x/cc1
#prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.17-4.8/libexec/gcc/x86_64-linux/4.8.3/cc1
```

```
builder@ubuntu-android:~/omni$ get_build_var MAKE
prebuilts/build-tools/linux-x86/bin/ckati --color_warnings --kati_stats MAKECMDGOALS=
```

```
kagura:/ # cat /proc/version
Linux version 4.9.185-667870-g42430b55f92b-dirty (nobody@android-build) (Android (5799447 based on r365631) clang version 9.0.6 (https://android.googlesource.com/toolchain/llvm-project 85305eaf1e90ff529d304abac8a979e1d967f0a2) (based on LLVM 9.0.6svn)) #2 SMP PREEMPT Mon Aug 19 01:38:11 CEST 2019
```

```
echo $PATH
/home/builder/omni/prebuilts/jdk/jdk9/linux-x86/bin:/home/builder/omni/out/soong/host/linux-x86/bin:/home/builder/omni/out/host/linux-x86/bin:/home/builder/omni/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin:/home/builder/omni/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin:/home/builder/omni/development/scripts:/home/builder/omni/prebuilts/devtools/tools:/home/builder/omni/external/selinux/prebuilts/bin:/home/builder/omni/prebuilts/misc/linux-x86/dtc:/home/builder/omni/prebuilts/misc/linux-x86/libufdt:/home/builder/omni/prebuilts/clang/host/linux-x86/llvm-binutils-stable:/home/builder/omni/prebuilts/asuite/acloud/linux-x86:/home/builder/omni/prebuilts/asuite/aidegen/linux-x86:/home/builder/omni/prebuilts/asuite/atest/linux-x86:/home/builder/omni/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.17-4.8/libexec/gcc/x86_64-linux/4.8.3:/home/builder/omni/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.17-4.8/x86_64-linux/bin:/home/builder/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
-->

[kernel]: https://github.com/ix5/kernel-sony/commit/6302f87e5ce1d9198707c42db9c502e0fe62f74b
[changesmd]: https://android.googlesource.com/platform/build/+/refs/tags/android-q-preview-6/Changes.md
[phony]: https://android.googlesource.com/platform/build/+/refs/tags/android-q-preview-6/Changes.md#phony_targets
[q-changes-path]: ../android-q-changes/#path-restrictions
