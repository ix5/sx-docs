---
title: "Signed App seapp_contexts"
description: ""
date: 2020-05-22T14:58:03+02:00
draft: false
---

By default, most platform developers set their apps to be signed with the
platform key by using `LOCAL_CERTIFICATE := PLATFORM` in make or `certificate:
platform` in blueprints. All apps signed by the platform key get assigned the
`platform_app` SELinux label by default and have elevated privileges and access
to protected broadcasts and permissions.

That approach falls flat when using GSIs though, as the "platform" key is now
recognized to be the key used to sign the GSI, which might differ from the one
used to generated the `/vendor` parts.

A better approach is to generate separate keys for apps in the vendor image and
get SELinux to recognize them.

**Side note:** This still leaves the issue unsolved at the framework level.
Android's frameworks-base defines lots of permissions and broadcasts with the
`protectionLevel=signature`, which means only apps signed with the platform key
can utilize them. Very few permissions are set as `signature|privileged`, which
means they are not avilable by default, but can be granted through
`privapp-permissions.xml`, see [Privileged Permissions Whitelisting][privapp].

This approach still only works for system applications though. For apps on
`/vendor` an additional `vendorPrivileged` clause in the `protectionLevel` is
needed, see for example the permission `BIND_IMS_SERVICE`, which is
vendor-accessible: [AndroidManifest.xml][vendorpriv].

### Certs & Inclusion

```
./development/tools/make_key \
    app_cert \
    "/C=US/ST=California/L=Mountain View/O=Android/OU=Android/CN=Android/emailAddress=android@android.com"
```
Generates `app_cert.pk8` and `app_cert.x509.pem`

Include the cert:
```
android_app_certificate {
    name: "ModemConfig_app_cert",
    certificate: "app_cert",
}

android_app {
    name: "ModemConfig",
    // Change this:
    //certificate: "platform",
    // To this:
    certificate: ":ModemConfig_app_cert",
    // [...]
}
```

### Exotic files

First, let SELinux recognize any signature by the `app_cert` key, as defined
by the public key file `app_cert.x509.pem`, as the signer value `@MODEMCONFIG`.

`keys.conf`:
```
[@MODEMCONFIG]
ALL : SonyOpenTelephony/ModemConfig/app_cert.x509.pem
```
The `ALL` statement is a shorthand for `USER`, `USERDEBUG` and `ENG`. You could
also define this certificate to only be valid for debugging.

MAC stands for *Mandatory Access Control.* This file translates recognized
signer values into `seinfo` values that sepolicy vectors can understand and use.

`mac_permissions.xml`:
```
<policy>
    <signer signature="@MODEMCONFIG" >
        <seinfo value="modemconfigapp" />
    </signer>
</policy>
```

### sepolicy

Now, onto the labeling. App SELinux context recognition has a different syntax
than e.g.  `file_contexts` or `service_contexts`.

`seapp_contexts`:
```
user=_app seinfo=modemconfigapp name=com.sony.opentelephony.modemconfig isPrivApp=true domain=modemconfig_app type=app_data_file
```
Note: For `seapp_contexts`, the syntax is a bit confusing. `user`, `seinfo`,
`name` and `isPrivApp` are *matching selectors.* `domain` and `type` are
*assignments.*

Thus, the previous statement reads:

> Match any app running under the `_app` user (e.g. not system user), with a
> signature that translates to the `modemconfigapp` `seinfo` value, with the
> name `com.sony.opentelephony.modemconfig`, which is a privileged app.

> Assign that app the `modemconfig_app` domain and for its data directory under
> `/data/`, use the `app_data_file` label.

Now we can actually write policy for the application.

`modemconfig_app.te`:
```
type modemconfig_app, domain;
app_domain(modemconfig_app)
# [...]
```

---

For more information, see the AOSP sources: [keys.conf][keys],
[mac_permissions.xml][perms], [seapp_contexts][contexts],
[post_process_mac_perms][post].

Thanks Luk for the inspiration, see [this commit][modemconf] and [this gist][gist].

[modemconf]: https://github.com/luk1337/SonyOpenTelephony/commit/9491a7180eea458ca14b01f06644021932a243f9
[gist]: https://gist.github.com/luk1337/f5611b777d5fb4c8a1f67517a660db8d
[keys]: https://android.googlesource.com/platform/system/sepolicy/+/refs/tags/android-10.0.0_r35/private/keys.conf
[perms]: https://android.googlesource.com/platform/system/sepolicy/+/refs/tags/android-10.0.0_r35/private/mac_permissions.xml
[contexts]: https://android.googlesource.com/platform/system/sepolicy/+/refs/tags/android-10.0.0_r35/private/seapp_contexts
[vendorpriv]: https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-10.0.0_r35/core/res/AndroidManifest.xml#2123
[privapp]: https://source.android.com/devices/tech/config/perms-whitelist
[post]: https://android.googlesource.com/platform/system/sepolicy/+/refs/tags/android-10.0.0_r35/tools/post_process_mac_perms
