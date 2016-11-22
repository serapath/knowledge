# Streams

```js
require('dom-console')()
/*******************************************************
  Something fun & unrelated
*******************************************************/
function fn1 (x) { return x*x }
function fn2 (x) { return x+9 }
function fn3 (x) { return x/5 }
var pipe = (...fns) => x => fns.reduce((v, f) => f(v), x)
var newFunc = pipe(fn1, fn2, fn3)
var result = newFunc(4)
console.log(result) // => 5
/*******************************************************
  STREAMS
  - it's a mirror of node core streams v2 and v3
  - enables use of streams in the browser and old nodejs
*******************************************************/
var read       = require('readable-stream')
var write      = read.Writable
var transform  = read.Transform
var duplex     = read.Duplex
/*******************************************************
  Detecting `end-of-stream` (or error)
  - it's hard because readable and writable have
    different APIs for detecting end of stream
*******************************************************/
var eos = require('end-of-stream')
eos(readableStream, function (err) {
  if (err) return console.log('stream close early on err')
  console.log('stream ended')
})
eos(writableStream, function (err) {
  if (err) return console.log('stream close early on err')
  console.log('stream finished')
})
/*******************************************************
  ERROR HANDLING
  - streams need to be closed individually
  - errors are not piped
*******************************************************/
var a=createStream(),b=createStream(),c=createStream()
a.on('error', function (e) { handleError(e) })
.pipe(b)
.on('error', function (e) { handleError(e) })
.pipe(c)
.on('error', function (e) { handleError(e) })
// dont use `.pipe()` (only if you know what you're doing)
// but only use `pump` instead
var pump = require('pump')
pump(a, b, c, function (err) {
  console.log('pipe finished', err)
})
setTimeout(function () {
  c.destroy() // when c is closed, pump will destroy a & b
}, 1000)
/*******************************************************
  FROM2 (convenience)
*******************************************************/
var from = require('from2')
// avoids sub classing noise for readable streams
function fromString (string) {
  return from(function (size, next) {
    // if there's no more content left in the string,
    // close the stream.
    if (string.length <= 0) return next(null, null)
    // Pull in a new chunk of text,
    // removing it from the string.
    var chunk = string.slice(0, size)
    string = string.slize(size)
    // emit "chunk" from the stream.
    next(null, chunk)    
  })
}
// pipe "hello world" out to stdout
fromString('hello world').pipe(process.stdout)
//------------------------------------------------------
var through2 = require('through2')
// avoids sub classing noise for transform streams
// - provides `.destroy()`
// `destroy` is not part of normal transform streams
// because it's not part of core which is a big problem
fs.createReadStream('ex.txt')
.pipe(through2(function (chunk, encoding, next) {
  for (var i = 0; i < chunk.length; i++) {
    if (chunk[i] === 97) chunk[i] = 122 //'a'->'z'
    this.push(chunk)
    next()
  }
}))
.pipe(fs.createWriteStream('out.txt'))
//------------------------------------------------------
var writer = require('flush-write-stream')
// avoids sub classing noise for writable streams
var ws writer(write, flush)
ws.on('finish', function () { console.log('finish') })
ws.write('hello')
ws.write('world')
ws.end()
function write (chunk, encoding, next) {
  // it's the normal `_write()` method
  console.log('writing', chunk.toString())
  next()
}
// flush is again exposed by core, but a needed feature
function flush (done) {
  // called before "finish" is emitted
  setTimeout(done, 1000) // wait one second
}
/*******************************************************
  READ STREAM
*******************************************************/
function reader () {
  var obj = read({ read: function () {}, objectMode: true })
  return obj
}
//reader().push(val)
//------------------
// optimize utilizing prototype chain
reader.prototype.__proto__ = read.prototype
function reader () {
  var obj = read({ read:function(){}, objectMode: true })
  obj.__proto__ = reader.prototype
  return obj
}
/*******************************************************
  WRITE STREAM
*******************************************************/
function writer () {
  var obj = write({ objectMode: true })
  obj._write = function (newval, enc, next) {
    console.log('writer newval=',newval)
    next()
  }
  return obj
}
//------------------
// optimize utilizing prototype chain
writer.prototype.__proto__ = write.prototype
writer.prototype._write = function (data, enc, next) {
  console.log('writer data=',data)
  next()
}
function writer () {
  var obj = write({ objectMode: true })
  obj.__proto__ = writer.prototype
  return obj
}
/*******************************************************
  TRANSFORM STREAM
*******************************************************/
function transformer () {
  var obj = transform({ objectMode: true })
  obj._transform = function (chunk, encoding, next) {
    this.push(chunk)
    next()
  }
  obj._flush = function (next) {
    next()
  }
  return obj
}
//------------------
// optimize utilizing prototype chain
transformer.prototype.__proto__ = transform.prototype
transformer.prototype._transform = function (chunk, enc, next) {
  this.push(chunk)
  next()
}
transformer.prototype._flush = function (next) { next() }
function transformer () {
  var obj = transform({ objectMode: true })
  obj.__proto__ = transformer.prototype
  return obj
}
/*******************************************************
  DUPLEX STREAM
*******************************************************/
function duplexer () {
  var obj = duplex({ objectMode: true })
  obj.__proto__ = duplexer.prototype
  obj._read = function () {}
  obj._write = function (chunk, enc, next) {
    this.push(chunk)
    next()
  }
  obj.on('data', function dontbuffer (data) {})
  return obj
}
//------------------
// optimize utilizing prototype chain
duplexer.prototype.__proto__ = duplex.prototype
duplexer.prototype._read = function () { }
duplexer.prototype._write = function (chunk, enc, next) {
  this.push(chunk)
  next()
}
function duplexer () {
  var obj = duplex({ objectMode: true })
  obj.__proto__ = duplexer.prototype
  obj.on('data', function dontbuffer (data) {})
  return obj
}
/*******************************************************
  EXAMPLES: some custom streams
*******************************************************/
function logstream () {
  var obj = write({ objectMode: true })
  obj._write = function (newval, enc, next) {
    console.log(newval)
    next()
  }
  return obj
}
function filter (fn) {
  var obj = transform({ objectMode: true })
  obj._transform = function (chunk, encoding, next) {
    if (fn(chunk)) this.push(chunk)
    next()
  }
  obj._flush = function (next) {
    next()
  }
  return obj
}
function map (fn) {
  var obj = transform({ objectMode: true })
  obj._transform = function (chunk, encoding, next) {
    this.push(fn(chunk))
    next()
  }
  obj._flush = function (next) {
    next()
  }
  return obj
}
/*******************************************************
  USAGE
*******************************************************/
var array2$ = require('from2-array')

console.log('==== Map & Filter ====')
array2$.obj([5,6,7,8,9])
  .pipe(filter(x => x>6))
  .pipe(map(x => x*10))
  // .on('data', data => console.log(data)) // => 70, 80, 90
  .pipe(logstream()) // => 70, 80, 90

console.log('====TIME: 0====')

var d = duplexer()
var log1 = logstream()
var log2 = logstream()
var log3 = logstream()


var arr1 = [{ name:'a1' },{ name:'b1' },{ name:'c1' }]
var arr2 = [{ name:'a2' },{ name:'b2' },{ name:'c2' }]

setTimeout(function () {
  console.log('====TIME: 100====')
  // in
  array2$.obj(arr1).pipe(d, { end: false })
},100)

setTimeout(function () {
  console.log('====TIME: 1000====')
  // in
  array2$.obj(arr2).pipe(d, { end: false })
  // out
  d.pipe(log1)
  d.pipe(log2)
},1000)

setTimeout(function () {
  console.log('====TIME: 1500====')
  // out
  d.pipe(log3)
}, 1500)

/*******************************************************
  For more information, check:
*******************************************************/
// try: https://www.youtube.com/watch?v=mlNUxIUS-0Q
// try: https://github.com/substack/stream-adventure
// try: https://github.com/substack/stream-handbook
```
