+++
title = "SELinux"
description = "How to configure"
date = 2018-12-08T22:15:22+02:00
weight = 15
draft = false
bref = "How to configure"
toc = false
+++

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

Sepolicy macros

Testing it out: either `mka sepolicy` or `make bootimage`
But faster: `make -j $(nproc) sepolicy_tests`!

```
PRODUCT_FULL_TREBLE_OVERRIDE := false
BOARD_USE_ENFORCING_SELINUX := true
```

Treble and changes from Oreo to Pie
`binder_in_vendor_violators` etc

`binder`, `hwbinder`, `vndbinder`

Interaction with closed-source "secure" stuff, `rild`

Apps, services, files, binaries

How to read denials

Where to look for reference (marlin, crosshatch, crosshatch-sepolicy)  
And permalinks to indiviual lines of policy

Things that can be left as-is (via `dontaudit`)

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

- https://selinuxproject.org/page/ObjectClassesPerms
- https://selinuxproject.org/page/PolicyLanguage
- https://selinuxproject.org/page/AVCRules
- https://selinuxproject.org/page/XpermRules

Tips:

- `restorecon -R /path`
- `ls -Z /path` or `ls -Z /path/file`

Examples:

- https://android.googlesource.com/device/google/marlin/+/android-9.0.0_r21/sepolicy/
- https://android.googlesource.com/device/google/crosshatch-sepolicy/+/android-9.0.0_r21

Different app levels:

- `seapp_contexts`
- `platform_app.te`
- `service_contexts`, `hwservice_contexts`

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


## How we tamed QcRilAm

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
typeattribute qcrilam binder_in_vendor_violators;
binder_use(qcrilam_service)
binder_call(qcrilam, vnd_qcril_audio_hwservice)
```
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

Ideally, we would like the Qualcomm blobs to function as vndbinder/vndservice
programs so that we can declare `BOARD_FULL_TREBLE_OVERRIDE`(?).

Something to keep in mind is: You are only allowing or disallowing things, not
influencing the flow of programs[^1]. Even if te macros can look like little
function statements, they only dictate what gets allowed, not how a program
behaves.

[^1]: Ok that one is not 100% correct, some programs will try to do things and when they get denied switch to a different strategy. But in general, you will not incluence control flow much with sepolicy.
