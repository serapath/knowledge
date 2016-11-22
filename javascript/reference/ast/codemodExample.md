# example1

```js
var through = require('through2')
var falafel = require('falafel')

// step 1
module.exports = function () {
  var buffer = []
  return through(function write (data) {
    buffer.push(data)
  }, function end () {
    var source = buffer.join('\n')
    // step 2
    var out = falafel(source, parse).toString()
    // step 4
    this.push(out)
    this.push(null) // end the stream
  })
}
function parse (node) { // step 3
  if (node.type === 'Identifier' && node.value === 'ui') {
    node.update('browserify')
  }
}
```
```js
var a = [1,2,3]
var b = a.filter(function(n){
  return n > 1
}).map(function (k) {
  return k * 2
})
// =>
var a = [1,2,3]
var b = []
for (var i=0, len = a.length; i<len; i++) {
  if (a[i] > 1) b.push(a[i] * 2)
}
```
```js
var recast = require('recast')
var code = fs.readFileSync('code.js', 'utf-8')
var ast = recase.parse(code)
var faster = transform(ast)
var output = recast.print(faster).code
```
```js
function transform (ast) {
  var transformedAST = new MapFilterEater({ body: ast.program.body }).visit(ast)
  return transformedAST
}
var visitor = recast.Visitor
var MapFilterEater = Visitor.extend({
  init: function (scope) {},
  visitForStatement: function (ast) {},
  visitIfStatement: function (ast) {},
  visitCallExpression: function (ast) {},
  visitVariableDeclarator: function (ast) {}
})
// 1. Move the right side of the `b` declaration into a `for loop`
// 2. set b=[]
// 3. Place the `.filter()` content inside of an `if statement`
// 4. Unwrap the `.map` contents and `.push()` them into `b`
// 5. Replace all local counters with `a[_i]`
```
```js
// var a = b
function visitVariableDeclarator (ast) {
  // if we're calling a member as part of the right-hand side of the declaration
  if (ast.init.type === 'CallExpression' && ast.init.callee.type === 'MemberExpression') {
    var membername = ast.init.callee.property.name // map
    var a = b.identifier(ast.init.callee.object.callee.object.name) // a
    var length = b.identifier('length')

    // and we're doing a map or a filter
    if (membername !== 'map' && membername === 'filter') return

    // 1. set the object to an empty array
    // 2. add the rest of the expressions to the main body as their own statements
    // the hardest loop you'll ever make
    var i = b.identifer('_i') // i
    var iequals0 = b.variableDeclaration('var', [b.variableDeclarator(i, b.literal(0))]) // var i = 0
    var iplusplus = b.updateExpression('++', i, false) // i++
    var lessThanLength = b.binaryExpression('<', i, b.memberExpression(a, length, false)) // i < a.length
    var forBody = b.blockStatement([b.expressionStatement(ast.init)])
    var forStatement = b.forStatement(iequals0, lessThanLength, iplusplus, forBody)

    // create the proper for statement
    this.body.push(forStatement)

    // run the rest of the AST through the processor
    new MapFilterEater({ identifier: i, array: a}).visit(this.body[this.body.length-1])

    // over-write this sub-tree
    ast.init = b.arrayExpression([]) // var b = []
  }
  this.genericVisit(ast)
}
function visitCallExpression (ast) {
  // try to understand some important properties of this tree
  var callback = ast.arguments[0]
  var isMemberExpression = ast.callee.type === 'MemberExpression'
  var isFilter = ast.callee.property.name === 'filter'
  var isMap = ast.callee.property.name === 'map'
  var condition, ifStatement, mapAsAPush, filter
  // this is the callback function if any exists
  var isFunctionExpression = callback && callback.type === 'FunctionExpression'
  // we only care about `.map()` called on something else (with a function expression)
  if (!isMap || !isFunctionExpression || !isMemberExpression) return
  // replace the map with a push (which saves out) and then just make the map return itself
  callback.body.body.filter(isReturn).forEach(function (body) {
    // wrap the things being returned in a push statement
    this.replaceArrayWithLeftIndex(body.arguments)
    var push = b.callExpression(b.memberExpression(b.identifier('b'), b.identifier('push'), false), [body.argument])
    mapAsPush = b.expressionStatement(push)
  }, this)
  filter = ast.callee.object
  // completely remove the map (in favor of `filter()`)
  ast.type = filter.type
  ast.callee = filter.callee
  // replace the return inside the `filter()` with an if statement
  condition = filter.arguments[0].body.body[0].argument
  ifStatement = b.ifStatement(condition, b.blockStatement(mapAsPush))
  ast.arguments[0].body.body[0]ifStatement
  // replace the both the filter/map with a single if statement
  ast.type = ifStatement.type
  ast.test = ifStatement.test
  ast.alternate = ifStatement.alternate
  ast.consequent = ifStatement.consequent
  // carry on transforming this code (for example the test needs transforming)
  this.genericVisit(ast)
}
function visitForStatement (ast) {
  // if we've accidently made an expression statement the body
  // of the for Statement, undo that to prevent an extra ';'
  if (ast.body.body.length === 1 &&
      ast.body.body[0].type === 'ExpressionStatement') {
    ast.body.body[0] = ast.body.body[0].expression
  }
  this.genericVisit(ast)
}
function visitIfStatement (ast) {
  this.replaceLeftWithArrayIndex(ast.test)
  this.genericVisit(ast)
}
function replaceLeftWithArrayIndex (expression) {
  expression.left = b.memberExpression(this.array, this.identifier, true)
}
```

# example2

```js
//remove-consoles.input.js
module.exports = {
  sum: function sum (a, b) {
    console.log('calling sum with', arguments)
    return a + b
  },
  multiply: function multiply (a, b) {
    console.warn('calling multiply with', arguments)
    return a * b
  },
  divide: function divide (a, b) {
    console.error(`calling divide with ${ arguments }`)
    return a / b
  },
  average: function average (a, b) {
    console.log('calling average with ' + arguments)
    return divide(sum(a, b), 2)
  }
}
// goal
module.exports = {
  sum: function sum (a, b) {
    return a + b
  },
  multiply: function multiply (a, b) {
    return a * b
  },
  divide: function divide (a, b) {
    return a / b
  },
  average: function average (a, b) {
    return divide(sum(a, b), 2)
  }
}

// remove calls to the console
//remove-consoles.js
module.exports = function transform (fileInfo, api) {
  // To get there, we need to convert the source into an AST, find the consoles, remove them, and then convert the altered AST back into source.
  var j = api.jscodeshift
  // COLLECTION wrapping the root node
  var root = j(fileInfo.source)
  //  Unless you have some exceptional knowledge of
  // the Mozilla Parser API, you’ll probably need a tool
  //  to help understand what the AST looks like.

  // to match:
  // {
  //   "type": "CallExpression",
  //   "callee": {
  //     "type": "MemberExpression",
  //     "object": {
  //       "type": "Identifier",
  //       "name": "console"
  //     }
  //   }
  // }

  // details
  var more = {
    callee: {
      type: 'MemberExpression',
      object: { type: 'Identifier', name: 'console' },
    },
  }
  // queries -> COLLECTION
  var callExpressions = root.find(j.CallExpression, more)

  // collection object type has a remove method
  // that will do just that

  // process:
  callExpressions.remove()

  // generate:
  return root.toSource()

  // OR
  return j(fileInfo.source)
    .find(j.CallExpression, {
        callee: {
          type: 'MemberExpression',
          object: { type: 'Identifier', name: 'console' },
        },
      }
    )
    .remove()
    .toSource()

}
// execute with
// $> jscodeshift -t remove-consoles.js remove-consoles.input.js -d -p
// -d to not replace content of file
// -p to show preview in logged output
/*
  Processing 1 files...
  Spawning 1 workers...
  Running in dry mode, no files will be written!
  Sending 1 files to free worker...

  module.exports = {
    sum: function sum (a, b) {
      return a + b
    },
    multiply: function multiply (a, b) {
      return a * b
    },
    divide: function divide (a, b) {
      return a / b
    },
    average: function average (a, b) {
      return divide(sum(a, b), 2)
    }
  }

  All done.
  Results:
  0 errors
  0 unmodified
  0 skipped
  1 ok
  Time elapsed: 0.604seconds
*/
```

## example2

```js
// deprecated.input.js
import g from 'geometry'
import otherModule from 'otherModule'
// var geometry = require('geometry').g
// var otherModule = require('otherModule')
// var g = geometry
var radius = 20
var area = g.circleArea(radius)
console.log(area === Math.pow(g.getPi(), 2) * radius)
console.log(area === otherModule.circleArea(radius))

// AST explorer
// {
//   "type": "ImportDeclaration",
//   "specifiers": [
//     {
//       "type": "ImportDefaultSpecifier",
//       "local": {
//         "type": "Identifier",
//         "name": "g"
//       }
//     }
//   ],
//   "source": {
//     "type": "Literal",
//     "value": "geometry"
//   }
// }

//deprecated.js
module.exports = function codemod (fileInfo, api) {
  var j = api.jscodeshift
  var root = j(fileInfo.source)
  // COLLECTION
  var importDeclaration = root.find(j.ImportDeclaration, {
    source: {
      type: 'Literal',
      value: 'geometry',
    },
  })
  // COLLECTION
  var identifiers = importDeclaration.find(j.Identifier)
  // returns node-path at index n in that collection
  // COLLECTION - first node in collection
  var nodePath = identifiers.get(0)
  var localName = nodePath.node.name

  // find
  /*
    {
      "type": "MemberExpression",
      "object": {
        "name": "geometry"
      },
      "property": {
        "name": "circleArea"
      }
    }
  */
  return root.find(j.MemberExpression, {
    object: {
      name: localName,
    },
    property: {
      name: "circleArea",
    },
  }).replaceWith(function (nodePath) {
    var node = NodePath.node
    node.property.name = 'getCircleArea'
    return node
  }).toSource()
}
// run:
// jscodeshift -t ./deprecated.js ./deprecated.input.js -d -p
/*
  Processing 1 files...
  Spawning 1 workers...
  Running in dry mode, no files will be written!
  Sending 1 files to free worker...
  All done.
  Results:
  0 errors
  1 unmodified
  0 skipped
  0 ok
  Time elapsed: 0.892seconds
*/
```

## example3

```js
// source
//signature-change.input.js

import car from 'car'
var suv = car.factory('white', 'Kia', 'Sorento', 2010, 50000, null, true)
var truck = car.factory('silver', 'Toyota', 'Tacoma', 2006, 100000, true, true)
// target
var suv = car.factory({
  color: 'white',
  make: 'Kia',
  model: 'Sorento',
  year: 2010,
  miles: 50000,
  bedliner: null,
  alarm: true,
})
var truck = car.factory({
  color: 'silver',
  make: 'Toyota',
  model: 'Tacoma',
  year: 2006,
  miles: 100000,
  bedliner: true,
  alarm: true,
})
// transform
//signature-change.js
module.exports = function transform (fileInfo, api) {
  var j = api.jscodeshift
  var root = j(fileInfo.source)
  // 1. Find the local name for the imported module
  // => find declaration for "car" import
  var importDeclaration = root.find(j.ImportDeclaration, {
    source: {
      type: 'Literal',
      value: 'car',
    },
  })
  // => get the local name for the imported module
  var localName = importDeclaration.find(j.Identifier)
    .get(0)
    .node.name
  // 2. Find all call sites to the .factory method
  // => find where `.factory` is being called
  var callSites = root.find(j.CallExpression, {
    callee: {
      type: 'MemberExpression',
      object: {
        name: localName,
      },
      property: {
        name: 'factory',
      },
    }
  })
  callSites.replaceWith(nodePath => {
    var node = nodePath.node
    // Read all arguments being passed in

    // Replace that call with a single argument which contains an object with the original values
    ;`node.arguments = [{ foo: 'bar' }]`
    // => ERR signature-change.input.js Transformation error
    // Error: {foo: bar} does not match type Printable
    // Turns out we can’t just jam plain objects into our AST nodes. Instead, we need to use builders to create proper nodes.

    // BUILDERS rigidly check that the different types of nodes are created correctly
    // All of the available AST node types are defined in the deffolder of the https://github.com/benjamn/ast-types/tree/master/def
    // mostly in `core.js` as came-cased, not pascal-cased version
    // https://github.com/benjamn/ast-types/blob/master/lib/types.js#L609
    // def("ObjectExpression")
    //   .bases("Expression")
    //   .build("properties")
    //   .field("properties", [def("Property")])
    // def("Property")
    //   .bases("Node")
    //   .build("kind", "key", "value")
    //   .field("kind", or("init", "get", "set"))
    //   .field("key", or(def("Literal"), def("Identifier")))
    //   .field("value", def("Expression"))

    var argKeys = [
      'color',
      'make',
      'model',
      'year',
      'miles',
      'bedliner',
      'alarm',
    ]
    // { foo: ‘bar’ }
    var argumentsAsObject = j.objectExpression(
      node.arguments.map(function (arg, i) {
        return j.property(
          'init',
          j.identifier(argKeys[i]),
          j.literal(arg.value)
        )
      })
    )
    node.arguments = [argumentsAsObject]
    return node
  })
  // specify print options for recast
  var opts = { quote:' single', trailingComma: true }
  return root.toSource(opts)
}
// execute
// jscodeshift -t signature-change.js signature-change.input.js -d -p

```
