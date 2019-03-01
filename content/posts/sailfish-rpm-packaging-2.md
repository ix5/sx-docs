---
title: "SailfishOS RPM packaging - Part 2"
description: "A small example"
date: 2019-02-28T17:18:41+01:00
draft: false
---

The most common use case for packaging is to have the output of a GNU `make`
setup be wrapped into a neat `.rpm` file. But we will take something more simple
for this example - we only want to install a single configuration file.

*You can view the finished project at [github][marina-porter-tools].*

So, first we need to decide what we want to install and where to. In our case,
the file is a [configuration file][journald-config] that tells `journald` to
save the system logs across reboots. It shall end up in
`/etc/system/journald.conf.d/journald-porting.conf`.
We now put the source files of the package into `sparse/*`.

Now we need to tell the packaging system some metadata about our project.  
[marina-porter-tools.spec][spec-file]
```
Name:	    marina-porter-tools
Summary:    Extra debugging tools and configs for porting
Version:    1
Release:    1
```

The interesting part is the `%install` section:
```
%install
mkdir -p %{buildroot}/
cp -r sparse/* %{buildroot}/
```
This tells the packaging system to install all files from `sparse` to the RPM
temporary `buildroot`, creating the final filesystem hierarchy already.

Now we need to tell rpm *explicitly* which files are contained inside the
finished package. We could do it the easy way:
```
%files
/etc/systemd/journald.conf.d/journald-porting.conf
```
But we can tell the package manager more about the installed files. In our case,
we want the user to be able to overwrite the config files we installed, so we
supply more metadata:
```
%files
%defattr(-,root,root,-)
%config /etc/systemd/journald.conf.d/journald-porting.conf
```

And that's it. Assuming you cloned [the example repo][marina-porter-tools] to
`hybris/mw/marina-porter-tools`, you can now use the `droid-hal-device` helper
script to build the finished `.rpm`:
```
./rpm/dhd/helpers/build_packages.sh \
    --build hybris/mw/marina-porter-tools \
    --spec rpm/marina-porter-tools.spec
```

Copy the generated `.rpm` file from `droid-local-repo/$DEVICE` to your phone and
install it via `pkcon` - or add the package to your device's build requirements
to ease early porting.

---

## Further references
- *Maximum RPM: Taking the RPM Package Manager to the Limit:*
  [Creating the Spec File](http://ftp.rpm.org/max-rpm/s1-rpm-build-creating-spec-file.html)
  and [Directives For the %files list](http://ftp.rpm.org/max-rpm/s1-rpm-inside-files-list-directives.html)
- [Fedora docs: Creating a Basic Spec file](https://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/Packagers_Guide/sect-Packagers_Guide-Creating_a_Basic_Spec_File.html)

[marina-porter-tools]: https://github.com/marina-bay/marina-porter-tools/tree/e7dafefc1afbdbe8b7bda2415c6a64ad20c23806
[journald-config]: https://github.com/marina-bay/marina-porter-tools/blob/e7dafefc1afbdbe8b7bda2415c6a64ad20c23806/sparse/etc/systemd/journald.conf.d/journald-porting.conf
[spec-file]: https://github.com/marina-bay/marina-porter-tools/blob/e7dafefc1afbdbe8b7bda2415c6a64ad20c23806/rpm/marina-porter-tools.spec
