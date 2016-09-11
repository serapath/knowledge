# Scaffold
The structure below is a snippet from the code base I use to scaffold
new projects with some basic files and folders I always find myself to
create manually anyways :-)

I welcome suggestions/questions/discussions - just drop me a message :-)

http://twitter.com/serapath

> @TODO: make it modular by creating certain folders or files and there
  parameterized content as seperate modules in order to be able to compose
  scaffolds

```js
function starter (name) {
  return {
    '@TODO/'        : {
      'README.md'     : `
         # @TODO
         * build the ${name} project
      `
    },
    'node_modules/' : { /* npm install */ },
    'public/'       : {
      // 'server'        : { /* docker, linux-image  */ }
      // 'cordova'       : { /* .apk's, ...          */ },
      // 'electron'      : { /* .exe, ...            */ },
      // 'atom'          : { /* plugin               */ },
      // 'chromeplugin'  : { /* extension            */ },
      // 'firefoxplugin' : { /* extension            */ },
      'browser/'      : {  // @TODO: "generate target" the same way "cordova" or "electron" and others in public/ are generated
        // 'assets'       : { },
        // 'bundle.js'     : '// content',
        'index.html'    : `<body><script src="bundle.js"></script></body>`,
      }
    },
    'source/'       : {
      'node_modules/' : { /* _customModules */ },
      'index.js'      : `
        console.log('${name}')
      `
    },
    '.gitignore'    : `
      @TODO/
      node_modules/
      !source/node_modules/
      npm-debug.log
    `,
    'index.html'    : `<iframe src="public/browser/index.html"></iframe>`, // @TODO iframe loading/initializing "app loader & manager (see gist)"
    'package.json'  : `
      {
        "scripts": {
          "start": "ecstatic --root public/browser & npm run watch & npm run cordova",
          "watch": "watchify source/index.js -p [ browserify-hmr -u http://$(my-local-ip):3123 ] -p ghostmodeify -t babelify -o public/browser/bundle.js -dv",
          "bundle": "browserify source/index.js -t babelify -o public/browser/bundle.js -dv",
          "cordova": "cpy public/browser/**/* public/cordova/www/ && gaze 'cpy $path public/cordova/www/' 'public/browser/**/*'",
          "electron": "# @TODO: build electron"
        }
      }
    `, // @TODO: upgrade "npm run bundle" to include optimizations (uglify, ...)
    'README.md'     : `# ${name}`
  }
}
```
