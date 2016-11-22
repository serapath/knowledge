# electron
* http://electron.atom.io/
* https://github.com/sindresorhus/awesome-electron
* search: https://npms.io/
* https://github.com/electron-userland
* https://github.com/electron/electron

https://duckduckgo.com/?q=electron+quickstart&t=h_&ia=software

```bash
  mkdir demo && cd demo
  npm init
  npm install electron --save-dev
  # electron is the new name for electron-prebuilt
  # ...electron-prebuilt is phased out end of 2016
  touch main.js index.html
```

## main.js

```js
var electron = require('electron')
var app = electron.app
var Bwindow = electron.BrowserWindow

app.on('ready', function () {
  win = new Bwindow()
  win.loadURL(`file://${__dirname}/index.html`)
})
```

## index.html

```html
<html>
  <script>
    var fs = require('fs')
    var os = require('os')
    var files = fs.readdirSync(os.homedir())
    document.body.innerHTML = files.join('<br>')
  </script>
  <script src="this/works/too.js"></script>
</html>
```
## package.json
```json
{
  "name": "my-cool-demo",
  "scripts": {
    "start": "electron ."
  }
}
```

`npm start`

# features
* geolocation
* webcam
* microphone
* webrtc
* css custom properties
* css containment
* window.fetch
* ~~cors~~
* async / await
* html imports
* desktop notifications
* push notifications
* web audio api
* mouse and keyboard with robotjs module

# userspace
* ~ 10.000 github repos depend on electron
* https://npmjs.com/package/electron-npm-packages
