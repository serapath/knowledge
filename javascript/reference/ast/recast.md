# recast

```js
var recast = require('recast')
var types = recast.types
var traverse = types.traverse
var n = types.namedTypes
var b = types.builders

// 1. parse
var ast = reacast.parse(
  "[3, 1, 10, 28].sort((a, b) => a - b)"
)
// 2. traverse
types.traverse(ast, function (node) {
  if (n.ArrowFunctionExpression.check(node)) {
    var body = node.body
  }
  if (node.expression) {
    node.expression = false
    body = b.blockStatement([b.returnStatement(body)])
  }

  var funExp = b.functionExpression(
    node.id, node.params, body,
    node.generator, node.expression
  )
  var bindExp = b.callExpression(
    b.memberExpression(funExp, b.identifier('bund'), false),
    [b.thisExpression()]
  )
  this.replace(bindExp)
})
// 3. generate
console.log(recast.print(ast).code)
// prints
[3, 1, 10, 28].sort(function(a, b) {
  return a - b;
}.bind(this))
```

# example (es2015 arrow function)
non-destructive partial source transformation

```js
// input
function max (a, ...rest) {
  rest.forEach(x => { if (x > a) a = x })
  return a
}
// output
function max (a) {
  var rest = Array.prototype.slice.call(arguments, 1)
  rest.forEach(function (x) { if (x > a) a =x }.bind(this))
  return a
}
// transformer
traverse(ast, function (node) {
  if (n.Function.check(node) && node.rest) {
    var sliceCallee = mx(mx(mx("Array", "prototype"), "slice"), "call")

    var sliceArgs = [
      b.identifier('arguments'),
      b.literal(node.params.length)
    ]

    var sliceCall = b.callExpression(sliceCallee, slicedArgs)

    var restVarDecl = b.variableDeclaration('var', [
      b.variableDeclarator(node.rest, sliceCall)
    ])

    node.body.body.unshift(restVarDecl)
    node.rest = null
  }
})
```

# non-destructive partial source transformation
write some one-time-code-modifier to match multiple
idioms and translate them into pretty looking modern

```bash
# the transform script should be absolutely
# LITTERED with fail-stop assertions!

# massive test suit is very helpful

# set yourself up to iterate rapidly
# accommodating more and more exotic cases
# as you encounter them

# feel free to fix rare cases by hand, but stack
# them in separate commits

# make transform scripts 'idempotent', NO EXCUSES!!!

# use "gnu parallel" to run the transform script in
# many processes simultaneously

find ~/www/html/js/lib | \
  grep "\.js$" | \
  time parallel ~/www/scripts/bin/classify --update
# 228.03s user 12.25s system 1229% cpu 19.548 total
```
