# elm thoughts
* signal graph
* feature: hot swapping
* feature: time travel debugger

* `Static Stream Graphs` vs. `Dynamic Stream Graphs`
  * `infinite streams` (continue) vs. `finite streams` (end)
    * = connected to outside world vs. internal state?
  * `hot streams` (buffer) vs. `cold streams` (pause)
  * `push` vs. `pull`
    * event driven => push driven
    * ...pull streams...


inputs -> transformations/state -> merge

* Signals are connected to the world
* Signals are infinite
* Signal graphs are static

# examples
* **input** Mouse.position: Signal (int, int)
* **input** Keyboard.lastPressed: Signal int

* **transforms** lift (a=>b) => Signal a => Signal b
  * lift isConsonant Keyboard.lastPressed

```js
var press$ = ehs(e => String.fromCharCode(e.keyCode||e.charCode))
var isUpper$ = transformer(c => c === c.toUpperCase())
press$.pipe(isUpper$).on('data', x=>console.log(x))
```

* **transforms** foldp (a=>b=>b) => b => Signal a => Signal b
  * (fold from the past)
```js
var count$ = transformer((c=>_=>c++)(0))
press$.pipe(count$).on('data', x=>console.log(x))
```

* **merge** Signal A => Signal A => Signal A
* **lift2** (a=>b=>c) => Signal A => Signal B => Signal C
```js
// ???
```
* **split**
```js
// ???
```

# helpers

```js
  var ehs = require('eventhandler-stream')
  var read = require('readable-stream')
  var transform = read.Transform
  function transformer (fn) {
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
```
