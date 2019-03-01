+++
title = "SELinux"
description = "How to configure"
date = 2018-12-08T22:15:22+02:00
weight = 15
draft = false
bref = "How to configure"
toc = true
+++

*Sorry this is such a mess, this is more of a braindump...*

---

## General
(from [SEAndroid-Aalto presentation][seandroid-aalto]):

> Subjects are primarily processes. Processes that participate in the access
> control are assigned a domain

> Objects are e.g. files, sockets.. Objects that participate in the access
> control are assigned a type

```
 X          N        allow   M                 Y
subjects   domains   rules  types          objects
                                 --------------[]
                                /      --------[]
[]---\        /-------------[]---- ---/--------[]
[]    ------[]------/-------[]--- ---/---------[]
[]----------[]\----/--------[]-\----/----------[]
[]      /  -[]-\------------[]--\--------------[]
[]-----/  /   \ \              \/\             []
[]       /     \------------[]--\-\------------[]
[]------/-                       --\-----------[]
                                    -----------[]
```

> An object is always an OS primitive of some sort. This is not a policy issue,
> this is reality. An object therefore belongs to one or more classes which are
> predefined by the system proper. A class can be e.g. file, socket, character
> device, ....

```
Objects   Classes      Permissions
[]--           /-----------[]
[]--\-------[]-------------[]
[]-/          \------------[]
[]-
```

> A class is associated with a fixed set of permissions (e.g read, write,
> open..), often closely mapped to system call functionality. Again, the set of
> available permission is predefined, policy does not change that But! An allow
> rule includes also the class, and within that the permissions allowed by the
> rule. Class and permission names are ”well known” as defined by NSA / SELinux

In general: `allow [domain] [type] : [class] {[ allowed permissions ]}`

- `[domain]`: Subject(a process in a domain)
- `[type]`: Object(a file or other resource)
- `[class]`: Resource class(from predefined set)
- `[allowed_permissions]`: Class-specific permissions that are allowed by this
  rule

## Testing

<!-- TODO: Expand testing tools info, e.g. sesearch, matchpathcon(and its gotchas for -->
<!-- selecting policy path!!!!) -->
Install needed tools for the host machine:
```
apt install setools python-networkx policycoreutils
```

**Testing it out:** either `mka selinux_policy` or `make bootimage`
But to only check if the policy is valid: `make -j $(nproc) sepolicy_tests`!


# Misc

Directories:

- `system/sepolicy`
- `device/sony/sepolicy`
- `device/sony/<platform>/sepolicy_platform`
- `system/bt/vendor_libs/linux/sepolicy`
- `system/extras/boottime_tools/bootio/sepolicy`
- ...

- `file_contexts`("real" files, but not necessarily limited to that),
  `genfs_contexts`("generated" files) -> contexts need to exist!
- `property_contexts` -> always need a corresponding type in `property.te`!

<!--
TODO:
- Sepolicy macros
- Interaction with closed-source "secure" stuff, `rild`
- Apps, services, files, binaries
- How to read denials
- Where to look for reference (marlin, crosshatch, crosshatch-sepolicy),
  permalinks to indiviual lines of policy
- Things that can be left as-is (via `dontaudit`)
-->


```
PRODUCT_FULL_TREBLE_OVERRIDE := false
BOARD_USE_ENFORCING_SELINUX := true
```

<!-- Ideally, we would like the Qualcomm blobs to function as vndbinder/vndservice -->
<!-- programs so that we can declare `BOARD_FULL_TREBLE_OVERRIDE`(?). -->

Treble and changes from Oreo to Pie
`binder_in_vendor_violators` etc

`binder`, `hwbinder`, `vndbinder`

Cleaning commands:
```
rm out/target/product/kagura/obj/ETC/*sepolicy* -r
find out/target/product/kagura/ -iname "*sepolicy*" -exec rm -rv {} \;
find out/target/product/kagura/ -iname "*selin*" -exec rm -rv {} \;
find out/target/product/kagura/ -iname "*file_context*" -exec rm -rv {} \;
```

- https://stackoverflow.com/questions/33779286/selinux-policy-definition-for-android-system-service-how-to-setup
- https://www.all-things-android.com/content/understanding-se-android-policy-files
- https://android.stackexchange.com/questions/120061/how-selinux-protects-android-from-rooting
- https://www.whitewinterwolf.com/posts/2016/08/15/examine-android-selinux-policy/
- https://serverfault.com/questions/521078/how-can-i-query-for-all-selinux-rules-default-file-contexts-etc-affecting-a-type

- https://selinuxproject.org/page/ObjectClassesPerms
- https://selinuxproject.org/page/PolicyLanguage
- https://selinuxproject.org/page/AVCRules
- https://selinuxproject.org/page/XpermRules (allowxperm etc, important for
  defining ioctls!)


Tips:

- `restorecon -R /path`
- `ls -Z /path` or `ls -Z /path/file`
- `ps -AZ`
- Always try to find out what the macros mean!
  Check `system/sepolicy/pulic/{te_macros,global_macros}` and any file you might
  find that explains a thing that does not fall within the standard
  `rule_name source_type target_type : class perm_set;` schema.
  Especially `binder_use()` and similar macros pull in `neverallow`s and things
  you might not expect.
- Have a copy of the `te_macros` and `global_macros` printed out!
- `load_policy` to reload policy on device
- `sesearch` and `seinfo` tools to examine policy(must be installed on host, not
  available on standard android device)
- Check paths with `matchpathcon -P <policy file dir> -f <file_contexts_file>
  /my/path` (**Important:** `-P` needs to point to the *directory* containing
  the `policy` file, not the policy file itself!)

Examples:

- https://android.googlesource.com/device/google/marlin/+/android-9.0.0_r21/sepolicy/
- https://android.googlesource.com/device/google/crosshatch-sepolicy/+/android-9.0.0_r21

Different app levels:

- `seapp_contexts`
- `platform_app.te`
- `service_contexts`, `hwservice_contexts`

From domain.te:

> Only `service_manager_types` can be added to `service_manager`  
> `# TODO: rework this:`  
> `neverallow * ~service_manager_type:service_manager { add find };`

From init.te:

> Init never adds or uses services via `service_manager`  
> `neverallow init service_manager_type:service_manager { add find };`

<!-- TODO: Make this example more clear -->

Or own domain via own `myapp.te`, with:
```
type myapp, domain;
type myapp_exec, exec_type, vendor_file_type, file_type;

init_daemon_domain(myapp);
add_service(myapp, qcrilam_service)

typeattribute myapp binder_in_vendor_violators;
binder_use(myapp)

allow myapp_service self:hwservice_manager find;
```
(for more, see
https://stackoverflow.com/questions/28914215/is-it-possible-to-use-the-package-name-as-the-domain-name-in-android-selinux)

Have a look at `system/sepolicy/public/te_macros`,
`system/sepolicy/public/global_macros`

Google's guides:

- https://source.android.com/security/selinux/customize
- https://source.android.com/security/selinux/implement

## Gotchas

- Confusing domains, types vs attributes(?), labels, objects, contexts, ...
- `service_manager` (from `add_service(domain, service)`) vs `servicemanager`
  (from `binder_use(domain)`). Yes, this is extremely confusing!
- Confusing public/vendor and private policy folders/"APIs"
- Confusing objects(mostly files) and running subjects like "apps"(the Java
  ones) or "services"(mostly C binaries that run from /system/bin or
  /vendor/bin, but can also be Java apps OR HAL/HIDL implementations)
- Things can be active(searching/finding something) or passive(being labeled
  something and then being interacted with). Attributes are mostly passive,
  while subjects and their actions are active.

## seapp

Is `seapp_contexts` a labeling mechanism? Or a sanity check through neverallows?
Can you leave out everything apart from `name=com.my.app`?

`seinfo` is mainly used to enforce signing requirements, e.g. you have
`mac_permissions.xml`(MAC = Mandatory Access Control) that specifies that
anything with `seinfo=platform` has to be signed by the platform key:
```
<?xml version="1.0" encoding="utf-8"?><policy>
[...]
<!-- Platform dev key in AOSP -->
<signer signature="@PLATFORM" >
  <seinfo value="platform" />
</signer>
</policy>
```

## Taming a service: QcRilAm

<!-- TODO: Changeme to reflect changed seapp_contexts version where we changed
qcrilam to a priv-app! -->
<!-- Or maybe even better, move this into the SELinux article series instead, maybe -->
<!-- part 3 -->

**THIS IS OUT OF DATE!**

QcRilAm is our own little app/service that proxies requests from rild to
`android.hardware.radio.am`. It was written by oshmoun and is needed to get
in-call audio to work.
It consists of two parts: One HIDL implementation and one Java app that acts as
a service. The two talk via binder.
```
# Establish vnd_qcril_audio_hwservice
# hwservice_contexts
vendor.qti.hardware.radio.am::IQcRilAudio           u:object_r:vnd_qcril_audio_hwservice:s0
vendor.qti.hardware.radio.am::IQcRilAudioCallback   u:object_r:vnd_qcril_audio_hwservice:s0
```

The HIDL service(called `vendor.qti.hardware.radio.am`) gets accessed by `rild` via
hwbinder calls(since it is a `hardware` service).

```
# Allow rild to access vendor.qti.hardware.radio.am HIDL services
# rild.te
hwbinder_use(rild)
add_hwservice(rild, vnd_qcril_audio_hwservice)
```

The persistent android service part of QcRilAm is a java app that will also
appear in a user's app listing called `com.sony.qcrilam`. It only serves to run
its one service called `com.sony.qcrilam.QcRilAmService`.

Right now, it appears in the process list as a `platform_app`:

```
$ ps -AZ | grep qcrilam
u:r:platform_app:s0:c512,c768  u0_a62        2045   585 3685212  80452 SyS_epoll_wait      0 S com.sony.qcrilam
```

To use a more restricted and thus safer domain:

```
# Assign com.sony.qcrilam.QcRilAmService a domain
# service_contexts
com.sony.qcrilam.QcRilAmService         u:object_r:qcrilam_service:s0
```
But wait! We need to define what a `qcrilam_service` is first!
```
# service.te
type qcrilam_service, service_manager_type;
```
<!-- TODO: vndservice!!! -->

Same goes for `vnd_qcril_audio_hwservice`:
```
# hwservice.te
type vnd_qcril_audio_hwservice,    hwservice_manager_type;
```
Alright, now we need to assign `qcrilam_service` to a domain. We could borrow from
an already existing one, but AOSP doesn't quite provide one that suits us.
So we create our own:
```
# qcrilam.te
type qcrilam, domain;
add_service(qcrilam, qcrilam_service)
# the add_service() macro is equivalent to this:
#allow qcrilam qcrilam_service:service_manager { add find };
# TODO: Not needed with vnd change!
#typeattribute qcrilam binder_in_vendor_violators;
#binder_use(qcrilam_service)
vndbinder_use(qcrilam_service)
binder_call(qcrilam, vnd_qcril_audio_hwservice)
```

<!-- TODO: This is out of date! -->
`com.sony.qcrilam` got assigned the `platform_app` label by SELinux
automatically. This is a very high privilege level, as opposed to
`untrusted_app` contexts. We want qcrilam to find its HIDL cousin via
hwservicemanager in addition to being able to call them via binder:
```
# platform_app.te
allow platform_app vnd_qcril_audio_hwservice:hwservice_manager find;
```
A final thing to keep in mind is that `rild` is very greedy with permissions.
Modems are a black box we have to contend with. Allow platform apps to talk to
rild via binder and vice versa:
```
# rild.te
binder_call(platform_app, rild);
binder_call(rild, platform_app);
```

# Wrapping up

Something to keep in mind is: You are only allowing or disallowing things, not
influencing the flow of programs[^1]. Even if te macros can look like little
function statements, they only dictate what gets allowed, not how a program
behaves.

[^1]: Ok that one is not 100% correct, some programs will try to do things and when they get denied switch to a different strategy. But in general, you will not incluence control flow much with sepolicy.

[seandroid-aalto]: https://wiki.aalto.fi/download/attachments/100218155/SEAndroid-Aalto.pdf?version=1&modificationDate=1425973622725&api=v2
