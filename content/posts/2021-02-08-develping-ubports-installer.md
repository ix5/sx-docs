---
title: "Developing for the Ubports Installer"
date: 2021-02-08T18:37:50+01:00
draft: false
slug: developing-ubports-installer
---

This post goes over adding a new device to the [ubports installer][installer]
for Ubuntu Touch and helps ensure that the configs were valid.

You need to have `python`, `nodejs` and `npm` installed.

### Set up the installer
```sh
git clone https://github.com/ubports/ubports-installer
cd ubports-installer
```

The README file tells you to run the `setup-dev` script, but it's safer to
validate ourselves what it does.

```sh
# Install required programs and npm packages
./setup-dev.sh
# Same as:
apt install nodejs npm libgconf-2-4
npm install
```

We can now launch the installer with `npm start`. For more verbose output, use
`npm start -- -vv`.

There are other "scripts" available to run via `run run`, for example
`npm run lint`. You can discover all available commands in [package.json][pkg]
under the `scripts: {...}` section.

### Set up device configs

To begin setting up our own device in the installer, we need to update the
separately-kept [installer configs][configs].

```sh
git clone https://github.com/ubports/installer-configs
cd installer-configs
npm install
```

Make your changes to `v2/<device>.yml`, then check if the YAML syntax is valid
with `npm run validate`.

The spec for the YAML schema is documented well, and you can look at existing
devices for inspiration.

Once we have created or updated our own installer configs, we need to let the
installer know about them. Let the build command convert the configs into JSON
files and serve them via a simple HTTP server:
```sh
npm run buildconfigs
cd public
python -m http.server
```

Now, we instruct the installer to fetch our updated configs from our own endpoint.

By default, the installer fetches available devices and configs from
[ubports.github.io/installer-configs/v2/](https://ubports.github.io/installer-configs/v2/).

The URL to fetch from is set in [src/core/api.js][api], and the installer
expects a response in JSON format.

Change
```js
baseURL: "https://ubports.github.io/installer-configs/v2/",
```
to:
```js
baseURL: "http://localhost:8000/v2/",
```

We also need to change the [function fetching the index file][json] a bit, from:
```js
const getIndex = () => api.get("/").then(({ data }) => data);
```
to:
```js
const getIndex = () => api.get("/index.json").then(({ data }) => data);
```

Now the installer fetches your local configs and you can test whether it
succeeds.

[installer]: https://github.com/ubports/ubports-installer
[configs]: https://github.com/ubports/installer-configs
[pkg]: https://github.com/ubports/ubports-installer/blob/95745e519351adc3189e7e285a553bc6c1e5100a/package.json#L27-L35
[api]: https://github.com/ubports/ubports-installer/blob/95745e519351adc3189e7e285a553bc6c1e5100a/src/core/api.js#L25
[json]: https://github.com/ubports/ubports-installer/blob/95745e519351adc3189e7e285a553bc6c1e5100a/src/core/api.js#L33
