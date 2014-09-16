#TeaScript (Coffee is overrated)

Note this is very early stage and definitely contains traces of javascript. Syntax is by no means anything close to final

##Goals
- Superset of javascript. Without driving you to drink coffee
- Preserve line numbers by compiling line to line
- Fix js weird/broken/annoying behaviour
- Prevent bikeshedding
  - Style build in e.g tabs or spaces not both
  - Most likely do this go style with a standard formatting tool
  - Version control friendly
    - No commas or required trailing commas
- Compile time access to AST
- Scoped primitives
- Take advantage of improvements to js transparently
- Provide tooling that encourages good software
 - Built in support for documentation


##Features

### File Scoped primitives

```js
  // Support for binary numbers

  num = 0101b //=> new BinaryNumber('0101b')

  // Support for decent string library

  someString = 'CamelCaseString'.toUnderscoreCase()
  //=> (new MyStringLib('CamelCaseString')).toUnderscoreCase()
  //=> myStringLib.toUnderscoreCase.call('CamelCaseString')
  //=> try('toUnderscoreCase', strToUnderScoreCase)

  // Fixed number type

  num1 = 10.7 //=> new BigNumber('10.7')
  num2 = 0.3 //=> new BigNumber('0.3')
  sum = num1 + num2 //=> num1['+'](num2)
  complex = num1 + num2 * num1 / num2
  // Apply js operator precedence rules i.e bedmas
  //-> num1+(num2 *(num1/num2))
  //=> num1['+'](num2['*'](num1['/'](num2)))

  // Date primitive

  startDate = 2014-02-01 //=> new moment('2014-02-01')
  endDate = 2014-02-05 //=> new moment('2014-02-05')

  startDate > endDate //=> startDate['>'](endDate)

  // Native data structures
  e = 'e'
  d = 'd'
  f = 'f'

  bst = e
       / \
      d   f
  //=> #compiles to something useful
  bst.contains('e') //=> true

  a = {}
  b = {}

  linkedList = a<->b //=> new LinkedList(a, b)

  linkedList.head()  == a //=> true
  linkedList.head().child() == b //=> true
  
  // Vectors
  <1,2,3> dot <2,3,4>
  // => 32
  var x = <1,2,3>
  var magnitude = |x|
  // => 3.7416
  
  // Sets
  var x = {1,2,3}
  var y = {4,5} union x
  // => {1,2,3,4,5}
  var z = {5,6} intersect y
  // => {5}
  
  // Tuples
  var x = (A,B,C)
  var y = x select 1,3
  // => (A,C)
  
  function y () {
    ("B", <1,2,3>, new Object())
  }
  
  var x = y() select 2, 3
  // => (<1,2,3>,{})
  var _,x = y()
  // => <1,2,3>
  
  // Native html element support

  renderedProducts =  <ul><li>Pizza</li></ul>
  //=> renderedProducts = new MyDomWrapper('<ul><li>Pizza</li></ul>')

  // Unwrap helper
  return <- mything //=> return mything.unwrap()
```

Could use some sort of resolver to handle dealing with primitive operations and redeclaration of built in types

primativeResolver('+', someVar, someOtherVar)

for strings we could perhaps extend existing string object or for methods we could replace someVar.toCamelCase with prototype call with this provided

#### Explicit variable type conversion

When a scoped primitive has been loaded variables/parameters can automatically be converted

Note this is only required for primatives which do not extend existing js primatives

```js
  // Convert parameter

  function snakeify('animal') {
    return animal.toUnderScoreCase()
  }
  /* =>
  function snakeify(animal) {
    animal = new MyStringLib(animal)
    return animal.toUnderscoreCase()
  }
  */
```

#### Invoking a primitive
```js
%'max %0.2f'% [100]
//=> myString("max %0.2f").invoke([100])
```

### Integrated documentation

Include docs in development code so it can be accessed from the repl.
Docs in markdown for the win


```js
  'md
   Adds two numbers
   @param a A number -> 1
   @param b A number -> 2
   @return The sum of the numbers -> 3
   @website [How to math](http://wikipedia.org/maths)
   '
  function add(a,b) {
    return a + b
  }

  add._docs()
  //=> Adds two numbers e.g add(1,2) => 3
  add._docs().website() //=> opens new tab with the published docs
```


### this => self

If self is used then the function is automatically bound to the correct context

```js
  function Animal(){}

  Animal.prototype.isMammal = function(){
    return propertiesOfMammal.checkEach(function(key, value){
      return self[key] == value
    });
  }
  // =>
  /*
  Animal.prototype.isMammal = function(){
    return propertiesOfMammal.checkEach(function(key, value){
      return this[key] == value
    }.bind(this));
  }
  */
```

### Simplified prototypical inheritance

Calling proto() will call the method with the same name in the prototype

```js

  function Alive () {
    // ...
  }
  
  Alive.prototype.heart = function () {
    return 10;
  }
  
  function Alive >> Animal() {
    // ...
  }
  
  Animal.prototype.heart = function() {
    return 5;
  }
  
  function Animal >> Beaver {
    // automatically extends prototype chain for beaver.
  }
  
  Beaver.prototype.heart = function() {
    return proto(1, args...); // Return 5. Went one level up the tree explicitly.
  }
  
  // OR
  
  Beaver.prototype.heart = function() {
    return proto(2, args...); // Returns 10. Went two levels up the tree explicitly.
  }
  
  var beaver = new Beaver();
  
  //=>
  /*
  a.has4Legs = function(){
    return this.prototype.apply(this, arguments)
  }
  */

```

### module system

Should be interoperable with common/AMD/es6 but allow dependency injection for testing purposes

```js

 import {
   express
   stdlib/String as MyString
 }

 express.createServer()

 //=>
 /*
  module.exports = {
    deps: ['express', {MyString: 'stdlib/String'}]
    run: function(express, MyString){
      express.createServer()
    }
  }
 */
```

### context
Allows functions to be executed in a context. Could allow the 'closure' of a
function to be exposed. Might need prototypically linked contexts for this to
work

```js
  function setATo5AndAddAWithB() withContext {
    a = 5
    return a + b
  }
  // or (pipe indicates blocked scope)
  setATo5AndAddAWithB {|
    a = 5
    return a + b
  |}

  setATo5AndAddAWithB(ç:{a: 1, b: 2}) // ç: sets the context
  //=>
  /*
  function setATo5AndAddAWithB(){
    var context = setATo5AndAddAWithB.context
    setATo5AndAddAWithB.context = null
    context.a = 5
    return context.a + context.b
  }

  setATo5AndAddAWithB.context = { a:1, b:2 }

  setATo5AndAddAWithB()


  */

```

`context.arguments()` could be a helper that fixes arguments weirdness



### Async like a boss

\#TBD

### Simple function defs

```js
  greet -> {
      console.log('hi')
  }
  //=>
  /*
  greet = function greet(){
    console.log('hi')
  }
  */
```
 or

 Cannot just use {...} as it is valid js
 ```js
 f{
   // do something
 }
 ```
 explicit functions with explicit returns
 ```js

 f{
  // do something and last expression will be returned
 >}
 ```

 Debug blocks. Run by default. Removed by compiling with --remove-debug-blocks
 Rather then removing them it could just comment them out and let uglification
 handle their removal
 ```js
 debug{
   assert typeof object == 'object'
   inspect object //=> pretty print object
   puts object //=> display string representation of object
 }
 //=>
 var __debug = require('teascript/debug');
 ___debug(function myFunc(){
   var context = myFunc.context;
   myFunc.context = null;
   context.assert(typeof object =='object')
   context.inspect(object)
   context.puts(object)
 });
 ```
 Might be a use case for functions with contexts. Doing this would allow people
 to change the default behaviour of assert, inspect, puts etc. Also prevents
 global pollution

### No commas

\#TBD

Do we need them to determine if there is a list of items?

If we do need them then we should make trailing commas required #vcs


### Compile time AST hooks

Allow modification of the teaScript and javascript AST during compilation

Could use annotations to provide hook points

#### Allows compile time checks of other languages
 E.g check sql for syntax

 ```js
 @sql
 vehicleQuery = 'select * from vehicles'
 ```

### DANGER ZONE

'danger zone'; when doing something valid syntatically valid js but warned
against by default

### Editor features

Provide visual indicator of implicit returns. Maybe this suggests that it is a
bad idea...
