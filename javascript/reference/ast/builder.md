# builder (constructor)

```js
var b = require('ast-types').builders

b.variableDeclaration('var', [
  b.variableDeclarator(
    b.identifier('answer'),
    b.binaryExpression(
      '*',
      b.literal(6),
      b.literal(7)
    )
  )
])

var estemplate = requie('estemplate')
estamplate('var <%= varName %> = 6 * 7', {
  varName: { type: 'Identifier', name: 'answer' }
})
```
