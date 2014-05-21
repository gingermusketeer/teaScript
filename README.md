#TeaScript (Coffee is overrated)

Note this is very early stage and definitely contains traces of javascript. Syntax is by no means anything close to final

##Goals
- Fix js weird/broken/anoying behaviour
- Prevent bikeshedding
  - Style build in e.g tabs or spaces not both
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


  // Fixed number type

  num1 = 10.7 //=> new BigNumber('10.7')
  num2 = 0.3 //=> new BigNumber('0.3')
  sum = num1 + num2 //=> num1['+'](num2)

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

  // Native html element support

  renderedProducts =  <ul><li>Pizza</li></ul>
  //=> renderedProducts = new MyDomWrapper('<ul><li>Pizza</li></ul>')
```

#### Explicit variable type conversion

When a scoped primitive has been loaded variables/parameters can automatically be converted

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

### context
Allows functions to be executed in a context

```js
  function setATo5AndAddAWithB() withContext {
    a = 5
    return a + b
  }

  setATo5AndAddAWithB(ç:{a: 1, b: 2}) // ç: sets the context
  //=>
  /*
  function setATo5AndAddAWithB(){
    var context = setATo5AndAddAWithB.context
    context.a = 5
    return context.a + context.b
  }

  setATo5AndAddAWithB.context = { a:1, b:2 }

  setATo5AndAddAWithB()

  setATo5AndAddAWithB.context = null
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
