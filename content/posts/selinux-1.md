---
title: "SElinux on Android: Part 1"
description: "Basics: How SELinux works on Android and how it fits into Android's security approach"
date: 2018-12-22T18:07:04+01:00
slug: selinux-android-part-1-basics
draft: true
---

<!-- ## TODO: -->
<!-- - why? -> apps spying, /proc/ leaks, ... -->
<!-- - properties, labels, permissions, capabilities -->
<!-- - file layout(tree output), `BOARD_SEPOLICY_x` fusion vars -->
<!-- - differential from other mechanisms like seccomp or minijail -->

## Bird's eye view

Broadly speaking, SELinux is a mechanism to control what "things" can access
other "things" on the system. Most operating systems already have a kind of
access control: All files have an owner and information about what non-owners
can do with the file(read/write/execute).

If a file has the owner "Steve", group "ACME Co." and permissions `rwx r-x ---`,
that means the following:

- The first three letters are Steve's permissions. `rwx` means he can `r`ead,
  `w`rite and e`x`ecute it (if it's a program).
- The next three letters are for users in the owning group. A bit like if Steve
  owns the file through his company. `r-x` means all the other people inside
  "Acme Co." can `r`ead and `execute` the file, but not `w`rite/change it.
- The last three letters stand for all others on the system. `---` means no one
  apart from Steve and his coworkers at ACME Co. can access the file in any way
  \- Bob from Roadrunner Industries is not allowed.

So that's simple enough. Only the *users* with correct permissions can do
things to files. But now we have a different problem: All programs need to run
under a username. If Steve runs program `foo` under his own username, that means
program `foo` can access everything that he has access to. Alright, then maybe
creating a deficated `foo` user for the `foo` program can solve this? We would
quickly run into even more problems. Because the `foo` user would have to become
a member of a lot of groups just to be able to access some files owned by those
groups, `foo` would soon have way too much access, be much more powerful than
makes sense.

And other problems arise: Only using file terminology quickly becomes limiting.
Programs do much more than just read and write files, they also communicate with
each other, make connections to the internet and much more. How do you regulate
that? You need a more capable approach to manage all that. This is where SELinux
steps in.

We'll skip over the origins and history of SELinux here and get right to how it
is implemented in Android.


## Terminology
SELinux deals in `subjects`, `domains`, `rules`, `types` and `objects`. These
five terms make up the base grammar of the whole system.

The following is heavily borrowed from the [excellent presentation about SElinux
on Android at aalto.fi][seandroid-aalto].

The relationship between the terms is as follows:
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
[]------/                        --\-----------[]
                                    -----------[]
```

> Subjects are primarily processes. Processes that participate in the access
> control are assigned a domain.

> Objects are e.g. files, sockets.. Objects that participate in the access
> control are assigned a type.

> An object is always an OS primitive of some sort. This is not a policy issue,
> this is reality. An object therefore belongs to one or more classes which are
> predefined by the system proper. A class can be e.g. file, socket, character
> device, ....

```
Objects   Classes      Permissions

[]--           /-----------[]
[]--\-------[]-------------[]
[]-/   /      \------------[]
[]-----             /------[]
[]    \            /
[]----------[]-------------[]
```

> A class is associated with a fixed set of permissions (e.g read, write,
> open..), often closely mapped to system call functionality. Again, the set of
> available permission is predefined, policy does not change that But! An allow
> rule includes also the class, and within that the permissions allowed by the
> rule. Class and permission names are ”well known” as defined by NSA / SELinux

An implementation of SELinux consists of a set of `rules` and `libselinux`(plus
some parts inside the kernel) that processes those rules and enforces access
based on them.

The general blueprint of a `rule` is as follows:
```
allow [domain] [type] : [class] {[ allowed permissions ]};`
```

- `[domain]`: Subject (a process in a domain)
- `[type]`: Object (a file or other resource)
- `[class]`: Resource class (from predefined set)
- `[allowed_permissions]`: Class-specific permissions that are allowed by this
  rule

An example would be:
```
allow browser dns_resolver_program:binder { find call };
```
This would mean members of the `browser` domain - e.g. `firefox`- is allowed to
communicate with a DNS resolver program over the `binder` Inter-Process
Communication("IPC") system.

These rules are collected in Android's SELinux policy - `sepolicy` for short. We
will use that term moving forward.

## File access

One primary misconception you could now develop would be to think that SELinux
replaces regular Linux file permissions. That is not the case, processes still
need the proper `read/write/execute` permissions to access files. SELinux is
just another layer on top of that. That is also the reason why it is an example
of `MLS` - Multi-layered Security.

SELinux "tags" all files with `labels`. Every file on the system has a label,
even if you have not assigned one - in that case, the files simply get the label
`unlabeled`. The file `label`s are different from regular file ownership.  They
assign a kind of class to files.

File labels are defined in `file_contexts` in Android's `sepolicy`. An example
file label would be:
```
/dev/rtc0            u:object_r:rtc_device:s0
/dev/rtc1            u:object_r:rtc_device:s0
```
This would mean `/dev/rtc0` and `/dev/rtc1` get the label `rtc_device`. Only
subjects(≈processes) with access to `rtc_device` are allowed to access the `rtc`
devices now.  
You can use regular expressions in file labels, too:
```
/dev/rtc[0-9](/.*)?  u:object_r:rtc_device:s0
```

You can view file contexts on an SELinux-enabled system by typing `ls -Z`. The
`-Z` switch also works for processes(`ps -Z`) and many other SELinux-aware
programs.

## Implementing
At this point, I would recommend you take a look at the Android documentation
page for [implementing SELinux][source-selinux-implement]. However, some areas
are out of date.

The build variables to include the vendor `sepolicy` in `BoardConfig.mk` have
changed in Android Oreo; `BOARD_SEPOLICY_UNION` [is deprecated][union-dep].

`sepolicy.mk`:
```
LOCAL_SEPOLICY := device/<vendor>/sepolicy
BOARD_VENDOR_SEPOLICY_DIRS += $(LOCAL_SEPOLICY)/vendor
# Here for completeness, but ignore these two for now:
BOARD_PLAT_PUBLIC_SEPOLICY_DIR += $(LOCAL_SEPOLICY)/public
BOARD_PLAT_PRIVATE_SEPOLICY_DIR += $(LOCAL_SEPOLICY)/private
```

Because of changes for splitting `sepolicy` over `/system` and `/vendor`
necessitated by Project Treble, the `sepolicy` now consists of platform, shared and
vendor parts. We will cover the changes in Oreo and Pie later in this series,
for now just put your vendor-specific changes into the directory pointed to by
`$(LOCAL_SEPOLICY)/vendor`.

The structure should look like this inside `device/<vendor>/sepolicy`:
```
├── sepolicy.mk
└── vendor
    ├── [...]
    ├── appdomain.te
    ├── attributes
    ├── bluetooth.te
    ├── device.te
    ├── domain.te
    ├── file_contexts
    ├── file.te
    ├── genfs_contexts
    ├── hal_wifi_default.te
    ├── hwservice_contexts
    ├── hwservice.te
    ├── init.te
    ├── platform_app.te
    ├── priv_app.te
    ├── property_contexts
    ├── property.te
    ├── seapp_contexts
    ├── service_contexts
    ├── service.te
    ├── vendor_init.te
    ├── vndservice_contexts
    └── vndservice.te
```

As a vendor, you should not need to modify
`platform` or `public` sepolicy (though bugs pop can up of course).

Next, read the [customization guide][source-seliux-cust-guide] and follow along
the other parts of the documentation at [source.android.com][source-selinux].

---

This series will return in [Part Two: Intermediate Concepts][intermediate].
Once you feel confident about `labels`, `permissions`, `capabilities`,
`contexts` and the like, move on to learn about denials, macros, toolings,
neverallows and more.

---

# References

[seandroid-aalto]: https://wiki.aalto.fi/download/attachments/100218155/SEAndroid-Aalto.pdf?version=1&modificationDate=1425973622725&api=v2
[source-selinux-implement]: https://source.android.com/security/selinux/implement
[union-dep]: https://source.android.com/security/selinux/customize#policy-placement
[source-seliux-cust-guide]: https://source.android.com/security/selinux/customize
[source-selinux]: https://source.android.com/security/selinux
[intermediate]: ../selinux-android-part-2-intermediate-concepts/
