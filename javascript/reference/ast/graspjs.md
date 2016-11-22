# graspjs
* http://www.graspjs.com/blog/2014/01/07/refactoring-javascript-with-grasp

```bash
  grasp '#x' # variable values
  grasp 'literal' # literals
  grasp 'str' # strings
  grasp '#/^extra/' # regexp
  grasp 'func[id=#getValue] return' # return
```

# example1
For instance changing all calls to "calc" from taking two arguments to taking one object literal instead (eg. `calc(1, 2)` to `calc({from: 1, to: 2})`, is as easy as:

```bash
$ grasp -r -e 'calc($a, $b)' -R 'calc({from: {{a}}, to: {{b}}})' file.js
```

(Where the two arguments to "calc" could be any arbitrary nested expressions, which could not be safely handled by a regular expression)

# example2
```js
  function multiply (x, y) {
    y = y || x // square if no second argument
    return x * y
  }
  multiply(5, 5) // => 25
  multiply(5) // => 5
  multiply(5, 0) // => 25, but should be 0
```
```bash
grasp -e '$x = $x || $d' -R 'if ({{x}}===undefined){ {{x}}={{d}} }'
# matches more complex expressions
# e.g. this.x = this.x || a(b() + 1).bind(this)
```
```js
```

```js
require('grasp-equery')
.query('if ($cond) return $yes; else return $no;', ast)

require('grasp-squery')
.query('if[then=return][else=retyrn]', ast)
