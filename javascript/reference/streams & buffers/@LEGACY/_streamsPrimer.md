# Buffers and Streams

**Buffers** - usually the first place to experience bottlenecks in any application

* `Observables` are `Streams v1`
* Later Stream versions support back pressure


```js
//support
// `.write()` to write data
// `.end()` to end the stream

// in node core, the take `buffers` or `strings`
// an encoding can be set
// they can be passed a callback

// they emit the 'finish' event when it's done writing

// they emit 'end' event when there is no more data
// they emit 'data' event when the stream has data

var writableFile$ = fs.createWriteStream('./path')
var opts = { method: 'POST' }
// http callback is more or less the only one without 'err' first
var outData = []
var writableHttp$ = https.request(opts, function (readableHttp$) {
  readableHttp$.on('data', data => outData.push(data))
  readableHttp$.on('error', callback)
  readableHttp$.on('end', callback(null, Buffer.concat(outData)))  
})
```

# Transform Streams (=build an assembly line)
```js
var transform = new stream.Transform({
  transform(chunk, encoding, next) {
    doElseSomethingAsync(chunk, (err, resp) => {

      if (err) return next(err)
      if (!resp) return next()
      if (!Array.isArray(resp)) return next(null, resp)
      for (let item of resp) this.push(item)
      next()

    })
  }
})
// writable.on('end'), then
// transform.on('flush'), then
// readable.on('end')

// => flush is not running before the writable stream ended
```

```js
// writable streams have an `.end()` method
// readable streams have an `end` event

// BACK PRESSURE

// if you want automatic back pressure handling, then:
// => don't do anything async inside a data event handler

readableFile$.pipe(transform).pipe(new stream.Writable( {
  write(chunk, encoding, next) {
    doSomethingAsync(chunk, next)
  }
})).on('finish', function () {
  console.log("pipe stream ended on 'finish', not 'end' event")
})
```

# Example JSON stream

```js
function makeJsonStream () {
  var first = true
  return new stream.Transform({
    objectMode: true,
    transform (chunk, encoding, next) {
      if (first) {
        this.push('[')
        first = false
      } else {
        this.push(',')
      }
      this.push(JSON.stringify(chunk))
      next()
    }
    flush (done) {
      this.push(']')
      done()
    }
  })
}
// OR
var JSONStream = require('json-stream') // 1 or 2 or 3

app.get('/path.json', (req, res) => {
  res.type('json')
  getData().pipe(JSONStream.stringify()).pipe(res)
})

```
