---
title: "Debugging JACK"
date: 2018-12-30T09:18:00+01:00
draft: true
author: Felix
---

## Some common errors and how to fix them

On Android 8.x, JACK needs some extra packages. check with `jack-diagnose`

In a bare buildroot, these might be missing, so install them:
```
apt install curl lsof
``

In a buildroot, if somehow installing the JACK .jar files into `~/.jack-server`
fails, run this on the host and bind-mount `~/.jack-server` into the target:

```
./prebuilts/sdk/tools/jack-admin install-server \
  ./prebuilts/sdk/tools/jack-launcher.jar \
  ./prebuilts/sdk/tools/jack-server-4.11.ALPHA.jar
```

(`jack-server-4.11.ALPHA.jar` might be named differently depending on the version
of android you are building)

Then ensure that `~/.jack-server` contains these files:
```
$ ls ~/.jack-server
jack/  client.jks  config.properties  server-1.jar  server.pem
logs/  client.pem  launcher.jar       server.jks
```
