---
title: "Debugging DAC_OVERRIDE"
description: ""
date: 2019-03-07T12:05:37+01:00
draft: fals
---

<!-- ``` -->
<!-- now the only possible idea I can imagine is  that smp_affinity is turning read only in some circumstances? that even possible? -->
<!-- but in any case, if this is the issue, we should dontaudit and forget about it -->
<!-- actually... -->
<!-- let's set the file read-only and see what happens -->
<!-- [  313.289008] type=1400 audit(1551922324.669:34): avc: denied { dac_override } for pid=1849 comm="msm_irqbalance" capability=1 scontext=u:r:msm_irqbalance:s0 tcontext=u:r:msm_irqbalance:s0 tclass=capability permissive=1 -->
<!-- so this perfectly reproduces what's seen in the normal usecase -->
<!-- but of course this is fabricated to be like this -->
<!-- let's hope that grepping for errors in strace will show some results with the unmodified msm_irqbalance -->
<!-- ``` -->

A very peculiar thing happens with a certain program, let's call it
`irq_assigner`: When in operation, it performs *some* kind of action that
requires the `DAC_OVERRIDE` capability:
```
avc: denied { dac_override } for capability=1 \
  scontext=u:r:irq_assigner:s0 tcontext=u:r:irq_assigner:s0 tclass=capability
```

This is the program in question, `/vendor/etc/init/irq_assigner.rc`:
```
service irq_assigner /vendor/bin/irq_assigner
    socket irq_assigner seqpacket 660 root system
    class core
    user root
    group root
    disabled

on property:sys.boot_completed=1
    enable irq_assigner
```

We checked all files it opens(which are few) and verified the UNIX
permissions(those in `rwxrwxrwx` format) were all correct, i.e. `root` had
sufficient permissions to access them.

Android's SELinux stack has no option to print the file for which `dac_override`
was requested, so we need some trickery.

First, let's try to find what even is causing the issue, the blunt way with a
`WARN_ON` to dump a stack trace:
```diff
diff --git a/security/selinux/avc.c b/security/selinux/avc.c
index 84d9a2e2bbaf..c845db70ed73 100644
--- a/security/selinux/avc.c
+++ b/security/selinux/avc.c
@@ -734,6 +734,10 @@ static void avc_audit_post_callback(struct audit_buffer *ab, void *a)
     if (ad->selinux_audit_data->denied) {
         audit_log_format(ab, " permissive=%u",
                  ad->selinux_audit_data->result ? 0 : 1);
+        if (ad->u.cap == 1) {
+            audit_log_format(ab, " cap=dac_override, dumping stack");
+            WARN_ON(ad->u.cap == 1);
+        }
     }
 }
``` 
Here we go then:
``` 
Call trace:
        : Exception stack(0xffffffc04a68f6b0 to 0xffffffc04a68f7e0)
f6a0    : 0000000000000004 0000007fffffffff

        : [<ffffff80083ab300>] avc_audit_post_callback+0x190/0x198
        : [<ffffff80083cc314>] common_lsm_audit+0xa8/0x710
        : [<ffffff80083ac008>] slow_avc_audit+0xa8/0xd4
        : [<ffffff80083af5d0>] cred_has_capability+0x11c/0x140
        : [<ffffff80083af640>] selinux_capable+0x4c/0x58
        : [<ffffff80083a642c>] security_capable+0x64/0x94
        : [<ffffff80080bdd38>] capable_wrt_inode_uidgid+0x40/0x98
        : [<ffffff8008253020>] generic_permission.part.34+0xac/0xdc
        : [<ffffff80082535a8>] __inode_permission2+0x70/0x104
        : [<ffffff80082536b0>] inode_permission2+0x38/0x74
        : [<ffffff80082567dc>] lookup_open+0x17c/0x588
        : [<ffffff8008257018>] path_openat+0x430/0xaa4
        : [<ffffff8008259744>] do_filp_open+0x70/0x104
        : [<ffffff8008244648>] do_sys_open+0x154/0x268
        : [<ffffff80082447e0>] SyS_openat+0x3c/0x48
        : [<ffffff8008083e00>] el0_svc_naked+0x34/0x38
``` 
The access vector is a dir/file opened with wrong permissions, as expected.

After some probing, a good way to intercept the relavant call is to modify
`generic_permission()` to print if a `DAC_OVERRIDE` was denied:
```diff
diff --git a/fs/namei.c b/fs/namei.c
index d8f858484538..234d7cd1a572 100644
--- a/fs/namei.c
+++ b/fs/namei.c
@@ -39,6 +39,8 @@
 #include <linux/init_task.h>
 #include <asm/uaccess.h>
 
+#include <linux/printk.h>
+
 #include "internal.h"
 #include "mount.h"
 
@@ -341,8 +343,12 @@ int generic_permission(struct inode *inode, int mask)
 
     if (S_ISDIR(inode->i_mode)) {
         /* DACs are overridable for directories */
-        if (capable_wrt_inode_uidgid(inode, CAP_DAC_OVERRIDE))
+        if (capable_wrt_inode_uidgid(inode, CAP_DAC_OVERRIDE)) {
             return 0;
+        } else {
+            pr_err("dac_override(dir) denied for cap=%3o inode=%lu with uid=%u gid=%u",
+                   mask&0777, inode->i_ino, inode->i_uid, inode->i_gid);
+        }
         if (!(mask & MAY_WRITE))
             if (capable_wrt_inode_uidgid(inode,
                              CAP_DAC_READ_SEARCH))
@@ -354,9 +360,21 @@ int generic_permission(struct inode *inode, int mask)
      * Executable DACs are overridable when there is
      * at least one exec bit set.
      */
-    if (!(mask & MAY_EXEC) || (inode->i_mode & S_IXUGO))
-        if (capable_wrt_inode_uidgid(inode, CAP_DAC_OVERRIDE))
+    /* IXUGO = (S_IXUSR|S_IXGRP|S_IXOTH) */
+    if (!(mask & MAY_EXEC) || (inode->i_mode & S_IXUGO)) {
+        if (capable_wrt_inode_uidgid(inode, CAP_DAC_OVERRIDE)) {
             return 0;
+        } else {
+            /* struct inode { */
+            /*     i_uid is of uid32_t = unsigned int */
+            /*     i_gid same */
+            /*     i_ino is of unsigned long */
+            /* } */
+            /* pr_err("dac_override denied for cap=%o inode=%lu with uid=%d gid=%d", */
+            pr_err("dac_override(file) denied for cap=%3o inode=%lu with uid=%u gid=%u",
+                   mask&0777, inode->i_ino, inode->i_uid, inode->i_gid);
+        }
+    }
     /*
      * Searching includes executable on directories, else just read.
```
 

With the logging and `WARN_ON` in place, time for a test run:
``` 
 ------------[ cut here ]------------
WARNING: CPU: 3 PID: 1681 at avc_audit_post_callback+0x190/0x198
Modules linked in:
CPU: 3 PID: 1681 Comm: irq_assigner Tainted: G        W       4.9.160-ge2af85baed78-dirty #6
name: SoMC Kagura-ROW (DT)
PC is at avc_audit_post_callback+0x190/0x198
LR is at avc_audit_post_callback+0x184/0x198
[...]
---[ end trace 0db9c115d7e8753c ]---
        : dac_override(dir) denied for cap=  3 inode=4026533027 with uid=0 gid=0
        : dac_override(dir) denied for cap=  3 inode=4026531862 with uid=0 gid=0
        : dac_override(dir) denied for cap=  3 inode=1146882 with uid=0 gid=0
irq_assigner: type=1400 audit(0.0:11): avc: denied { dac_override } for \
  capability=1 scontext=u:r:irq_assigner:s0 tcontext=u:r:irq_assigner:s0 \
  tclass=capability permissive=0 cap=dac_override, dumping stack
[...]
---[ end trace 0db9c115d7e87539 ]---
Call trace:
Exception stack(0xffffffc04a68f6b0 to 0xffffffc04a68f7e0)
[...]
dac_override(dir) denied for cap=  3 inode=4026532260 with uid=0 gid=0
------------[ cut here ]------------
WARNING: CPU: 2 PID: 1681 at avc_audit_post_callback+0x190/0x198
CPU: 2 PID: 1681 Comm: irq_assigner Tainted: G W 4.9.160-ge2af85baed78-dirty #6
PC is at avc_audit_post_callback+0x190/0x198
LR is at avc_audit_post_callback+0x184/0x198
[...]
Call trace:
[...]
dac_override(dir) denied for cap=  3 inode=4026532239 with uid=0 gid=0
```

The high `inode` numbers in the `4.000.000.000` range correspond to *directories* in
`/proc/irq`:
``` 
kagura:/ # find /proc -inum 4026533027 2&>/dev/null
/proc/irq/531
kagura:/ # find /proc -inum 4026532260 2&>/dev/null
/proc/irq/82
``` 

Permissions on `/proc/irq` and its subdirs are `dr-xr-xr-x`, as created by
[proc_create_data()][proc-cd] (parent dir with
[S_IRUGO=(S_IRUSR|S_IRGRP|S_IROTH)][proc-parent]).

Interesting, so we now the handling of dirs/files in `/proc/irq/` is the issue.
Let's `strace` the relevant parts:
```
[...]
openat(AT_FDCWD, "/proc/irq/74/smp_affinity", O_RDONLY) = 8
fstat(8, {st_mode=S_IFREG|0644, st_size=0, ...}) = 0
read(8, "1\n", 1024)                    = 2
close(8)                                = 0
```
And later:
```
openat(AT_FDCWD, "/proc/irq/81/smp_affinity", O_WRONLY|O_CREAT|O_TRUNC, 0666) = 8
fstat(8, {st_mode=S_IFREG|0644, st_size=0, ...}) = 0
write(8, "2", 1)                        = 1
close(8)                                = 0
```

Compare that to the behaviour of a different version that does not have the
issue:
```
openat(AT_FDCWD, "/proc/irq/105/smp_affinity", O_RDWR) = 6
fstat(6, {st_mode=S_IFREG|0644, st_size=0, ...}) = 0
write(6, "1", 1)                        = 1
close(6)                                = 0
```
Quite different...

The fix should be easy, change the `fopen()` mode from
`O_WRONLY|O_CREAT|O_TRUNC` to `O_RDWR`(=`0664`).

[proc-cd]: https://github.com/sonyxperiadev/kernel/blob/aosp/LE.UM.2.3.2.r1.4/kernel/irq/proc.c#L356
[proc-parent]: https://github.com/sonyxperiadev/kernel/blob/aosp/LE.UM.2.3.2.r1.4/fs/proc/generic.c#L501
