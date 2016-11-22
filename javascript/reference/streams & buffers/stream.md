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
  WRITE STREAMS (sinks)
******************************************************************************/
myWriteStream.__proto__ = write.prototype
// @TODO: add additional myReadStream class methods
function myWriteStream () {
  var write$ = write({ objectMode: true })
  write$.__proto__ = myWriteStream.prototype
  write$._write = function (chunk, encoding, next) {
    // @TODO: do stuff with the data
    // call `next()` signals to be ready for next chunk
    // next(null) // can pass error first
  }
  // @TODO: add additional instance only stuff
  return write$
}
// USAGE
var myWrite = myWriteStream()
myWrite.write('write data to `myWrite`')
/******************************************************************************
  MAKE READ STREAMS (sources)
******************************************************************************/
myReadStream.__proto__ = read.prototype
// @TODO: add additional myReadStream class methods
function myReadStream () {
  var read$ = read({ read: function () {}, objectMode: true })
  read$.__proto__ = myReadStream.prototype
  // @TODO: add additional instance only stuff
  return read$
}
// USAGE
var mr = myReadStream()
mr.on('data', function (data) { console.log('read data from `mr`') })
// testing:
mr.push('make data available to be read from `mr`')
/******************************************************************************
  MAKE TRANSFORM STREAMS (sink for data to directly feed a connected source)
******************************************************************************/
myTransformStream.__proto__ = transform.prototype
// @TODO: add additional myReadStream class methods
function myTransformStream () {
  var transform$ = transform({ objectMode: true })
  transform$.__proto__ = myTransformStream.prototype
  transform$._transform = function (chunk, encoding, next) {
    // @TODO: do stuff with the data
    // call `next()` signals to be ready for next chunk
    // call `next(null, chunk)` pushes data to the readable queue
    // next(null, chunk) // === PassThrough
  }
  // @TODO: add additional instance only stuff
  return transform$
}
// USAGE
var mt = myTransformStream()
mt.write('write data to `mt`')
mt.on('data', function (data) {
  console.log('read transformed "write data to `mt`" from `mt`')
})
/******************************************************************************
  MAKE DUPLEX STREAMS (data sink & data source - not necessarily connected)
******************************************************************************/
myDuplexStream.__proto__ = duplex.prototype
// @TODO: add additional myReadStream class methods
function myDuplexStream () {
  // var CUSTOM = []
  var duplex$ = duplex({
    writableObjectMode: true,
    read (size) {
      // "`this.push('some data')` makes data available to listeners"
      // e.g.
      // this.push(CUSTOM.pop())
    },
    write(chunk, encoding, next) {
      // @TODO: do stuff with the data
      // call `next()` signals to be ready for next chunk
      // next(null) // can pass error first
      // e.g.
      // CUSTOM.push(chunk)
      // next()
    }
  })
  duplex$.__proto__ = myDuplexStream.prototype
  // @TODO: add additional instance only stuff
  return duplex$
}
// USAGE
var dt = myDuplexStream()
dt.write('write data to `dt`')
dt.on('data', function (data) { console.log('read data from `dt`')})
```
