# How to make and use streams
* https://nodejs.org/api/stream.html
* https://www.npmjs.com/package/readable-stream
* inspiration: (see pull streams)


```js
var read       = require('readable-stream')
var write      = read.Writable
var transform  = read.Transform
var duplex     = read.Duplex


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
  MAKE TRANSFORM STREAMS
******************************************************************************/
myTransformStream.__proto__ = transform.prototype
// add additional myTransformStream class methods
function myTransformStream () {
  var transform$ = transform({ objectMode: true })
  transform$.__proto__ = myTransformStream.prototype
  transform$._transform = function (newval, enc, next) {
    // do stuff
    // next(null, newval) // === PassThrough
  }
  // add additional instance only stuff
  return transform$
}

/******************************************************************************
  MAKE DUPLEX STREAMS
******************************************************************************/
myDuplexStream.__proto__ = duplex.prototype
// add additional myDuplexStream class methods
function myDuplexStream () {
  var duplex$ = duplex({
    writableObjectMode: true,
    read (size) { },
    write(chunk, encoding, next) {
      // do stuff
      // next(null, chunk)
    }
  })
  duplex$.__proto__ = myDuplexStream.prototype
  // add additional instance only stuff
  return duplex$
}


```
