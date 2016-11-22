# meta coding
The most well-read documentation for how to engineer your app is the current codebase
If you change your code, you can change the future
* https://en.wikipedia.org/wiki/Abstract_semantic_graph
* https://en.wikipedia.org/wiki/Abstract_syntax_tree
* https://en.wikipedia.org/wiki/Parse_tree


# pipeline (transpiling)
it's all simple tree operations, use: https://astexplorer.net/

input string -> babylon parser -> AST -> transformer[s] -> AST -> babel-generator -> output string
```js
// guarantees (for acorn and esprima)
parse(generate(parse(code))) === parse(code)
```

0. sourcing
  * process a `source` once or use a file watcher to process a `source` over and over again.
  * A `source` is being processed by the `pipeline`
1. scan & parse (e.g. esprima, acorn) => Convert the source into AST
  1. Buffer up the stream of source code
  2. Read your raw JavaScript source
    * syntax traversal (e.g. estraverse)
      * with scope (e.g. escope)
  3. Parse out every single thing that is happening
  4. Return an AST that represents your code
2. Traverse/Transform the AST
  1. find node's to operate on (pattern matching)
  2. FLAGGING
    * add location information to nodes
    * tools add more flags
  3. create new nodes and/or information to be inserted
    * `ast-types` (pattern matching)
    ```js
      // CREATE - FIND
      def('Expression').bases('Node', "Pattern")
      def('ObjectExpression').bases('Expression')
        .build('properties').field('properties', [def('Property')])
      def('Property').bases('Node')
        .build('kind', 'key', 'value')
        .field('kind', or('init', 'get', 'set'))
        .field('key', or(def('Literal'), def('Identifier')))
        .field('value', def('Expression'))
      // type out javascript literal in jscodeshift
      // e.g. convert for into while loop
      // CREATE - UPATE
      statements`
        ${node.init}
        while(${node.test}) {
          ${node.body.body}
          ${node.update}
        }
      `
    ```
  4. update the AST by replacing and fitting new node in
    * optimizing & minifying (e.g. esmangle)
      * uses fixed-point iteration ove AST
      * preserves location information
    * code coverage (e.g. istanbul)
      * augments code for tracing execution
3. print/unparse/generate it into code (e.g. escodegen)
  * Re-generate and output the transformed source code
  * with proper formatting that looks like humans wrote it
  * can generate SourceMap too
  ```js
    // how to get code back into javascript that has
    // the same formatting as before?
    recast.print(ast)
    // why not use babel? because babel creates
    // code that doesnt try to look human readable
  ```

# Tools
+ ~~SpiderMonkey: Reflect.parse~~
+ Acorn (newer than esprima, 2x speed of esprima)
* ~~babylon~~
+ ~~UglifyJS (has custom format)~~
+ ~~Traceur (has custom format)~~
+ https://github.com/ajaxorg/treehugger
  + http://ajaxorg.github.io/treehugger/test.html
+ ...
* ~~espree~~
+ Esprima (browserify, eslint, ...)
  + estraverse (traverse/transform)
  + recast (traverse/transform)
  + grasp-equery
  + grasp-squery
  + ast-types
  + estemplate
  + escodegen (generate)
  + esvalidate
  + eslint
    1. builds AST
    2. traverses nodes
    3. checks if it has tests for nodes & if so, runs them
    ```js
      // if (someCondition) {
      //   if (someOtherContdition) {}
      // }
      // =>
      // eslint commonjs npm node modules
      module.exports = function test (context) {
        return {
          // keys are node.type's, e.g. IfStatement
          IfStatement: function (node) {
            // e.g.
            var ancestors = context.getAncestors()
            var parent = ancestors.pop()
            var grandparent = ancestors.pop()
            if (parent.type === 'IfStatement' ||
              (parent.type === 'BlockStatement' &&
               grandparent.type === 'IfStatement'
            )) {
              context.report(node, 'Unexpected bad code')
            }
          }
        }
      }
    ```
