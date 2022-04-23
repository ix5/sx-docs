---
title: "Kernel Github Action"
slug: "kernel-github-action"
description: ""
date: 2022-03-06T13:36:46+01:00
draft: true
---


```yaml
name: Kernel CI

# Controls when the action will run.
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches:
      # TODO: Get rid of this, only run upon pull requests
      - 71r1-github-actions
  pull_request:
    branches:
      - aosp/LA.UM.7.1.r1

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  # https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idcontainer

  ### Jobs ###
  kernel:
    strategy:
      fail-fast: false # Do not stop build if one matrix strand fails
      matrix:
        # Manual enumeration to get device:platform pairing
        include:
          #- device: pdx201
          #  platform: seine
          #- device: griffin
          #  platform: kumano
          - device: bahamut
            platform: kumano
          #- device: kirin
          #  platform: ganges
          #- device: mermaid
          #  platform: ganges
          #- device: akari
          #  platform: tama
          - device: apollo
            platform: tama
          #- device: akatsuki
          #  platform: tama
          #- device: pioneer
          #  platform: nile
          #- device: discovery
          #  platform: nile
          #- device: voyager
          #  platform: nile
          #- device: maple
          #  platform: yoshino
          #- device: poplar
          #  platform: yoshino
          #- device: lilac
          #  platform: yoshino

    runs-on: ubuntu-20.04
    container:
      image: ghcr.io/ix5/kernel-base:dev
    env:
      GIT_CLONE: "git clone --single-branch --depth=1"
      REMOTE: "https://github.com/sonyxperiadev"
      BASEDIR: "/srv/android"
      KERNEL_VERSION: "4.14"
      KERNEL_BASE_BRANCH: "aosp/LA.UM.7.1.r1"
      KERNEL_BASEDIR: "/srv/android/kernel/sony/msm-4.14"
      ANDROID_BUILD_TOP: "/srv/android"
      COMPILER: "clang"

    ### Steps ###
    steps:

      - name: Get date
        id: get-date
        run: |
          echo "::set-output name=day::$(/bin/date -u "+%Y-%m-%d")"
          echo "::set-output name=month::$(/bin/date -u "+%Y-%m")"
        shell: bash

      - name: Cache access for compressed Kernel source repos
        id: cache_kernel_repos
        uses: actions/cache@v2
        with:
          path: |
            ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar.br
          # Save caches once per day
          key: kernel-sources-tar-br-${{ env.KERNEL_VERSION }}-${{ steps.get-date.outputs.day }}
          restore-keys: |
            kernel-sources-tar-br-${{ env.KERNEL_VERSION }}-${{ steps.get-date.outputs.day }}
            kernel-sources-tar-br-${{ env.KERNEL_VERSION }}-${{ steps.get-date.outputs.month }}

      - name: Cache access for compressed Kernel output dir
        id: cache_kernel_tmp
        uses: actions/cache@v2
        with:
          path: |
            ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar.br
            #${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}
          # Save caches once per day
          key: kernel-tmp-tar-br-${{ env.KERNEL_VERSION }}-${{ matrix.platform }}-${{ steps.get-date.outputs.day }}
          restore-keys: |
            kernel-tmp-tar-br-${{ env.KERNEL_VERSION }}-${{ matrix.platform }}-${{ steps.get-date.outputs.day }}
            kernel-tmp-tar-br-${{ env.KERNEL_VERSION }}-${{ matrix.platform }}-${{ steps.get-date.outputs.month }}

      - name: TEMP install brotli
        run: apt install -y brotli

      - name: Attempt extracting compressed Kernel output dir
        continue-on-error: true
        run: |
          # Abort if no archive found
          [ ! -f $BASEDIR/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar.br ] && exit 0
          cd $BASEDIR
          brotli -d ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar.br
          rm -f ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar.br
          mkdir -p ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}/
          tar -xf  ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar -P -C ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}/
          rm -f  ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar
          ls -la ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}/
        shell: bash

      - name: Attempt extracting compressed Kernel source dir
        continue-on-error: true
        run: |
          # Abort if no archive found
          [ ! -f $BASEDIR/kernel-sources-${{ env.KERNEL_VERSION }}.tar.br ] && exit 0
          cd $BASEDIR
          brotli -d ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar.br
          rm -f ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar.br
          mkdir -p $KERNEL_BASEDIR/
          tar -xf ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar -P -C $KERNEL_BASEDIR
          rm -f ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar
          ls -la $KERNEL_BASEDIR
        shell: bash

      - name: Clone kernel repo
        # Note: cache-hit is set to 'false' if restore-keys is invoked!
        # So this check is useless:
        #if: steps.cache_kernel_repo.outputs.cache-hit != 'true'
        # Instead, check if repo already exists and "fail"
        continue-on-error: true
        env:
          GIT_FETCH: "git fetch --depth=1 origin"
        run: |
          # Abort if already cloned
          [ -d $KERNEL_BASEDIR/kernel/.git ] && exit 0
          mkdir -p $KERNEL_BASEDIR/
          # First, checkout base branch (since refs/* not available for clone directly)
          # > warning: Could not find remote branch refs/heads/71r1-github-actions to clone.
          # > fatal: Remote branch refs/heads/71r1-github-actions not found in upstream origin
          # Note: Using sonyxperiadev as origin since $KERNEL_BASE_BRANCH might not exist on fork
          $GIT_CLONE $REMOTE/kernel $KERNEL_BASEDIR/kernel -b $KERNEL_BASE_BRANCH
          cd $KERNEL_BASEDIR/kernel
          git remote rename origin sony
          # Then add fork as "origin"
          git remote add origin https://github.com/${{ github.repository }}
          $GIT_FETCH ${{ github.ref }}
          git checkout remotes/origin/$(echo ${{ github.ref }} | sed 's|refs/heads/||')
          # Need to remove existing dir
          rm -rf $KERNEL_BASEDIR/kernel/techpack/audio
        shell: bash

      - name: Clone kernel sub-repos
        # Note: cache-hit is set to 'false' if restore-keys is invoked!
        # So this check is useless:
        #if: steps.cache_repos.outputs.cache-hit != 'true'
        # Instead, check if repo already exists and "fail"
        continue-on-error: true
        run: |
          # Abort if already cloned
          [ -d $KERNEL_BASEDIR/kernel/techpack/audio/.git ] && exit 0
          mkdir -p $KERNEL_BASEDIR/
          # Need to remove existing dir
          rm -rf $KERNEL_BASEDIR/kernel/techpack/audio
          $GIT_CLONE $REMOTE/kernel-techpack-audio $KERNEL_BASEDIR/kernel/techpack/audio -b $KERNEL_BASE_BRANCH
          $GIT_CLONE $REMOTE/kernel-techpack-data-kernel $KERNEL_BASEDIR/kernel/techpack/data-kernel -b $KERNEL_BASE_BRANCH
          $GIT_CLONE $REMOTE/kernel-defconfig $KERNEL_BASEDIR/kernel/arch/arm64/configs/sony -b $KERNEL_BASE_BRANCH
          $GIT_CLONE $REMOTE/vendor-qcom-opensource-wlan-fw-api $KERNEL_BASEDIR/kernel/drivers/staging/wlan-qc/fw-api -b aosp/LA.UM.8.1.r1
          $GIT_CLONE $REMOTE/vendor-qcom-opensource-wlan-qca-wifi-host-cmn $KERNEL_BASEDIR/kernel/drivers/staging/wlan-qc/qca-wifi-host-cmn -b aosp/LA.UM.8.1.r1
          $GIT_CLONE $REMOTE/vendor-qcom-opensource-wlan-qcacld-3.0 $KERNEL_BASEDIR/kernel/drivers/staging/wlan-qc/qcacld-3.0 -b aosp/LA.UM.8.1.r1
        shell: bash

      - name: Clone out-of-tree scripts
        run: git clone https://github.com/ix5/oot $BASEDIR/oot -b shared-tmp

      - name: Reset kernel to ref of this run
        # Note: cache-hit is set to 'false' if restore-keys is invoked!
        # So this check is useless:
        #if: steps.cache_kernel_repo.outputs.cache-hit == 'true'
        # Instead, just eat the penalty and always force the fetch
        env:
          GIT_FETCH: "git fetch --depth=1 origin"
          GIT_RESET: "git reset --hard"
        run: |
          cd $KERNEL_BASEDIR/kernel
          # Trigger re-index so that further hard-reset will not touch non-modified files
          git status -s
          $GIT_FETCH ${{ github.ref }}
          $GIT_RESET origin/$(echo ${{ github.ref }} | sed 's|refs/heads/||')
        shell: bash

      - name: Pull latest for repos
        #if: steps.cache_repos.outputs.cache-hit == 'true'
        # Same here: Just always force fetching
        env:
          GIT_FETCH: "git fetch --depth=1 origin"
          GIT_RESET: "git reset --hard"
        run: |
          cd $KERNEL_BASEDIR/kernel/techpack/audio && $GIT_FETCH $KERNEL_BASE_BRANCH && git status -s && $GIT_RESET origin/$KERNEL_BASE_BRANCH
          cd $KERNEL_BASEDIR/kernel/techpack/data-kernel && $GIT_FETCH $KERNEL_BASE_BRANCH && git status -s && $GIT_RESET origin/$KERNEL_BASE_BRANCH
          cd $KERNEL_BASEDIR/kernel/arch/arm64/configs/sony && $GIT_FETCH $KERNEL_BASE_BRANCH && git status -s && $GIT_RESET origin/$KERNEL_BASE_BRANCH
          cd $KERNEL_BASEDIR/kernel/drivers/staging/wlan-qc/fw-api && $GIT_FETCH aosp/LA.UM.8.1.r1 && git status -s && $GIT_RESET origin/aosp/LA.UM.8.1.r1
          cd $KERNEL_BASEDIR/kernel/drivers/staging/wlan-qc/qca-wifi-host-cmn && $GIT_FETCH aosp/LA.UM.8.1.r1 && git status -s && $GIT_RESET origin/aosp/LA.UM.8.1.r1
          cd $KERNEL_BASEDIR/kernel/drivers/staging/wlan-qc/qcacld-3.0 && $GIT_FETCH aosp/LA.UM.8.1.r1 && git status -s && $GIT_RESET origin/aosp/LA.UM.8.1.r1

      - name: Clone ramdisk images
        # sodp-ramdisks contains following structure:
        # | sodp-ramdisks (as "product")
        # |-- apollo
        #     |-- ramdisk.img
        #     | - ramdisk-recovery.img
        run: |
          mkdir -p $BASEDIR/out/target
          rm -rf $BASEDIR/out/target/product
          git clone https://github.com/ix5/sodp-ramdisks -b aosp-10 $BASEDIR/out/target/product

      - name: Run kernel CI script
        run: |
            cd /srv/android/oot/
            /srv/android/oot/oot.sh ${{ matrix.device }}
        #run: |
        #  mkdir -p ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}/
        #  touch ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}/Makefile
        #  ls -la ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}/
        shell: bash

      - name: Archive boot images
        uses: actions/upload-artifact@v2
        with:
          name: ${{ matrix.device }}-boot.img
          path: /srv/android/oot/${{ matrix.device }}-boot.img

      - name: Archive dtbo images
        uses: actions/upload-artifact@v2
        with:
          name: ${{ matrix.device }}-dtbo.img
          path: /srv/android/oot/${{ matrix.device }}-dtbo.img

      - name: Archive gzipped kernel images
        uses: actions/upload-artifact@v2
        with:
          name: ${{ matrix.device }}-Image.gz
          path: /srv/android/out/kernel-4.14/${{ env.COMPILER }}/${{ matrix.platform }}/arch/arm64/boot/Image.gz

      - name: Archive kernel dtb files
        uses: actions/upload-artifact@v2
        with:
          name: ${{ matrix.device }}-dtbs
          # Note: Might change from qcom/ to somc/
          path: /srv/android/out/kernel-4.14/${{ env.COMPILER }}/${{ matrix.platform }}/arch/arm64/boot/dts/qcom/*.dtb

      - name: Archive kernel dtbo files
        uses: actions/upload-artifact@v2
        with:
          name: ${{ matrix.device }}-dtbos
          # Note: Might change from qcom/ to somc/
          path: /srv/android/out/kernel-4.14/${{ env.COMPILER }}/${{ matrix.platform }}/arch/arm64/boot/dts/qcom/*.dtbo

      # Saves a nice deal of space and will get re-generated anyway
      # 336M .tmp_vmlinux1
      # 338M .tmp_vmlinux2
      # 338M vmlinux
      # 907M vmlinux.o
      # 5,5M .tmp_System.map
      # 2,3M .tmp_kallsyms1.o
      # 2,3M .tmp_kallsyms2.o
      # 14M  .tmp_kallsyms1.S
      # 14M  .tmp_kallsyms2.S
      # 35M arch/arm64/boot/Image
      # 13M arch/arm64/boot/Image.gz
      # 15M arch/arm64/boot/Image.gz-dtb
      - name: Cleanup intermediates before saving caches
        continue-on-error: true
        run: |
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/vmlinux
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/vmlinux.o
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/.tmp_vmlinux1
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/.tmp_vmlinux2
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/.tmp_System.map
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/.tmp_kallsyms1.o
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/.tmp_kallsyms2.o
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/.tmp_kallsyms1.S
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/.tmp_kallsyms2.S
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/arch/arm64/boot/Image
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/arch/arm64/boot/Image.gz
          rm -f $BASEDIR/out/kernel-$KERNEL_VERSION/$COMPILER/${{ matrix.platform }}/arch/arm64/boot/Image.gz-dtb

      # Compress using brotli instead of gzip
      # 196M cache.br     <-- brotli
      # 195M cache.br.gz  <-- brotli+gzip penalty
      # 1,3G cache.tar
      # 338M cache.tgz    <-- gzip
      #
      # == Create gzip-compressed tar as run by cache@v2 ===
      # time tar --posix -z -cf cache.tgz -P -C out-kernel-4.14-clang/ .
      # tar --posix -z -cf cache.tgz -P -C out-kernel-4.14-clang/ .  77,23s user 2,66s system 102% cpu 1:17,93 total
      #
      # ### total gzip: 77.2s ###
      #
      # === Create uncompressed tar ===
      # time tar --posix -cf cache.tar -P -C out-kernel-4.14-clang/ .
      # tar --posix -cf cache.tar -P -C out-kernel-4.14-clang/ .  0,22s user 1,88s system 47% cpu 4,446 total
      # === Brotli-compress plain tar ===
      # time brotli cache.tar -k -q 6 -o cache.br -v
      # Compressed [cache.tar]: 1.289 GiB -> 195.187 MiB
      # brotli cache.tar -k -q 6 -o cache.br -v  67,39s user 0,65s system 99% cpu 1:08,59 total
      # === cache@v2 will attempt to re-compress, assuming most costly compression level of 9 ===
      # time gzip --keep -9 cache.br
      # gzip --keep -9 cache.br  8,90s user 0,10s system 99% cpu 9,012 total
      #
      # ### total brotli: 67.4s + 0.2s + 8.9s ~= 76.5s ###
      #
      # --> We can fit almost 1.7x as many archives into cache if
      #     pre-compressing using brotli and still not lose too much time

      - name: Compress Kernel output dir
        # We can save some time by skipping this step if kernel tmp will not be uploaded anyway
        if: steps.cache_kernel_tmp.outputs.cache-hit != 'true'
        run: |
          cd $BASEDIR
          tar --posix -cf ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar -P -C ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }} .
          brotli -k -q 6 ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar -o ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar.br
          # Free up some runner resources so it won't run out of disk space when running post-cache@v2
          rm -f ${{ env.BASEDIR }}/out-kernel-${{ env.KERNEL_VERSION }}-${{ env.COMPILER }}-${{ matrix.platform }}.tar
          rm -rf ${{ env.BASEDIR }}/out/kernel-${{ env.KERNEL_VERSION }}/${{ env.COMPILER }}/${{ matrix.platform }}
        shell: bash

      - name: Compress Kernel source dir
        # We can save some time by skipping this step if kernel sources will not be uploaded anyway
        if: steps.cache_kernel_repos.outputs.cache-hit != 'true'
        run: |
          cd $BASEDIR
          tar --posix -cf ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar -P -C $KERNEL_BASEDIR .
          brotli -k -q 6 ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar -o ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar.br
          # Free up some runner resources so it won't run out of disk space when running post-cache@v2
          rm -f ${{ env.BASEDIR }}/kernel-sources-${{ env.KERNEL_VERSION }}.tar
          rm -rf $KERNEL_BASEDIR
        shell: bash
```
