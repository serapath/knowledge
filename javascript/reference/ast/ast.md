# code examples
```js
var sourcecode1 = 'n * n;'
var sourcecode2 = 'if (someCondition) {}'
var sourcecode3 = `
  // Life, Universe, and Everything
  var answer = 6 * 7
`
var sourcecode4 = 'console.log("Hello, World!")'
```

# pipeline
```js
  var sourcecode = sourcecode1 || sourcecode2 || sourcecode3
  // source -esprima-> AST -escodegen-> source
  var parse = require('esprima').parse // parse + tokenize
  // var parse = require('acorn').parse
  // parse(code, { ecmaVersion: 6 })
  var generate = require('escodegen').generate
  function parse (sourcecode) {
    // Lexical Analysis
      // A tokenizer just splits text into smaller units such as words.
      // Tokenizing into letters, syllables, sentences etc. is also possible.
    var tokens = tokenize(sourcecode)
      // A lexer does the same plus attachs extra information to each token.
      // If we tokenize into words, a lexer would attach tags like number, word, punctuation etc.
    // Syntactic Analysis
      // A parser usually uses the output of a lexer and constucts a parse tree.
    var ast = parseASTfrom(tokens)
    return ast
  }
  function generate (AST) {
    return generateCodeFrom(AST)
  }
  var ast = parse(sourcecode)
  var code = generate(ast)
```

## tokenization
e.g. https://github.com/thejameskyle/babel-handbook/blob/master/translations/en/plugin-handbook.md
```js
var tokens3 = tokenize(sourcecode3)
var tokens3 = [
  { type: 'Keyword',    value: 'var' },
  { type: 'Identifier', value: 'answer' },
  { type: 'Punctuator', value: '=' },
  { type: 'Numeric',    value: '6' },
  { type: 'Punctuator', value: '*' },
  { type: 'Punctuator', value: '7' }
]
var tokens1 = tokenize(sourcecode1)
var tokens1 = [
  {type:{/*...*/},value:"n",start:0,end:1,loc:{/*...*/}},
  {type:{/*...*/},value:"*",start:2,end:3,loc:{/*...*/}},
  {type:{/*...*/},value:"n",start:4,end:5,loc:{/*...*/}},
  /*...*/
]
// tokens1[x].type => describes the token
{
  type: {
    label: 'name',
    keyword: undefined,
    beforeExpr: false,
    startsExpr: true,
    rightAssociative: false,
    isLoop: false,
    isAssign: false,
    prefix: false,
    postfix: false,
    binop: null,
    updateContext: null
  },
  ...
}
```

## AST (abstract syntax tree)
* is an Intermediate Representations (IR)
* each node is tagged with its type(s)
* nodes have no state or knowledge of context
* disallows construction of invalid programs
* similar syntactic productions are meaningfully grouped

```js
var ast1 = parser(sourcecode1) // => parsing(tokenize(sourcecode1))
var ast1 = {
  type: "FunctionDeclaration",
  id: {
    type: "Identifier",
    name: "square"
  },
  params: [{
    type: "Identifier",
    name: "n"
  }],
  body: {
    type: "BlockStatement",
    body: [{
      type: "ReturnStatement",
      argument: {
        type: "BinaryExpression",
        operator: "*",
        left: {
          type: "Identifier",
          name: "n"
        },
        right: {
          type: "Identifier",
          name: "n"
        }
      }
    }]
  }
}
// --------------------------------------------------------
var ast2 = parser(sourcecode2) // => parsing(tokenize(sourcecode2))
var ast2 = {
  type: 'IfStatement',
  test: {
    type: 'Identifier',
    name: 'someCondition'
  },
  consequent: {
    type: 'BlockStatement',
    body: []
  }
  alternate: null
}
/* Example AST:
- FunctionDeclaration
- Identifier (id)
- Identifier (params[0])
- BlockStatement (body)
- ReturnStatement (body)
- BinaryExpression (argument)
- Identifier (left)
- Identifier (right)
*/
// --------------------------------------------------------
var ast3 = parser(sourcecode3)
var ast3 = {
  type: 'Program',
  body: [{
    type: 'VariableDeclaration',
    declarations: [{
        type: 'VariableDeclarator',
        id: { type: 'Identifier', name: 'answer' },
        init: {
          type: 'BinaryExpression',
          operator: '*',
          left: { type: 'Literal', value: 6 },
          right: { type: 'Literal', value: 7}
        }
    }],
    kind: 'var'
  }]
}
// --------------------------------------------------------
var acorn = require('acorn')
acorn.parse('var answer = 6 * 7', { locations: true })
// In each node
loc: {
  start: {
    line: 2,
    column: 0
  },
  end: {
    line: 2,
    column: 19
  }
}
// --------------------------------------------------------
var esprima = require('esprima')
var ast4 = esprima.parse(sourcecode4, { sourceType: 'script' })
// =>
{
  type: 'Program',
  body: [
    type: 'ExpressionStatement',
    expression: {
      type: 'CallExpression',
      callee: {
        type: 'MemberExpression',
        computed: false,
        object: {
          type: 'Identifier',
          name: 'console'
        },
        property: {
          type: 'Identifier',
          name: 'log'
        }
      },
      arguments: [{
        type: 'Literal',
        value: 'Hello, World!',
        raw: 'Hello, World!'
      }]
    }
  ],
  sourceType: 'script'
}
```

### nodes
1. Dave Herman (Mozilla)
2. SpideMonkey
3. Specified in the `estree` spec

## e.g.


### node types
* (File)
* (Program)
* SwitchCase(1)
* Property(1)
* Literal(1)
* Identifier(1), used for names of:
  * variables, functions, methods, object keys, etc...
* Declaration(3)
* Expression(14)
  * FunctionExpression
  * MemberExpression
  * CallExpression
  * NewExpression
  * ConditionalExpression
  * LogicalExpression
  * UpdateExpression
  * AssignmentExpression
  * BinaryExpression
  * UnaryExpression
  * SequenceExpression
  * ObjectExpression
  * ArrayExpression
  * ThisExpression
  * NEW TemplateLiteral
    * NEW TemplateElement(s), region between `${}`
* Statement(18)
  * DebuggerStatement
  * ForInStatement
  * DoWhileStatement
  * WhileStatement
  * CatchClause
  * TryStatement
  * ThrowStatement
  * ReturnStatement
  * SwitchStatement
  * WithStatement
  * ContinueStatement
  * BreakStatement
  * LabeledStatement
  * IfStatement
  * ExpressionStatement
  * BlockStatement
  * EmptyStatement

# attributes
  * raw and cooked only differ when there are escape characters

## traverse (walk & visit)
* When transforming, start with the `AST` you want and then work backwards.
* Often this means pasting code using the **esprima online visualization tool**
* Or outputting trees into JS files and manually diffing them
* sometimes it helps to do
```js
var source = recast.prettyPrint(ast, { tabWidth: 2 }).code
var source = context.getSource(node) // in ESlint
```
### debugging ast's
```js
var inspect = require('util').inspect
inspect(ast, { depth: null })
```

### state
State is the enemy of AST transformation.
State will bite you over and over again and your assumptions about state will almost always be proven wrong by some syntax that you didn't consider.
The better way to deal with this is recursion. So let's make like a Christopher Nolan film and put a visitor inside of a visitor.

### visitor pattern
"visitor" pattern is common when working with AST
it means while traversing the AST, something only
cares for certain kinds of nodes
=> ~match a certain selector/pattern...

```js
var visitor = {
  'Identifier': {
    enter: function visit (node) { /*
      traversing down each branch of the tree
    */ },
    exit: function visit (node) { /*
      traversing back up each branch of the tree
      after hitting dead ends
    */ }
  }
}
// example walk:
// Enter FunctionDeclaration
// Enter Identifier (id)
// Hit dead end
// Exit Identifier (id)
// Enter Identifier (params[0])
// Hit dead end
// Exit Identifier (params[0])
// Enter BlockStatement (body)
// Enter ReturnStatement (body)
// Enter BinaryExpression (argument)
// Enter Identifier (left)
// Hit dead end
// Exit Identifier (left)
// Enter Identifier (right)
// Hit dead end
// Exit Identifier (right)
// Exit BinaryExpression (argument)
// Exit ReturnStatement (body)
// Exit BlockStatement (body)
// Exit FunctionDeclaration

//-----------------------------------
// example
var ast = diff.before
esrecurse.visit(ast, {
  // export function a () {}
  ExportNamedDeclaration: function (node) {

  },
  // function a () {}
  FunctionDeclaration: function (node) {

  }
})
// inspecting function declarations
function inspectFunction (node, visibility) {
  return {
    name: node.id.name, // 'buildHouse'
    params: node.params.map(param=>param.name),
    visibility: visibility || 'private',
    outputType: getOutputTupe(node)
  }
}

//-----------------------------------

var updateParamNameVisitor = {
  Identifier (path) {
    if (path.node.name === this.paramName) {
      path.node.name = "x"
    }
  }
}
var MyVisitor = {
  FunctionDeclaration (path) {
    var param = path.node.params[0]
    var paramName = param.name
    param.name = "x"
    path.traverse(updateParamNameVisitor, { paramName })
  }
}
```

```js
var counter = 0, map = {}

result = estraverse.replace(tree, {
  enter: function (node) {
    if (node.type === 'Identifier' && node.name[0] === '_')
      node.name = map[node.name]
        || (map[node.name] = '$' + counter++)
  }
})
```
```js
var Renamer = recast.Visitor.extend({
  init: function () {
    this.counter = 0
    this.map = {}
  },
  getId: function (name) {
    return this.map[name]
      || (this.map[name] = '$' + this.counter++)
  },
  visitIdentifier: function (node) {
    if (node.name[0] === '_')
      node.name = this.getId(node.name)
  }
})
```

### scopes
When writing a transform, we want to be wary of scope. We need to make sure we don't break existing code while modifying different parts of it
* We may want to add new references and make sure they don't collide with existing ones
* maybe we just want to find where a variable is referenced
* We want to be able to track these references within a given scope

```js
/*
var global = "I am in the global scope"
function scopeOne() {
  var one = "I am in the scope created by `scopeOne()`"
  function scopeTwo() {
    one = "reference update in `scopeOne` from within `scopeTwo`"
    var two = "I am in the scope created by `scopeTwo()`"
    var global = "new `global`, leaving global reference alone."
  }
}
*/
var scope = {
  path: path,
  block: path.node,
  parentBlock: path.parent,
  parent: parentScope,
  bindings: [/*...*/]
}
//When you create a "new scope" you do so by giving it
// * a path and
// * a parent scope.
// Then during the traversal process it collects all the references ("bindings") within that scope.

// => Once that's done, there's all sorts of methods you can use on scopes
```

### bindings (References all belonging to a particular scope)
```js
function scopeOnce() {
  var ref = "This is a binding";

  ref; // This is a reference to a binding

  function scopeTwo() {
    ref; // This is a reference to a binding from a lower scope
  }
}
// Example
var binding = {
  identifier: node,
  scope: scope,
  path: path,
  kind: 'var',

  referenced: true,
  references: 3,
  referencePaths: [path, path, path],

  constant: false,
  constantViolations: [path]
}
// With this information you can find all the references to a binding, see what type of binding it is (parameter, declaration, etc.), lookup what scope it belongs to, or get a copy of its identifier. You can even tell if it's constant and if not, see what paths are causing it to be non-constant.
```

### paths (in babel)
A Path is an object representing the link between two nodes

```js
// When you have a visitor that has a Identifier() method, you're actually visiting the path instead of the node. This way you are mostly working with the reactive representation of a node instead of the node itself.
var parent = {
  type: "FunctionDeclaration",
  id: {
    type: "Identifier",
    name: "square"
  },
  /*...*/
}
var child = {
  "parent": {
    "type": "FunctionDeclaration",
    "id": {...},
    /* ... */
  },
  "node": {
    "type": "Identifier",
    "name": "square"
  }
}

var metadata =
{
  "parent": {...},
  "node": {...},
  "hub": {...},
  "contexts": [],
  "data": {},
  "shouldSkip": false,
  "shouldStop": false,
  "removed": false,
  "state": null,
  "opts": null,
  "skipKeys": null,
  "parentPath": null,
  "context": null,
  "container": null,
  "listKey": null,
  "inList": false,
  "parentKey": null,
  "key": null,
  "scope": null,
  "type": null,
  "typeAnnotation": null
}
// As well as tons and tons of methods related to adding, updating, moving, and removing nodes, but we'll get into those later.
```


## generate
```js
var output = escodegen.generate(ast, {
  sourceMap: true,
  sourceMapWithCode: true
})
fs.writeFileSync('out.js', output.code)
fs.writeFileSync('out.js.map', output.map.toString())
```
