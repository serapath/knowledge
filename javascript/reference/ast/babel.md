# api
babel is a compiler for javascript
  * bundlers
  * ASTs
  * parsers
  * static analysis
  * language evolution

* https://github.com/thejameskyle/babel-handbook/blob/master/translations/en/plugin-handbook.md#api


* `babylon` (parser)

* `babel-traverse`

* `babel-types`
  * definitions
  * builders
  * validators
  * converters

* `babel-generator`

* `babel-template`

# babel plugin
uses "visitor pattern"

```js
module.exports = function (babel) {
  var visitor = {
    'VariableDeclaration': function (path) {
      if (path.node.kind === 'const') {
        path.node.kind = 'var'
      }
    }
  }
  return { visitor: visitor }
}

// ...

module.exports = function (Babel) {
  return new Babel.Plugin('plugin-example', {
    visitor: {
      'FunctionDeclaration': swapWithExpression
    }
  })
}

function swapWithExpression (node, parent) {
  // act on a matched node
  node.type = "FunctionExpression"
  node.id = null
  return Babel.types.variableDeclaration('var', [
    Babel.types.variableDeclarator(id, node)
  ])
}
// =>
// turns
function help () {}
// into
var help = function () {}
```


# Transformation Operations

* visiting
  * get the path of sub-node
  * check if a node is a certain type
  * check if an identifier is referenced

* manipulation
  * replacing a node
  * replacing a node with multiple nodes
  * replacing a node with a source string
  * inserting a sibling node
  * removing a node
  * replacing a parent
  * removing a parent

* scope
  * checking if a local variable is bound
  * generating a UID
  * pushing a variable declaration to a parent scope
  * rename a binding and its references

# plugin options
* ...

# building nodes
* ...

# best practices
* avoid traversing the AST as much as possible

Traversing the AST is expensive, and it's easy to accidentally traverse the AST more than necessary. This could be thousands if not tens of thousands of extra operations.

Babel optimizes this as much as possible, merging visitors together if it can in order to do everything in a single traversal.

* merge visitors whenever possible

* do not traverse when manual lookup will do

* optimizing nested visitors

* being aware of nested structures
