# lint

## example


# rules
* http://eslint.org/docs/rules/handle-callback-err.html
* block-scoped-var
* brace-style
* camelcase
* complexity
* consistent-return
* consistent-this
* curly
* default-case
* dot-notation
* eol-last
* eqeqeq
* func-names
* func-style
* ...
* no-jquery
* https://github.com/eslint/eslint/blob/master/lib/rules/no-console.js
* https://github.com/eslint/eslint/blob/master/lib/rules/no-loop-func.js
* https://github.com/eslint/eslint/blob/master/lib/rules/max-params.js
* indent
* no-extend-native
* no-next-next
* security
* internationalization
* ...

```js
'use strict'
// Disallow sparse arrays
module.exports = function (context) {
  return {
    'ArrayExpression': function (node) {
      if (node.elements.indexOf(null) > -1) {
        context.report(node, 'Unexprected comma in middle of array.')
      }
    }
  }
}
```

# JSHint
* mixes rule engine with parser
