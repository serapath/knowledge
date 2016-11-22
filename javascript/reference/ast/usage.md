# Usage


# Examples

## scope analysis & refactoring
* resolve identifier scoping according to the language specification. One of them is
* https://github.com/Constellation/escope
* https://github.com/ariya/esrefactor
  * uses escope after getting AST from esprima
  * to locate occurences of a given identifier



1. Walking the syntax tree of a JavaScript code is often the first step towards building a specialized static analyzer

2. when the analysis involves variables and functions within the code, an additional scope analysis is necessary
(it permits a more thorough examination of those variables and functions, including to check if some identifiers accidentally leak to the global scope)

Walking the syntax tree, we keep track of variable declaration so that we can cross-reference it within any **assignment**

(of course, we must take into account complications such as variable hoisting and function scope)

`npm install leaky`
```json
{
  "scripts": {
    "test": "leaky mycode.js && node run-tests.js"
  }
}
```

`npm install unused`
```js
// a flag allows `unused` to ignore placeholder params
[4, 5, 6].forEach(function (value, index) {
  console.log(index)
})
```

Integrate them as
* test procudures
* or pre-commit hooks
* or editor behaviors


## Identifier Highlighting
```js
// is far from trivial
// but helper libraries exist which help resolve identifiers
var escope = require('escope')
  // scope analysis
var esrefactor = require('esrefactor')
  // use scope analysis to locate identifier occurances

// example http://esprima.org/demo/highlight.html

```

## Rename refactoring
```js
// next step instead of just highlighting
// monitor possible change to code
// => appropriately **rename** other identical references

// example http://esprima.org/demo/rename.html

// uses escope, so only works on identifiers with the same name in scope and not on those which are for example shadowed

var esrefactor = require('esrefactor')
var ctx = new esrefactor.Context('var x; x = 42')
var id = ctx.identify(4) // locate identifier
var code = ctx.rename(id, 'answer')
// var answer; answer = 42

```

## Scope Analysis - Walking the syntax tree
first step towards building a **static analyzer**
* if variables or functions are involved often a **scope analysis** is necessary
* allows for example to check whether identifiers
leak to the global scope
```js
var code = `
  var foo
  function test (t) {
    var foo = 5
    t = 3
    tt = 'tt'
    var x = 123
    return function () {
      var meh = foo
      bla = 'asdf'
      console.log(x)
    }
  }
`
var esprima = require('esprima')
var escope = require('escope')
// e.g. implicit declaration at the global scope.
function find_leak(code) {
  var AST = esprima.parse(code, { loc: true })
  var leaks = []
  var globalScope = escope.analyze(AST).scopes[0]
  globalScope.implicit.variables.forEach(function (v) {
    var id = v.identifiers[0]
    leaks.push({
      name: id.name,
      line: id.loc.start.line
    })
  })
  return leaks
}
```
