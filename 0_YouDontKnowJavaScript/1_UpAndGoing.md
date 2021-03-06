# Title

## Built-in Type Methods

The built-in types and subtypes have behaviors exposed as properties and methods that are quite powerful and useful

Briefly, there is a _String_ object wrapper form typically called a _native_ that pairs with the primitive _string type_, this object wrapper is the one that defines the methods and properties

Ejemplo:

  ```js
  var a = "hello world";
  var b = 3.14159;
  a.length; // 11
  a.toUpperCase(); // "HELLO WORLD"
  b.toFixed(4); // "3.1416"
  ```

  When you use a primitive value like "hello world" as an object by referencing a property or method (e.g., a.toUpperCase() in the previous snippet)

  JS automatically “boxes” the value to its object wrapper counterpart (hidden under the covers)

A _string_ value can be wrapped by a _String object_

A _number_ can be wrapped by a _Number object_

A _boolean_ can be wrapped by a _Boolean object_

## Comparing Values

There are two main types of value comparison that you will need to make in your JS programs

1. equality
2. inequality

The result of any comparison is a strictly boolean value

### Coercion

Comes in two forms in JavaScript: _explicit_ and _implicit_

```js
// Explicit
var a = "42"
var b = Number(a)

a; // "42"
b; // 42
```

```js
var a = '42'
var b = a * 1 // "42" implicitly coerced to 42

a; // "42"
b; // 42
```

### Truthy & falsy

1. _Falsy_
    1. "" (empty string)
    2. 0, -0, NaN (invalid number)
    3. null, undefined
    4. false

2. _Truthy_
    Any value thats not falsy, is truthy

A non-boolean value only follow truthy/false coercion if it's actually coerced to a _boolean_

### Equality

The difference between `==` and `===` is that `==` allows coercion when checking for value equality

```js
var a = "42";
var b = 42;
a == b; // true
a === b; // false
```

## Function Scopes

You use the _var_ keyword to declare a variable that will belong to the current function scope, or the global scope if at the top level outside of any function

_Hoisting_ wherever a _var_ appears inside a scope, that declaration is taken to belong to the entire scope and accesible everywhere throughout

Metaphorically, this behavior is called _hoisting_, when a _var_ declaration is conceptually "moved" to the top of its enclosing scope

```js
var a = 2

foo(); // works because foo() declaration is "hoisted"

function foo() {
  a = 3;
  console.log(a) // 3
  var a;
}
console.log(a) // 2
```

### Nested scopes

When you declare a variable it is available anywhere in that scope as well as any lower/inner scopes

```js
function foo() {
  var a = 1

  function bar() {
    var b = 2

    function baz(){
      var c = 3

      console.log( a, b, c) // 1,2,3
    }

    baz();
    console.log( a,b ) // 1, 2
  }

  bar()
  console.log(a) //1
}

foo();
```

1. c no esta disponible en _bar()_ porque esta declarada en _baz()_
2. b no esta disponible en _foo()_

Con ES6 ya podemos definir variables con _let_ dentro de llaves {}, lo que genera que su scope no salga de ahi permitiendo mejor mantenibilidad

## Functions as Values

```js
function foo() {

}
```

En el código anterior, _foo_ es básicamente una variable in the outer enclosing scope thats given a reference to the _function_. That is, the function itself is a value

No solo se le puede pasar un argumento a una función, sino una función misma puede ser un valor que es asignado a una variable, pasada o retornada de otra función

```js
var foo = function() {

};

var x = function bar() {

};
```

1. The first function expression assigned to the _foo_ variable is called anonymous

2. The second function expression is named (bar)

## Immediately Invoked Function Expressions (IIFEs)

We could execute a function expression if we used `foo()` or `x()`

There's another way to execute a function expression, referred as immediately invoked function expression

```js
(function IIFE(){
  console.log('Hello')
})()
```

1. The outer (...) that surrounds the (`function IIFE(){..}`) function expression is just a nuance of JS grammar needed to prevent it from being treated as a normal function declaration

2. The final () at the end of the expression is what actually executes the function expression referenced immediately before

That may seem strange, but it's not as foreign as first glance, consider the similarities between foo and IIFE

```js
function foo() {...}

// foo function reference expression,
// then `()` executes it
foo();

// `IIFE` function expression,
// then `()` executes it
(function IIFE(){ .. })();
```

- Listing the (function IIFE(){ .. }) before its executing () is the same as including foo before its executing()
- In both cases, the function reference is executed with () immediately after it

IIFE is just a function, and functions create variable scope, using an IIFE is often used to declare variable that wont affect the surrounding code outside the IIFE

```js
var a = 42;
(function IIFE(){
  var a = 10;
  console.log( a ); //10
})();
console.log( a ) // 42
```

## Closure

You can think of closure as a way to remember and continue to access a functions scope (its variables) even once the function has finished running

```js
function makeAdder(x) {
  // parameter 'x' is an inner variable

  // inner function add() uses 'x' so
  // it has a 'closure' over it
  function add(y) {
    return y + x;
  };

  return add;
}
```

1. The reference to the inner _add()_ function that gets returned with each call to the outer _makeAdder()_ is able to remember whatever x value was passed in to _makeAdder()_

Lets use _makeAdder()_

```js
// `plusOne` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`
var plusOne = makeAdder( 1 );

// `plusTen` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`

49var plusTen = makeAdder( 10 );
plusOne( 3 );
plusOne( 41 ); // 4 <-- 1 + 3
plusTen( 13 ); // 42 <-- 1 + 41
```

1. When we call _makeAdder(1)_ we get back a reference to its inner _add()_ that remembers x as 1. We call this function reference _plusOne()_

2. When we call _plusOne(3)_, it adds 3 (its inner y) to the 1 (remembered by x) and we get 4 as the result

### Closure from MDN web docs

A closure is the combination of a function bundled together (enclosed) with reference to its surrounding state (the _lexical environment_)

In other words, a closure _gives you_ access to an outer functions scope from an _inner function_

In JavaScript, closures are created every time a function is created

1. _Lexical Scoping_

    Consider the following code

    ```js
    function init() {
      var name = 'Mozilla'; // name is a local variable created by init
      function displayName() { // displayName() is the inner function, a closure
        alert(name); // use variable declared in the parent function
      }
      displayName();
    }
    init();
    ```

    1. _init()_ creates a local variable _name_ and a function _displayName()_

    2. _displayName()_ is an inner function that is defined inside _init()_ and is available only within the body of the _init()_ function

    3. Inner functions have access to the variables of outer functions, _displayName()_ can access _name_

    This is an example of _lexical scoping_, which describes how a parser resolves variable names when functions are nested

    The word _lexical_ refers to the fact that lexical scoping uses the _LOCATION_ where a variable is declared within the source code to determine where that variable is available

2. _Closure_

    Consider the following code

    ```js
    function makeFunc() {
      var name = 'Mozilla';
      function displayName() {
        alert(name);
      }
      return displayName;
    }

    var myFunc = makeFunc();
    myFunc();
    ```

    Running this code has exactly the same effect as the previous example of the _init()_ function

    The difference and interesting is that the _displayName()_ inner function is _returned_ from the outer function _BEFORE BEING EXECUTED_

    Mucho ojo con lo último, se retorna la función sin haber sido EJECUTADA

    1. Intuitivamente, pensaríamos que al terminar de correr la function externa _makeFunc()_, la variable _name_ no seria accesible pero este no es el caso de JS por los closures

    2. A closure is the combination of a function and the lexical environment within which that function was declared

        - This environment consists of any local variables that were _in-scope_ at the time the closure was created

        - _myFunc_ is a reference to the instance of the function _displayName_ that is created when _makeFunc_ is run

        - La instancia de _displayName_ mantiene una referencia a su lexical environment, en el cual, la variable _name_ existe

        - Por esta razón, cuando _myFunc_ es invocada, la variable _name_ se puede usar y el `alert` envía el texto Mozilla

    ```js
    function makeAdder(x) {
      return function(y) {
        return x + y;
      };
    }

    var add5 = makeAdder(5);
    var add10 = makeAdder(10);

    console.log(add5(2));  // 7
    console.log(add10(2)); // 12
    ```

    En este ejemplo, definimos una función _makeAdder(x)_, que toma un argumento _x_ y retorna una nueva función

    La función que retorna toma un argumento _y_ y retorna la suma de x + y

    En esencia, _makeAdder_ es una fabrica de funciones. Crea funciones que pueden agregar un valor especifico al argumento que se le pase

    _add5_ y _add10_ son ambos _closures_. Comparten el mismo cuerpo de la función pero almacenan diferente _lexical environment_

    En _add5_ el _lexical environment_, x=5 mientras que en _add10_ es 10

3. _Practical closures_

    Los closures son útiles porque dejan asociar data (_lexical environment_) con una función que utiliza esa data

    Esto tiene parecido a la programación orientada a objeto, donde los objetos permiten asociar data con uno o mas métodos

    Podemos usar los closures en los casos que normalmente usaríamos un objeto con un solo método

    Situations where you might want to do this are particularly common on the web

    You define some behavior and then attach it to an event that is triggered by the user. The code is attached as a callback (a single function that is executed in response to the event)

    ```js
    function makeSizer(size) {
      return function() {
        document.body.style.fontSize = size + 'px';
      };
    }

    var size12 = makeSizer(12);
    var size14 = makeSizer(14);
    var size16 = makeSizer(16);

    document.getElementById('size-12').onclick = size12;
    document.getElementById('size-14').onclick = size14;
    document.getElementById('size-16').onclick = size16;
    ```

4. _Emulating private methods with closures_

    Private methods can be emulated by closures

    The following code illustrates how to use closures to define public functions that can access private functions and variables, these closures follow the _Module Design Pattern_

    ```js
    var counter = (function() {
      var privateCounter = 0;
      function changeBy(val) {
        privateCounter += val;
      }

      return {
        increment: function() {
          changeBy(1);
        },

        decrement: function() {
          changeBy(-1);
        },

        value: function() {
          return privateCounter;
        }
      };
    })();

    console.log(counter.value());  // 0.

    counter.increment();
    counter.increment();
    console.log(counter.value());  // 2.

    counter.decrement();
    console.log(counter.value());  // 1.
    ```

    1. In the previous examples, each closure had its own lexical environment

    2. Here, there is a single lexical environment that is shared by the three functions

    3. The lexical environment is created in the body of an anonymous function, which is executed as soon as it has been defined (IIFE)

    4. The lexical environment contains two private items, a variable called _privateCounter_ and a function _changeBy()_

    5. You can't access the private members from outside the anonymous function

    6. You can access them using the three public functions that are returned

    7. Those three public functions are closures that share the same lexical environment, they each have access to the _privateCounter_ and _changeBy()_

    ```js
    var makeCounter = function() {
      var privateCounter = 0;
      function changeBy(val) {
        privateCounter += val;
      }
      return {
        increment: function() {
          changeBy(1);
        },

        decrement: function() {
          changeBy(-1);
        },

        value: function() {
          return privateCounter;
        }
      }
    };

    var counter1 = makeCounter();
    var counter2 = makeCounter();

    alert(counter1.value());  // 0.

    counter1.increment();
    counter1.increment();
    alert(counter1.value()); // 2.

    counter1.decrement();
    alert(counter1.value()); // 1.
    alert(counter2.value()); // 0.
    ```

5. _Closure Scope Chain_

    Every closure has three scopes:

    1. Local scope (own scope)
    2. Outer functions scope
    3. Global scope

    A common mistake is not realizing that in the case where the outer functions _is itself_ a nested function, access to the outer function's scope includes the enclosing scope of the outer function

    Effectively creating a chain of function scopes

    ```js
    // global scope
    var e = 10;
    function sum(a){
      return function(b){
        return function(c){
          // outer functions scope
          return function(d){
            // local scope
            return a + b + c + d + e;
          }
        }
      }
    }

    console.log(sum(1)(2)(3)(4)); // log 20

    // You can also write without anonymous functions:

    // global scope
    var e = 10;
    function sum(a){
      return function sum2(b){
        return function sum3(c){
          // outer functions scope
          return function sum4(d){
            // local scope
            return a + b + c + d + e;
          }
        }
      }
    }

    var sum2 = sum(1);
    var sum3 = sum2(2);
    var sum4 = sum3(3);
    var result = sum4(4);
    console.log(result) //log 20
    ```

6. _Creating closures in loops: A common mistake_

    Prior to the introduction of the _let_ keyword, a common problem with closures occurred when you created them inside a loop

    Consider the example:

    ```html
    <p id="help">Helpful notes will appear here</p>
    <p>E-mail: <input type="text" id="email" name="email"></p>
    <p>Name: <input type="text" id="name" name="name"></p>
    <p>Age: <input type="text" id="age" name="age"></p>
    ```

    ```js
    function showHelp(help) {
      document.getElementById('help').textContent = help;
    }

    function setupHelp() {
      var helpText = [
          {'id': 'email', 'help': 'Your e-mail address'},
          {'id': 'name', 'help': 'Your full name'},
          {'id': 'age', 'help': 'Your age (you must be over 16)'}
        ];

      for (var i = 0; i < helpText.length; i++) {
        var item = helpText[i];
        document.getElementById(item.id).onfocus = function() {
          showHelp(item.help);
        }
      }
    }

    setupHelp();
    ```

    1. The _helpText_ array defines 3 helpful hints, each associated with the ID of an input field in the document

    2. The loop, cycles through these definitions, hooking up an _onfocus_ event to each one that show the associated help

    3. These will always show the help method for age (last one)

    4. The reason for this is that the functions assigned to _onfocus_ are closures

    5. They consist of the function definition and the captured environment from the _setupHelp()_ function scope

    6. Three closures have been created by the loop, but each one shares the same single lexical environment, which has a variable with changing values (_item_)

    7. This is because the variable _item_ is declared with _var_ and thus has function scope due to hoisting

    8. The value of _item.help_ is determined when the _onfocus_ callbacks are executed.

    9. Because the loop has already run its course by that time, the _item_ variable object has been left pointing to the last entry

    One solution in this case is to use more closures, in particular, to use a function factory

    ```js
    function showHelp(help) {
      document.getElementById('help').textContent = help;
    }

    function makeHelpCallback(help) {
      return function() {
        showHelp(help);
      };
    }

    function setupHelp() {
      var helpText = [
          {'id': 'email', 'help': 'Your e-mail address'},
          {'id': 'name', 'help': 'Your full name'},
          {'id': 'age', 'help': 'Your age (you must be over 16)'}
        ];

      for (var i = 0; i < helpText.length; i++) {
        var item = helpText[i];
        document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
      }
    }

    setupHelp();
    ```

    - This works as expected

    - Rather than the callbacks all sharing a single lexical environment, the _makeHelpCallback_ function creates a new lexical environment for each callback, in which _help_ refers to the corresponding string from the _helpText_ array

    One other wat to write the above using anonymous closures is

    ```js
    function showHelp(help) {
      document.getElementById('help').textContent = help;
    }

    function setupHelp() {
      var helpText = [
          {'id': 'email', 'help': 'Your e-mail address'},
          {'id': 'name', 'help': 'Your full name'},
          {'id': 'age', 'help': 'Your age (you must be over 16)'}
        ];

      for (var i = 0; i < helpText.length; i++) {
        (function() {
          var item = helpText[i];
          document.getElementById(item.id).onfocus = function() {
            showHelp(item.help);
          }
        })(); // Immediate event listener attachment with the current value of item (preserved until iteration).
      }
    }

    setupHelp();
    ```

    If you don't want to use more closures, you can use the _let_ keyword

    ```js
    function showHelp(help) {
      document.getElementById('help').textContent = help;
    }

    function setupHelp() {
      var helpText = [
          {'id': 'email', 'help': 'Your e-mail address'},
          {'id': 'name', 'help': 'Your full name'},
          {'id': 'age', 'help': 'Your age (you must be over 16)'}
        ];

      for (let i = 0; i < helpText.length; i++) {
        let item = helpText[i];
        document.getElementById(item.id).onfocus = function() {
          showHelp(item.help);
        }
      }
    }

    setupHelp();
    ```

### Modules

The most common usage of closure in JavaScript is the module pattern.

Modules let you define private implementation details (variables, functions) that are hidden from the outside world, as well as a public API that is accessible from the outside

```js
function User() {
  var username, password;

  function doLogin(user,pw) {
    username = user
    password = pw
  }

  var publicAPI = {
    login: doLogin
  }

  return publicAPI
}

// Create a User module instance
var fred = User();

fred.login("fred", "123")
```

1. The _User()_ function serves as an outer scope that
    1. Holds the variables `username`, `password` and the inner `doLogin()` function
        - These are all _PRIVATE_ inner details of this _User module_ that cannot be accessed from the outside world

2. Executing _User()_ creates an instance of the User module
    - A whole new scope is created, a new copy of each of these inner variables/functions
    - We assign this instance to _fred_
    - If we run _User()_ again, we did get a new instance entirely separate from _fred_

3. The inner _doLogin()_ function has a closure over username and password, meaning it will retain its access to them even after the _User()_ function finishes running

4. _publicAPI_ is an object with one property/method on it, _login_ which is a reference to the inner _doLogin()_ function.
    - When we return _publicAPI_ from User(), it becomes the instance we call _fred_

5. At this point, the outer _User()_ function has finished executing
    - Normally, you did think the inner variables like _username_ and _password_ have gone away
    - They have not, because there's a closure in the _login()_ function keeping them alive

6. That's why we can call _fred.login()_
    - That's the same as calling the inner _doLogin()_
    - _fred.login()_ can still access username and password

## this Identifier

If a function has a _this_ reference inside it, that reference usually points to an object

But which _object it point to_ depends on how the function was called

_this_ does not refer to the function itself

```js
function foo() {
  console.log( this.bar )
}

var bar = "global"

var obj1 = {
  bar: "obj1",
  foo: foo,
}

var obj2 = {
  bar: "obj2"
}

foo(); // global
obj1.foo(); // "obj1"
foo.call(obj2); // "obj2"
new foo(); // undefined
```

There are four rules for how _this_ gets set ann are shown in the last 4 lines of code

1. _foo()_ sets _this_ to the global object in non-strict mode
    - In strict mode, _this_ would be undefined and you would get an error

2. _obj1.foo()_ sets _this_ to the obj1 object

3. _foo.call(obj2)_ sets _this_ to the obj2 object
    - _Function.prototype.call()_ calls a function with a given _this_ value and arguments provided individually

4. _new foo()_ sets _this_ to a brand new empty object

To understand what _this_ points to, you have to examine how the function in question was called

## Prototypes

When you reference a property on an object and that property does not exist

1. JavaScript will automatically use that object's internal prototype reference to find another object to look for the property on

2. You could think of this almost as a fallback if the property is missing

3. The internal prototype reference linkage from one object to its fallback happens at the time the object is created

The simplest wat to illustrate it is with a built-in utility called `Object.create(..)`

```js
var foo = {
  a:42
};

// create 'bar' and link it to 'foo'
var bar = Object.create( foo )

bar.b = 'Hello world'

bar.b // Hello world
bar.ba // 42
```

![1](/images/YDKJS_0.png)

1. The `a` property doesn't actually exist on the `bar` object. But because `bar` is prototype-linked to `foo`, JavaScript automatically falls back to looking for `a` on the `foo` object

2. This linkage may seem like a strange feature of the language. The most common way this feature is used is try to emulate/fake a class mechanism with inheritance

3. A more natural way of applying prototypes is a patter called _behavior delegation_ where you intentionally design your linked objects to be able to delegate from one to the other for parts of the needed behavior

## Non-JavaScript

Most JS is written to run in and interact with environment like browsers. A good chunk of the stuff that you write in your code is, strictly speaking not directly controlled by JavaScript

The most common non-Javascript is the DOM API

`var el = document.getElementById("foo")`

1. The `document` variable exists as a global variable when your code is running in a browser.

2. It's not provided by the JS engine, nor is it particularly controlled by the Javascript specifications

3. It takes the form of something that looks like a normal JS object, but it's not really exactly that. It's special `object` often called a `host object`

4. The `getElementById(..)` method looks like a normal JS function, but it's just a thinly exposed interface to a built-in method provided by the DOM from your browser
