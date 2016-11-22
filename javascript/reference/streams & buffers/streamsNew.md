# Introduction

```js
// Streams I/O events trigger JS execution


EQ = Event Queue [,,,] => while (/*
  Node.js Event Loop
  => kernel async I/O [ [] []----[] ] => EQ
  => Thread Pool [ [...] [...] [...] ] => EQ
*/)

```

Stream is an array laid out in time rather than in memory

Streams are build around the concept of unix pipes

Data flows from upstream
Data flows to downstream

## Streams 1 (legacy mode, flowing mode)
* node v0.8
* push streams - blindly send data downstream
* backpressure problems (like observables)
* pause method is only advisory, writer can ignore it
`stream.on('data', function (chunk) { /*...*/ })`

## Streams 2
* node v0.10
* pull streams - ask upstream for more data
* backpressure auto-managed by stream when using `.pipe`
* introduce api with `.on('readable', fn)` & `.read()`
* `resume`,`pause`,`data` handlers revert to push streams
  * thus no backpressure, so don't use it when possible
```js
stream.on('readable', function () {
  for(var chunk; (chunk=stream.read()) !== null;) {
    // process chunk
  }
})
```

## Streams 3
* node v0.12
* backpressure is managed by stream, thus
  every event makes the stream pull itself
  * attach 'data' handlers
  * call 'resume'
  * pipe(writable)
* allows to attach a "data producer upstream" to
  multiple "data consumers downstream", where
  producer goes at slowest speed of consumers
* attached "data" handler will not revert "flowing mode"
  automatically

# Streams

```js
var readable  = require('readable-stream')/*
 - gets data (in chunks) from an input source
 - e.g. Standard input
*/
var writable  = readable.Writable/*
  - sends data (in chunks) to an output sink
  - e.g. Standard output
*/
var duplex    = readable.Duplex/*
  uses `._read()` and `._write()`
  - is both, readable and Writable
  - e.g. network sockets

*/
var transform = readable.Transform/*
  uses `._read()` and `._write()` and `._transform`
  - implements the duplex stream
  - manipulates data as it passes through
  - e.g. zlib
  - `PassThrough` is a transform function noop(){}
    - it's useful for hooking up non-stream events
      into a stream architecture
*/
```

# object mode
```js
// Binary streams can only process strings and buffers,
// but in "object mode", a `chunk` can also be an object
var through2 = require('throuh2')
var stream = through2({objectMode:true}, transform)
function transform (chunk, encoding, callback) {
  console.log(chunk)
  console.log(typeof chunk)
  this.push(chunk)
}
stream.write({hell: 'World!'})
```

# high water mark
* for controlling the size of back pressure buffer between two streams



---

```js
/**
* streams-generic
* Streams on the perfect world (tm).
*/
getASStreams(function (stream) {
  stream.pause() // hold! we are not ready.

  doSomethingAsync(function (err, somethingWeNeed) {
    stream.on('data', function () {
      // ok, handle data.
    })
    stream.resume() // ok, give me data.
  })
})

/**
* streams 1: Backpressure problems, non-pausable.
* Not intended to work, just explanatory
*/
getASStreams(function (stream) {
  // Data may have already come through and been lost.
  stream.pause() // Advisory

  doSomethingAsync(function (err, somethingWeNeed) {
    stream.on('data', function () {
      // ok, handle data.
    })
    stream.resume() // Advisory as well.
  })
})
/**
* streams 2: Paused from beginning, pipe Backpressure
* handled internally.
* Not intended to work, just explanatory
*/
getASStreams(function (stream) {
  // Stream is already paused, reverts the stream into
  stream.pause() // legacy mode (=flowing mode)
  // ...thus reverts from "pull" back to "push"

  doSomethingAsync(function (err, somethingWeNeed) {
    stream.on('data', function () { // attaching
      // => unpauses the stream automatically
      // handle data.
    })
    stream.resume()
  })
})
/**
* streams 3: Backpressure managed by stream.
* No data loss.
* Not intended to work, just explanatory
*/
getASStreams(function (stream) {
  // Stream is already paused from beginning
  stream.pause()
  // does nothing if stream is already paused

  doSomethingAsync(function (err, somethingWeNeed) {
    stream.on('data', function () {
      // attaching handler calls resume
    })
    stream.resume() // Already called but calls
    // read() until stream is done
  })
})
```

## example
```js
var zlib = require('zlib')
var fs   = require('fs')
var gzip$ = zlib.createGzip()
var inp$  = fs.createReadStream('input.txt')
var out$  = fs.createWriteStream('input.txt.gz')
inp$.pipe(gzip$).pipe(out$)
```


## Backpressure
* Every stream has a little default internal buffer
* When that buffer is full because downstream can't process fast enough, the stream will signal upstream to pause
