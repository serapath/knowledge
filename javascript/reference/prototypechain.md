# Prototype Chain

My thoughts on: https://davidwalsh.name/javascript-objects-deconstruction#simpler-object-object

I think after revisiting the topic after a very very long time I tried to summarize my thoughts below.  
I tried to cover all edge cases and make stuff very fast and flexible.

Old ways:

* `Object.create` is magic too - maybe more than `new` which people might know from other contexts
* `this` is something that cannot be avoided, because it has to be used in "prototype methods" anyway

All the old ways of object instantiation use magic to set the prototype chain, but now you can do it directly.

* `__proto__` attribute can be considered "magic" too, but at least it sets directly the one thing we are interested in :-)

The `__proto__` property is nice, because many browsers implemented it already when it was not a standard, but with `es6` it's even a standard now. It allows to set **directly** the one thing that we care about in all this, which is:
* **the `prototype chain` of objects** :-)

### Different ways of setting up the JavaScript Prototype Chain
```js
function ES5_Classic_Version (test) {
/*****************************************************************************/
  Person.prototype.sayName = function () { console.log('My name is ' + this.name) }
  Person.prototype.sayAge  = function () { console.log('I am ' + this.age) }
  function Person (name, age) {
    if (!(this instanceof Person)) return new Person(name, age)
    this.name = name
    this.age = age
  }
/*****************************************************************************/
  Student.prototype = Object.create(Person.prototype)
  Student.prototype.saySkill = function () { console.log('I know ' + this.skill) }
  function Student (name, age, skill) {
    if (!(this instanceof Student)) return new Student(name, age, skill)
    Person.call(this, name, age)
    this.skill = skill
  }
/*****************************************************************************/
  ;(test||function(){})(console.log((arguments.callee + '').split('(')[0].split(' ')[1]), Person, Student)
}



function ES5_Version___proto__ (test) {
/*****************************************************************************/
  Person.prototype.sayName = function () { return 'My name is ' + this.name }
  Person.prototype.sayAge = function () { return 'I am ' + this.age }
  function Person (name, age) { return { __proto__: Person.prototype, name: name, age: age } }
/*****************************************************************************/
  Student.prototype.__proto__= Person.prototype
  Student.prototype.saySkill = function () { return 'I know ' + this.skill }
  function Student (name, age, skill) {
    var obj = Person(name, age)
    obj.__proto__ = Student.prototype
    obj.skill = skill
    return obj
  }
/*****************************************************************************/
  ;(test||function(){})(console.log((arguments.callee + '').split('(')[0].split(' ')[1]), Person, Student)
}



function ES5_Version___proto___short (test) {
/*****************************************************************************/
  Person.prototype = { sayName: function () { return 'My name is ' + this.name }, sayAge: function () { return 'I am ' + this.age } }
  function Person (name, age) { return { __proto__: Person.prototype, name: name, age: age } }
/*****************************************************************************/
  Student.prototype = { __proto__: Person.prototype, saySkill: function () { return 'I know ' + this.skill } }
  function Student (name, age, skill) { var o = Person(name, age); o.__proto__ = Student.prototype; o.skill = skill; return o; }
/*****************************************************************************/
  ;(test||function(){})(console.log((arguments.callee + '').split('(')[0].split(' ')[1]), Person, Student)
}



function ES6_Version (test) {
/*****************************************************************************/
  Person.prototype.sayName = function () { console.log('My name is ' + this.name) }
  Person.prototype.sayAge = function () { console.log('I am ' + this.age) }
  function Person (name, age) { return { __proto__: Person.prototype, name, age } }
/*****************************************************************************/
  Student.prototype.__proto__ = Person.prototype
  Student.prototype.saySkill = function () { console.log('I know ' + this.skill) }
  function Student (name, age, skill) { return Object.assign({ __proto__: Student.prototype, skill }, Person(name, age)) }
/*****************************************************************************/
  ;(test||function(){})(console.log((arguments.callee + '').split('(')[0].split(' ')[1]), Person, Student)
}



function ES6_Version_short (test) {
/*****************************************************************************/
  Person.prototype = { sayName () { console.log('My name is ' + this.name) }, sayAge () { console.log('I am ' + this.age) } }
  function Person (name, age) { return { __proto__: Person.prototype, name, age } }
/*****************************************************************************/
  Student.prototype = { __proto__: Person.prototype, saySkill () { console.log('I know ' + this.skill) } }
  function Student (name, age, skill) { return Object.assign({ __proto__: Student.prototype, skill }, Person(name, age)) }
/*****************************************************************************/
  ;(test||function(){})(console.log((arguments.callee + '').split('(')[0].split(' ')[1]), Person, Student)
}
```

### ES7 - doesnt work yet
```js
function ES7_Version_without_ObjectAssign (test) {
/*****************************************************************************/
  Person.prototype.sayName = function () { console.log('My name is ' + this.name) }
  Person.prototype.sayAge = function () { console.log('I am ' + this.age) }
  function Person (name, age) { return { __proto__: Person.prototype, name, age } }
/*****************************************************************************/
  Student.prototype.__proto__ = Person.prototype
  Student.prototype.saySkill = function () { console.log('I know ' + this.skill) }
  function Student (name, age, skill) { return { ...Person(name, age), __proto__: Student.prototype, skill } }
/*****************************************************************************/
  ;(test||function(){})(console.log((arguments.callee + '').split('(')[0].split(' ')[1]), Person, Student)
}


function ES7_Version_without_ObjectAssign_short (test) {
/*****************************************************************************/
  Person.prototype = { sayName () { console.log('My name is ' + this.name) }, sayAge () { console.log('I am ' + this.age) } }
  function Person (name, age) { return { __proto__: Person.prototype, name, age } }
/*****************************************************************************/
  Student.prototype = { __proto__: Person.prototype, saySkill () { console.log('I know ' + this.skill) } }
  function Student (name, age, skill) { return { ...Person(name, age), __proto__: Student.prototype, skill } }
/*****************************************************************************/
  ;(test||function(){})(console.log((arguments.callee + '').split('(')[0].split(' ')[1]), Person, Student)
}
```

### Test

```js
function test (name, Person, Student) {
  console.log(name)
  var x = Person('jenny', 20)
  //console.log(x)
  x.sayName()
  x.sayAge()
  console.log(x instanceof Person)
  console.log(x instanceof Student)
  var y = Student('jimmy', 22, 'history')
  console.log(y)
  y.sayName()
  y.sayAge()
  y.saySkill()
  console.log(y instanceof Person)
  console.log(y instanceof Student)
  console.log('-------- with `new` --------')
  var x = new Person('jenny', 20)
  console.log(x)
  x.sayName()
  x.sayAge()
  console.log(x instanceof Person)
  console.log(x instanceof Student)
  var y = new Student('jimmy', 22, 'history')
  console.log(y)
  y.sayName()
  y.sayAge()
  y.saySkill()
  console.log(y instanceof Person)
  console.log(y instanceof Student)
  console.log('----------------------------')
}

;[
  ES5_Classic_Version,
  ES5_Version___proto__,
  ES5_Version___proto___short,
  ES6_Version,
  ES6_Version_short,
  ES7_Version_without_ObjectAssign,
  ES7_Version_without_ObjectAssign_short
].forEach(function (scenario) { scenario(test) })
```
