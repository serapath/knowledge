# aster
+ aster (asterjs.github.io/aster) (code builder)
  + => faster than gulp and/or browserify
npm keyword: aster-plugin

* aster-concat
* aster-equery
* aster-squery
* aster-rename-ids
* aster-traverse
* aster-uglify

* aster
  * src
  * watch
  * dest
  * runner
* aster-parse
  * js
  * esnext
  * jsx
  * coffee
* aster-changed

```js
aster.watch(['index.js'])
.throttle(500)
.map(changed(function (src) {
  src.map(equery({
    'if ($cond) return $expr1; else return $expr2;':
      'return <%= cond %> <%= expr1 %> : <%= expr2 %>'
  }))
}))
.map(concat('built.js'))
.map(umd({exports: 'superLib'}))
.map(aster.dest('dist', {sourceMap: true}))
.subscribe(aster.runner)
```
