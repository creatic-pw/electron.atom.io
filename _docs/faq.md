---
version: v1.4.4
category: ignore
redirect_from:
    - /docs/faq/electron-faq/
    - /docs/v0.24.0/faq/
    - /docs/v0.25.0/faq/
    - /docs/v0.26.0/faq/
    - /docs/v0.27.0/faq/
    - /docs/v0.28.0/faq/
    - /docs/v0.29.0/faq/
    - /docs/v0.30.0/faq/
    - /docs/v0.31.0/faq/
    - /docs/v0.32.0/faq/
    - /docs/v0.33.0/faq/
    - /docs/v0.34.0/faq/
    - /docs/v0.35.0/faq/
    - /docs/v0.36.0/faq/
    - /docs/v0.36.3/faq/
    - /docs/v0.36.4/faq/
    - /docs/v0.36.5/faq/
    - /docs/v0.36.6/faq/
    - /docs/v0.36.7/faq/
    - /docs/v0.36.8/faq/
    - /docs/v0.36.9/faq/
    - /docs/v0.36.10/faq/
    - /docs/v0.36.11/faq/
    - /docs/v0.37.0/faq/
    - /docs/v0.37.1/faq/
    - /docs/v0.37.2/faq/
    - /docs/v0.37.3/faq/
    - /docs/v0.37.4/faq/
    - /docs/v0.37.5/faq/
    - /docs/v0.37.6/faq/
    - /docs/v0.37.7/faq/
    - /docs/v0.37.8/faq/
    - /docs/latest/faq/
breadcrumb: FAQ
source_url: 'https://github.com/electron/electron/blob/master/docs/faq.md'
title: "Electron FAQ"
sort_title: "electron faq"
---

# Electron FAQ

## When will Electron upgrade to latest Chrome?

The Chrome version of Electron is usually bumped within one or two weeks after
a new stable Chrome version gets released. This estimate is not guaranteed and
depends on the amount of work involved with upgrading.

Only the stable channel of Chrome is used. If an important fix is in beta or dev
channel, we will back-port it.

For more information, please see the tutorial/security.

## When will Electron upgrade to latest Node.js?

When a new version of Node.js gets released, we usually wait for about a month
before upgrading the one in Electron. So we can avoid getting affected by bugs
introduced in new Node.js versions, which happens very often.

New features of Node.js are usually brought by V8 upgrades, since Electron is
using the V8 shipped by Chrome browser, the shiny new JavaScript feature of a
new Node.js version is usually already in Electron.

## How to share data between web pages?

To share data between web pages (the renderer processes) the simplest way is to
use HTML5 APIs which are already available in browsers. Good candidates are
[Storage API][storage], [`localStorage`][local-storage],
[`sessionStorage`][session-storage], and [IndexedDB][indexed-db].

Or you can use the IPC system, which is specific to Electron, to store objects
in the main process as a global variable, and then to access them from the
renderers through the `remote` property of `electron` module:

```javascript
// In the main process.
global.sharedObject = {
  someProperty: 'default value'
}
```

```javascript
// In page 1.
require('electron').remote.getGlobal('sharedObject').someProperty = 'new value'
```

```javascript
// In page 2.
console.log(require('electron').remote.getGlobal('sharedObject').someProperty)
```

## My app's window/tray disappeared after a few minutes.

This happens when the variable which is used to store the window/tray gets
garbage collected.

If you encounter this problem, the following articles may prove helpful:

* [Memory Management][memory-management]
* [Variable Scope][variable-scope]

If you want a quick fix, you can make the variables global by changing your
code from this:

```javascript
const {app, Tray} = require('electron')
app.on('ready', () => {
  const tray = new Tray('/path/to/icon.png')
  tray.setTitle('hello world')
})
```

to this:

```javascript
const {app, Tray} = require('electron')
let tray = null
app.on('ready', () => {
  tray = new Tray('/path/to/icon.png')
  tray.setTitle('hello world')
})
```

## I can not use jQuery/RequireJS/Meteor/AngularJS in Electron.

Due to the Node.js integration of Electron, there are some extra symbols
inserted into the DOM like `module`, `exports`, `require`. This causes problems
for some libraries since they want to insert the symbols with the same names.

To solve this, you can turn off node integration in Electron:

```javascript
// In the main process.
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({
  webPreferences: {
    nodeIntegration: false
  }
})
win.show()
```

But if you want to keep the abilities of using Node.js and Electron APIs, you
have to rename the symbols in the page before including other libraries:

```html
<head>
<script>
window.nodeRequire = require;
delete window.require;
delete window.exports;
delete window.module;
</script>
<script type="text/javascript" src="jquery.js"></script>
</head>
```

## `require('electron').xxx` is undefined.

When using Electron's built-in module you might encounter an error like this:

```
> require('electron').webFrame.setZoomFactor(1.0)
Uncaught TypeError: Cannot read property 'setZoomLevel' of undefined
```

This is because you have the [npm `electron` module][electron-module] installed
either locally or globally, which overrides Electron's built-in module.

To verify whether you are using the correct built-in module, you can print the
path of the `electron` module:

```javascript
console.log(require.resolve('electron'))
```

and then check if it is in the following form:

```
"/path/to/Electron.app/Contents/Resources/atom.asar/renderer/api/lib/exports/electron.js"
```

If it is something like `node_modules/electron/index.js`, then you have to
either remove the npm `electron` module, or rename it.

```bash
npm uninstall electron
npm uninstall -g electron
```

However if your are using the built-in module but still getting this error, it
is very likely you are using the module in the wrong process. For example
`electron.app` can only be used in the main process, while `electron.webFrame`
is only available in renderer processes.

[memory-management]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management
[variable-scope]: https://msdn.microsoft.com/library/bzt2dkta(v=vs.94).aspx
[electron-module]: https://www.npmjs.com/package/electron
[storage]: https://developer.mozilla.org/en-US/docs/Web/API/Storage
[local-storage]: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
[session-storage]: https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage
[indexed-db]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
