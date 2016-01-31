Electron App Env
=====

This is an experimental application environment built on [Electron](http://electron.atom.io/).
It's a browser that runs web-servers.
It's not yet complete.

A typical application:

```js
var http = require('http')
var appenv = require('eappenv')

// read config from env vars
var port = process.env.PORT

// create the HTTP server
http.createServer(function (req, res) {
  res.writeHead(200, 'Ok', { 'Content-Type': 'text/html' })
  res.end('Hello, world')
}).listen(port)

if (appenv.isActive()) {
  // register the http server for other apps to find
  appenv.registerService({
    title: 'Hello World Application',
    appname: 'helloworld',
    port: port,
    protocols: ['http'],
    interfaces: ['page']
  })
}
```

## Overview

EAppEnv is chromium + server orchestration.
It manages configuration, server-app lifecycle, and the browser windows.
This leaves the applications to behave like typical node web-apps, with a few extra APIs for working with the environment.

### App Lifecycle & Config

EAppEnv orchestrates the applications so they can depend on each other.
It runs them in child processes, and passes config via environment variables.
It also establishes an IPC interface using [node's stdio api](https://nodejs.org/api/child_process.html#child_process_options_stdio).

### Service Registry

The applications are expected to create services at their assigned port range.
They can then announce the services to EAppEnv, and query the registry.
When they find services by other applications they want to use, they can establish connections and do work.

This is an important part of the environment.
It lets applications depend on each other, and avoid duplicated functionality.

### URL Scheme

EAppEnvu introduces the `app:` scheme using [electron's protocol api](http://electron.atom.io/docs/v0.36.5/api/protocol/).
This scheme integrates with EAppEnv's service-registry to route requests to the active applications.

The scheme is very simple:

```
app-url        = "app:" <app-identifier> "/" <path>
app-identifier = [A-z0-9-._]+
path           = [A-z0-9-._~:/?#[]@!$&'()*+,;=]*
```

Some example URLs:

```
app:notifications
app:mail/compose
app:mail/view/%258gV+3yzSEUiWulnOq7aUiNMD1ckGICjPumWw8m5SG38=.sha256
````

A request to `app:notifications` would look in the service-registry for the `appId` of `notifications`.
If an entry is found, the request would be routed to that service (eg to `localhost:9999`).

Some notes about how these URLs are used:
 - The URLs are expected to point to interfaces. They are *always* redirected to an `http://localhost:<port>` address. Therefore, they should only be expected to be used as GET endpoints for HTML pages.
 - Applications can share identifiers. If there's a collision, the user's asked which application they want to use when they click on a link.

### UI

Like any browser, EAppEnv creates and manages [BrowserWindows](http://electron.atom.io/docs/v0.36.5/api/browser-window/) on behalf of the apps.
The `BrowserWindow` is created without node integration.
Therefore the application frontends behave mostly like in typical Web apps.

One crucial difference is, EAppEnv's windows have no "chrome" - no tabs or address bar.
Each window is free-standing, as you'd expect on the desktop.

(In the future, EAppEnv may add window-management behaviors, such as tiling and "tabs.")

### Default Applications

Some applications are included with EAppEnv, by default, and cannot be overridden by userland.

`app:launch` provides the standard open-window interface.

### Security & Sandboxing

EAppEnv's `BrowserWindow` will be sandboxed with standard web-security.

Unfortunately, we don't have a node sandbox tool at our disposal yet, and so the backend process is given full access to node APIs.
This will have to be a temporary condition!
Once a backend sandbox is available, EAppEnv will be able to install apps much more freely, and better protect users, which is always a good thing.

### Web / EAppEnv Portability

EAppEnv is designed to be as un-intrusive as possible.
Applications can test whether they're being run by EAppEnv, or stand-alone in node, and run correctly for either environment.

This has two results:

 1. It's easy to port an exist node web-app to EAppEnv
 2. An EAppEnv app can act as a web-app for remote access in a standard browser


## Howto Use

### Run from source

```
cd ~
git clone https://github.com/pfraze/eappenv.git
cd eappenv
npm install
npm start
```

### Testing

If you run using `npm test` instead of `npm start`, EAppEnv will load using `./test/apps` as its apps path.

### Building into a platform-specific program

```
npm run release
```

### More info

This repo is based on https://github.com/szwacz/electron-boilerplate.
Check that repo to get more information on the structure and scripts.


## Howto Write an App

Refer to [test/apps/hello-world](./test/apps/hello-world) for help.

Create a standard node project, with a `package.json` and `index.js`.
In the `package.json`, include an "app" attribute at the toplevel.
For now, it's value just needs to be truthy.


## API

The `eappenv` provides a set of methods for interacting with EAppEnv from the applications.

### registerService(opts)

### queryServices(opts)

### "ready" event

