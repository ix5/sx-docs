---
title: "SElinux on Android: Part 2"
description: "Intermediate concepts"
date: 2019-03-06T18:07:04+01:00
slug: selinux-android-part-2-intermediate-concepts
draft: true
---

## Stuff:
- reading denials, audit2allow
- better tools: sesearch, semanage, matchpathcon
- label types: properties, seapp(later), files, capabilities, binder(later), services(later), sockets, attributes, file types, domains, devices, services, ...
- working with aosp policy, private/public/vendor
- neverallows
- macros
- `appdomain` and `untrusted_app`
- difference between `file_labels` and `genfs_contexts`(and limitations of the
  latter)

`_contexts` syntax(no macros!) -> m4defs limitations -> wrong! see
system/sepolicy/README:
> Additionally, OEMs can specify BOARD_SEPOLICY_M4DEFS to pass arbitrary m4
> definitions during the build. A definition consists of a string in the form of
> macro-name=value. Spaces must NOT be present. This is useful for building
> modular policies, policy generation, conditional file paths, etc. It is
> supported in the following file types:
> - All *.te and SE Linux policy files as passed to checkpolicy
> - file_contexts
> - service_contexts
> - property_contexts
> - keys.conf

[Part Three: Treble and onward: Security changes][??????????????????????????????TODO].
