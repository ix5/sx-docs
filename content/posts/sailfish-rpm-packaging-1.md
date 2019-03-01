---
title: "SailfishOS RPM Packaging - Part 1"
description: "Some tips"
date: 2019-02-27T21:37:10+01:00
draft: false
---

A short word on how `.spec` and `.inc` files work then using `rpm` packaging for
Sailfish.

## Internal variables

Rpm-internal variables - i.e. variables defined in `.spec` or `.inc` files - can
be used as follows:

1. First, define the variable, e.g. via `%define device kagura`.
2. Then use the variable via `%{device}`.

## Inclusion

`.spec` files can include `.inc` files via `%include` statements, e.g.
`%include common-defs.inc`.

It makes most sense to first define a variable in a device-specific `.spec` file
and then call an `.inc` file which uses the variable.

## Packaging

When using a single `.spec` or `.inc` file to define multiple packages, it makes
sense to use the following pattern:

[droid-hal-device.inc][dhd-name]
```rpm
Summary: 	Droid HAL package for %{rpm_device}
Name: 		droid-hal-%{rpm_device}
Version: 	0.0.6
Provides:	droid-hal
BuildRequires:  systemd
[...]
%description
%{summary}.
```
The main name for all packages is `droid-hal-%{rpm_device}`, e.g.
`droid-hal-f8331`.

Then, [further down the file][dhd-package], the sub-packages inherit the stem:
```
%package devel
# Requires: %%{name} = %%{version}-%%{release}
Provides: droid-hal-devel
Summary: Development files for droid-hal device: %{rpm_device}
%description devel
Device specific droid headers for %{rpm_device}.
Needed by libhybris
```
Thus, `%package devel` gets expanded to `droid-hal-f8331-devel`.

[dhd-name]: https://github.com/mer-hybris/droid-hal-device/blob/31c62347e952ff7ac2da79c29020e92ed8a56008/droid-hal-device.inc#L128
[dhd-package]: https://github.com/mer-hybris/droid-hal-device/blob/31c62347e952ff7ac2da79c29020e92ed8a56008/droid-hal-device.inc#L176
