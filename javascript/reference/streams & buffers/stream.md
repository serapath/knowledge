# How to make and use streams

```js
var read       = require('readable-stream')
var write      = read.Writable
// ...
var duplex     = read.Duplexable


/******************************************************************************
  MAKE WRITE STREAMS
******************************************************************************/
myWriteStream.__proto__ = write.prototype
// add additional myReadStream class methods
function myWriteStream () {
  var write$ = write({ objectMode: true })
  write$.__proto__ = myWriteStream.prototype
  write$._write = function (newval, enc, next) {
    _put(o, p, key, newval, next)
  }
  // add additional instance only stuff
  return write$
}
/******************************************************************************
  MAKE READ STREAMS
******************************************************************************/
myReadStream.__proto__ = read.prototype
// add additional myReadStream class methods
function myReadStream () {
  var read$ = read({ read: function () {}, objectMode: true })
  read$.__proto__ = myReadStream.prototype
  // add additional instance only stuff
  return read$
}
/******************************************************************************
  MAKE DUPLEX STREAMS
******************************************************************************/
// ...
```
