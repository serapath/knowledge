# Stream Modules
```js
var split2 = require('split2')
// transform text stream into a line stream
// deals with utf-8 chars

fs.createReadStrea(file)
.pipe(split2())
.on('data', function (line) {
  console.log(line)
})
```
