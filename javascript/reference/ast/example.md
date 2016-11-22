```js
var esprima = require('esprima')

var code = 'function jqcon() {}'

/*
An Object (Tree) That Represents The Structure Of Your Code
It can be used to analyze code
Does Not Store Everything - It is abstract
e.g. Does not store that a JS function starts/ends with open/close curly brackets

SpiderMonkey Parse API is recognized by the community as a standard for structured JS representation
*/
var ast = esprima.parse(code) // alternative: acorn
var ast = {
  "type": "FunctionDeclaration",
  "id": {
    "type": "Identifier",
    "name": "jqcon"
  },
  "params": [],
  "body": {
    "type": "BlockStatement",
    "body": []
  },
  "expression": false
}
// AST Traversal and Update Methods
// Adheres To The Mozilla SpiderMonkey Parser API
// => traversing ASTs is tricky since not all AST child nodes have a unified interface
// e.g. VariableDeclaration has nested children nodes within a declarations property,
//      while an IfStatement has nested children within a consequent property
var estraverse = require('estraverse')
estraverse.traverse(ast, {
  enter: enter,
  leave: function(node, parent) {}
})

function enter (node, parent) {
  if(node.type === 'Identifier' && node.name === 'jqcon') {
    // Changes the 'jqcon' function name to 'jqcon_is_awesome'
    node.name = node.name + '_is_awesome';  
  }
} // or Estraverse.replace()
estraverse.replace(ast, {
  enter: function (node, parent) {
    if(node.type === 'Identifier' && node.name === 'jqcon') {
      // Changes the 'jqcon' function name to 'jqcon_is_awesome'
      return { 'type': 'Identifier', 'name': 'jqcon_is_awesome' };
    }
  }
})
// ALTERNATIVE - https://github.com/estools/esquery

// generate code with escodegen
// Adheres To The Mozilla SpiderMonkey Parser API
var escodegen = require('escodegen')
// Returns a string of code represented by the AST
var regenerated_code = escodegen.generate(ast)
// But Couldn't I Write an AST Code Generator Myself?
// You could. I'll see you next year, when you're finished
document.body.innerHTML = code + '<br>' + regenerated_code
```
