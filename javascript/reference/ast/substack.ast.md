esprima and acorn are interchangeable
slightly different format
but flags to turn on and off parts of the generated AST

# burrito
```js
var fs = require('fs')
var burrito = require('burrito')
var functionList = []
fs.readFile('someFile.js', function (err, data) {
  if (err) throw err
  var dataStr = data.toString()
  burrito(dataStr, function (node) {
    if (node.name === 'function' || node.name === 'defun'){
      functionList.push(node.value[0])
    }
  })
})
```

# falafel
https://github.com/substack/jquerysf-2015

  a
 / \
 b  c
    /\
   d  e    =>  b d e c a


```js
var falafel = require('falafel')
var output = falafel(src, function (node) {
  if (node.type === 'CallExpression') {
    node.update('CALL')
    // console.log(node.type, node.source())
    // console.log('----------------------')

    // node.parent
    // node.source() // gives source code
    // node.update() // update node
  }
})
```

# static-module
```js
var staticModule = require('static-module')
var quote = require('quote-stream')
var fs = require('fs')
var sm = staticModule({
  fs: {
    readFileSync: function (file) {
      return fs.readFileSync(file)
    }
  }
})
```

# bulk-require
```js
var bulkrequire = require('bulk-require')
// ...
```

# glslify
webgl shaders using static-module
```js
var flslify = require('glslify')
var src = glslify(__dirname + 'shader.glsl')
console.log(src)
```

# static-eval
```js
var evaluate = require('static-eval')
var parse = require('acorn').parse

var src = process.argv[2]
var ast = parse(src).body[0].expression

console.log(evaluate(ast))

```

# coverify
```js
var x = 5;
var y = (x + 10) * 2
console.log(x/y)
```

# live-patch
```js

```
