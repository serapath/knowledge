# jscodeshift (codemod toolkit)
* https://github.com/facebook/jscodeshift/pull/119
* example:
https://www.toptal.com/javascript/write-code-to-rewrite-your-code
https://vramana.github.io/blog/2015/12/21/codemod-tutorial/

The interface that `jscodeshift` provides is a wrapper around `recast` and `ast-types` packages.
* recast handles the conversion from source to AST and back
* ast-types handles the low-level interaction with the AST nodes.

Think of codemods as a scripted find and replace functionality that can read and write code.


JSCodeshift: AST -> AST based codemod tool


## usage
https://github.com/facebook/jscodeshift/wiki/jscodeshift-Documentation

```bash
npm install -g jscodeshift
jscodeshift -t some-transform.js input-file.js -d -p
# transforms input-file.js without altering it
```

**codemod**
* Code that is written with the sole intent of transforming other code. An example would be a piece of code that takes a normal function, and rewrites it to be an arrow function.
* Distributing groups of files to different processes and running them in parallel is something jscodeshift excels at, allowing you to run complex transformations across a huge codebase in seconds.

**AST**
* A tree representation of a piece of source code. When working with JavaScript tools that allow you to inspect/modify an AST, this will most commonly be exposed to you in the form of nested JavaScript objects.
* consists of nested `node`s
* nodes can be described via path
* a collection of path's can describe the AST

**node**
	* The representation of a single construct in an AST
	* example 'FunctionExpression'
	* nodes often have other nodes nested in them
	* simple objects without any methods
	* do not contain information about parents or scope

**path** (=node-path)
	* An object that wraps a single node
	* exposes API to modify/inspect node information
	* provided by `ast-types`
	* a way to trasverse the AST
	* contain information about parents and scope of nodes
	* wrapped node is accessible via `path.node`
	* has methods to change the underlying node
	* scope & relations info of node, not about node itself

**collection**
	* group of zero or more node-path objects
	* exposes helpers to transform all contained paths, or traverse them further
	* jQuery style
	* returned when jscodeshift queries the AST
	*

helper information:  
https://www.npmjs.com/package/jscodeshift-helper

# example
```js
// transform function
module.exports = function codeModName (file, api) {
	// file: file on which to perform the transform
	// api: jscodeshift API
	var j = api.jscodeshift // transformer
	var AST = j(file.source)
	// NODE TYPES:
	// j.Identifier
	// j.CallExpression
	// ...
	// BUILD: (has helpful error msg's)
	// j.identifier(name)
	// j.callExpression(...)

	AST.find(j.CallExpression, {
		callee: {
			type: 'Identifier',
			name: 'foo'
		}
	}).forEach(function (p) {
		// p is a path in AST to a node that matches
		// p.node references the AST node directly
		// p.parent references that nodes parent scope
		//
		p.get('callee').replace(j.identifier('bar'))
	})

	return AST.toSource({
		useTabs: true,
		quote: 'single'
	})
}
```

```js
module.exports = function transformer(file, api) {
    var j = api.jscodeshift
    var root = j(file.source)
    var TypeDefinition = j.CallExpression
    // (Optional) A filter can be either a function that returns true/false, or an object
    // to partially match against all nodes found for the specified TypeDefinition
    var filter = {
        callee: {
            object: {
                name: 'console'
            },
            property: {
                name: 'log'
            }
        }
    }
    // collection of AST path's
    var consoleLogCalls = root.find(TypeDefinition, filter)
    consoleLogCalls.forEach(p => {
      /*
      Although collections and paths expose many helpful APIs to declaratively modify the AST, we can always grab a reference to the node wrapped in a path, and mutate it directly
      */
        p.node.callee.property.name = 'warn'
    })
    return root.toSource() // calls recast() directly
}
```
# helper methods
* https://github.com/facebook/jscodeshift/blob/master/src/collections/Node.js
* https://github.com/facebook/jscodeshift/blob/master/src/Collection.js
* console.log a collection or node in AST Explorer, and use your browser’s DevTools to inspect the prototype chain
* explore `api.stats(‘some message here’)` to log information. When you run the jscodeshift binary with the dry command line argument, it will inform you of how many times the stats function was called with each unique value.

# jscodeshift (one of code migration)
jscodeshift is a toolkit for running codemods
over multiple JS files. It provides:

* A runner, which executes the provided transform for each file passed to it. It also outputs a smmary of how many files have (not) been implemented
* A wrapper around `recast`, providing a different API. Recast is an AST-to-AST transform tool and also tries to preserve the style of original code as much as possible

```js
module.exports = function (fileInfo, api) {
  return api.jscodeshift(fileInfo.source)
    // .findVariableDeclarators('foo')
    // .renameTo('bar')
    // .toSource()
}
// =>
// turns
var foo = 'asdf'
// into
var bar = 'asdf'
```
```js
function transformer (file, api) {
  var j = api.codeshift
  var {expression, statement, statements} = j.template
  return j(file.source)
    .find(j.Identifier)
    .replaceWith(p=>j.identifier(p.node.name.split('')
      .reverse().join(''))
    )
    .toSource()
}
// j is a function to parse sourcecode
// it also has constants as names of AST

```


# codemod
change complex systems incrementally,
with a clearly envisioned end-state in mind

## jscodeshift
```js
// code
merge(firstObject, {b:1}, secondObject)
merge(jsconf,food,talkes)
thisIsNotMerge()

// codemod
const flatten = a => Array.isArray(a) ? [].concat(...a.map(flatten)) : a
module.exports = (file, api) => {
	const j = api.jscodeshift
	const root = j(file.source)

	const update = p => j(p).replaceWith(
		j.objectExpression(flatten(p.value.arguments.map(
          arg=> arg.type === 'ObjectExpression' ? arg.properties :
            j.spreadProperty(arg)
        )))
	)
	// pattern matching
	root.find(j.CallExpression, {callee:{name:'merge'}})
	  .forEach(update)

	return root.toSource()
}
```
```js
// old code
var sourcecode = 'Yo ' + name + '! How are you doing?'  
// target
var dreamcode = `Yo ${name}! How are you doing?`

// how to start? maybe try:
// old code
var sourcecode = a + b
// target
var dreamcode = `${a}${b}`

// codemod
function flatten (node) {
  var isBE = node.type === 'BinaryExpression'
  var isPLUS = node.operator === '+'
  return isBE && isPLUS ?
    [...flatten(node.left), ...flatten(node.right)]
    : [ node ]
}
function isStringNode (node) {
  var isLiteral = node.type === 'Literal'
  var isString = typeof node.value === 'string'
  return isLiteral && isString
}
module.exports = function codemod (file, api) {
  var j = api.jscodeshift
  // var expression = j.template.expression
	// var statement  = j.template.statement
	// var statements = j.template.statements
	function convertToTemplateString (p) {
    var nodes = flatten(p.node)
    if (!nodes.some(isStringNode)) return p.node
    function map (list, fn, accumulator) {
			while (list.length) fn(list.shift(), accumulator)
			return accumulator
		}
		function build (x, o) {
			if (x.type === 'Literal') {
				o.temp += x.value
			} else {
				// @TODO: handle (un)escaping
				var te = { cooked: o.temp, raw: o.temp }
				o.temp = ''
				o.quasis.push(j.templateElement(te,false))
				o.exps.push(x)
			}
		}
		var o = map(nodes, build, {quasis:[],exps:[],temp:''})
		var s = o.temp
		var exps = o.exps
		var quasis = o.quasis
		// @TODO: handle (un)escaping
		quasis.push(j.templateElement({cooked:s,raw:s},true))
    return j.templateLiteral(quasis, exps)
	}

	return j(file.source)
    // pattern matching
    .find(j.BinaryExpression, { operator: '+' })
	// transform
    .replaceWith(convertToTemplateString)
	// generate
    .toSource()
}
```
