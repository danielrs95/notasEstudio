# this & Object Prototypes

## 1. this or That ?

_this_  is a special identifier keyword that's automatically defined in the scope of every function

### Why this?

Let's try to illustrate the motivation and utility of _this_

```js
function identify() {
  return this.name.toUpperCase();
}

function speak() {
  var greeting = "Hello, I'm " + identify.call( this );
  console.log( greeting );
}

var me = {
  name: "Kyle"
};

var you = {
  name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```

- The _how_ will be addressed later, let's look on the _why_

- This code snippet allows the `identify()` and `speak()` functions to be reused against multiple _context_ objects(`me` and `you`)

  - Rather than needing a separate version of the function for each object

Instead of relying on _this_ you could have explicitly passed in a context object to both `identify()` and `speak()`

```js
function identify(context) {
  return context.name.toUpperCase();
}

function speak(context) {
  var greeting = "Hello, I'm " + identify( context );
  console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

- _this_ provides a more elegant way of implicitly "passing along" an object reference, leading to cleaner API design and easier reuse

- The more complex the model, more messier gets to pass explicit _context_

### Confusions

_this_ creates confusion when developers try to think about it too literally. There are two meanings often assumed, both incorrect

#### Itself

The first common temptation is to assume _this_ refers to the function itself

Consider the following code, where we attempt to track how many times a function was called

```js
function foo(num) {
  console.log( "foo: " + num );

  // keep track of how many times `foo` is called
  this.count++;
}

foo.count = 0;
var i;

for (i=0; i<10; i++) {
  if (i > 5) {
    foo( i );
  }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```

- `foo.count` is still 0, but `foo(..)` was in fact called four times
  - The frustration stems from a too literal interpretation of what _this_ (in `this.count++`) means

- `foo.count=0` is adding a property `count` to the function object `foo`
  - But `this.count` reference inside the function, `this` is not pointing to that function object

- Accidentally a global variable `count` with a value of `NaN` was created

Some developers would then, hack the solution creating another object to hold the `count` property

```js
function foo(num) {
  console.log( "foo: " + num );

  // keep track of how many times `foo` is called
  data.count++;
}

var data = {
  count: 0
}

var i;

for (i=0; i<10; i++) {
  if (i > 5) {
    foo( i );
  }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( data.count ); // 4
```

To reference a function object from inside itself, `this` by itself will typically be insufficient, you need a reference to the function via a lexical identifier (variable) that points at it

```js
function foo(num) {
  foo.count = 4; // 'foo' refers to itself
}

setTimeout( function(){
  // anonymous function (no name), cannot
  // refer to itself
}, 10 );
```

- The function callback passed to `setTimeout` has no name identifier, so there's no proper way to refer to the function object itself

```js
function foo(num) {
  console.log( "foo: " + num );

  // keep track of how many times `foo` is called
  foo.count++;
}

foo.count = 0

var i;

for (i=0; i<10; i++) {
  if (i > 5) {
    foo( i );
  }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

Another way of approaching the issue is to force `this` to actually point at the `foo` function object using `call`

```js
function foo(num) {
  console.log( "foo: " + num );

  /// keep track of how many times `foo` is called
  // Note: `this` IS actually `foo` now, based on
  // how `foo` is called (see below)
  this.count++;
}

foo.count = 0

var i;

for (i=0; i<10; i++) {
  if (i > 5) {
    // using `call(..)`, we ensure the `this`
    // points at the function object (`foo`) itself
    foo.call( foo,i );
  }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

#### Its Scope

The next most common misconception about the meaning of `this` is that is somehow refers to the functions scope

`this` does not in any way, refer to a functions lexical scope

### Whats this?

- `this` is not an author-time binding, but a _runtime biding_
- It is contextual based on the conditions of the functions invocation
- `this` binding has nothing to do with where a function is declared, but has to do with the manner in which the function is called

When a function is invoked, an activation record, (execution context) is created

- This record contains info about where the function was called from (the call-stack)

- _HOW_ the function was invokes and _WHAT_ parameters were passed

- One of the properties of this record is the `this` reference, which will be used for the duration of that functions execution

### Review

- `this` is neither a reference to the function itself, nor is it a reference to the functions lexical scope

- `this` is actually a binding that is made when a function is invoked, and _what_ it references is determined entirely by the call-site where the function was called

## 2. this All Makes Sense Now

_this_ is a binding for each function invocation, based entirely on its call-site (how the function is called)

### Call-site

To understand _this_ binding, we have to understand the _call-site_, the location in code where a function is called (not where it's declared)

We must inspect the call-site to answer the question, what is _this_ referencing to ?

Its important to think about the _call-stack_ (the stack of functions that have been called to get us to the current moment in execution)

The _call-site_ we care about is in the invocation _before_ the currently execution function

```js
function baz() {
  // call-stack is: `baz`
  // so, our call-site is in the global scope

  console.log( "baz" );
  bar(); // <-- call-site for `bar`
}

function bar() {
  // call-stack is: `baz` -> `bar`
  // so, our call-site is in `baz`

  console.log( "bar" );
  foo(); // <-- call-site for `foo`
}

function foo() {
  // call-stack is: `baz` -> `bar` -> `foo`
  // so, our call-site is in `bar`

  console.log( "foo" );
}

baz(); // <-- call-site for `baz`
```

### Nothing but Rules

Lets learn how the _call-site_ determines where _this_ will point during the execution of a function

You must inspect the _call-site_ and determine which of 4 rules applies

#### Default Binding

The most common case of function calls: standalone function invocation. Think of this as the default catch-all rule

```js
function foo() {
  console.log(this.a)
}

var a = 2
foo() // 2
```

- Variables declares in the global scope (`var a=2`) are synonymous with _global-object properties_ of the same name

- When `foo()` is called, `this.a` resolves to our global variable `a`

  - The _default binding_ for `this` applies to the function call, an points `this` to the global object

- We examine the call-site to se how `foo()` is called

  - `foo()` is called with a plain, undecorated function reference. None of the other rules apply here, so the _default binding_ applies

If `strict mode` is in effect, the global object is not eligible for the _default binding_

```js
function foo() {
  'use strict'

  console.log(this.a)
}

var a = 2
foo() // TypeError: `this` is `undefined`
```

#### Implicit Binding

Another rule is whether the _call-site_ has an _context object_ (also referred to as an owning or containing object)

```js
function foo() {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

obj.foo(); // 2
```

1. Notice how `foo()` is declared and then added as a reference property onto `obj`

    - The function is not really "owned" or "contained" by the `obj` object

    - However, the _call-site_ uses the `obj` context to reference the function. So you could say the `obj` object "owns" or "contains" the _function reference_ at the time the function is called

2. At the point that `foo()` is called, it's preceded by and object reference to `obj`

    - When there is a context object for a function reference, the _implicit binding_ rule says that it's that object that should be used for the function calls `this` binding

    - Because `obj` is the `this` for the `foo()` call, `this.a` is synonymous with `obj.a`

Only the top/last level of an object property reference chain matters to the _call-site_

```js
function foo() {
  console.log(this.a)
}

var obj2 = {
  a: 42,
  foo: foo
}

var obj1 = {
  a: 2,
  obj2: obj2
}

obj1.obj2.foo(); // 42
```

##### Implicitly lost

One of the most common frustrations that `this` binding creates is when an implicitly bound function loses that binding, which usually falls back to the _default binding_

```js
function foo() {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

var bar = obj.foo

var a = "oops global" // 'a' also property on global object
bar(); // oops global
```

1. `bar` appears to be a reference to `obj`, it's really just another reference to `foo` itself

    - The _call-site_ is what matters, and the _call-site_ is `bar()`, which is a plain, undecorated call, and thus the _default binding_ applies

The more subtle, more common, and unexpected way this occurs is when we consider passing a callback function

```js
function foo() {
  console.log( this.a );
}

function doFoo(fn) {
  // `fn` is just another reference to `foo`

  fn(); // <-- call-site!
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global"; // `a` also property on global object
doFoo( obj.foo ); // "oops, global"
```

1. Parameter passing is just an implicit assignment

    - Since we're passing a function, it's an implicit reference assignment, the end result is the same as the previous snippet

Que pasa si a la función a la que le pasamos el _callback_ no es nuestra función sino una que viene por defecto en el lenguaje. No  hay diferencia, es el mismo resultado

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global"; // `a` also property on global object

setTimeout(obj.foo, 100) // oops, global
```

Think about this crude theoretical pseudoimplementation of `setTimeout()`

```js
function setTimeout(fn, delay) {
  // wait for the delay
  fn(); // <-- call-site!
}
```

1. Es común que nuestra function callback pierda su `this` binding

2. Pero, otra manera en que `this` nos sorprende es cuando la función a la cual le estamos pasando el _callback_ intencionalmente cambia el `this` for the call

    - Event handlers in JS libraries are quite fond of forcing your callback to have a `this` that points to, for instance, the DOM elements that triggered the event

#### Explicit Binding

All functions have some utilities available to them (via their `[[Prototype]]`), specifically `call(..)` and `apply(...)` methods

This both methods take, as their first parameter, an object to use for the `this` and then invoke the function with that `this` specified.

Since you are directly stating what `this` to be, is called _explicit binding_

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};

foo.call(obj) // 2
```

- If you pass a simple primitive value (string, boolean or number) as the `this` binding, the primitive value is wrapped in its objects form (`new String(..)`, `new Boolean(..)`, `new Number(...)`), this is called boxing

- _explicit binding_ alone still doesn't offer any solution to a function losing its intended `this` binding

##### Hard binding

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
};

var bar = function() {
  foo.call(obj)
}

bar() // 2
setTimeout(bar,100) // 2

// hard-bound `bar` can no longer have its `this` overridden
bar.call( window ); // 2
```

1. We create a function `bar()`, internally, manually calls `foo.call(obj)`

    - Forcibly invoking `foo` with `obj` binding for `this`
    - No matter how you later invoke the function `bar`, it will always invoke `foo` with `obj`
    - This binding is both explicit and strong, we call it _hard binding_

The most typical way to wrap a function with a _hard binding_ creates a pass-through of any arguments passed and any return value received

```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something
}

var obj = {
  a: 2,
};

var bar = function() {
  return foo.apply(obj,arguments)
}

var b = bar(3) // 2 3
console.log(b) // 5
```

Another way to express this pattern is to create a reusable helper:

```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something
}

// simple 'bind' helper
function bind(fn, obj) {
  return function() {
    return fn.apply(obj,arguments)
  }
}

var obj = {
  a: 2,
};

var bar = bind(foo, obj)

var b = bar(3) // 2 3
console.log(b) // 5
```

Since _hard binding_ is such a common patter, it's provided with a built-in utility as of ES5, `Function.prototype.bind`

```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something
}

var obj = {
  a: 2,
};

var bar = foo.bind(obj)

var b = bar(3) // 2 3
console.log(b) // 5
```

1. `bind(...)` return a new function that is hardcoded to call the original function with the `this` context set as you specified

##### API call "contexts"

Many libraries functions, and many new built-in functions in JS and host environment, provide an optional parameter usually called "context" which is designed as a work-around for not having to use `bind(..)`

```js
function foo(el) {
  console.log( el, this.id );
}

var obj = {
  id: "awesome",
};

// use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj );
// 1 awesome 2 awesome 3 awesome
```

1. Internally, these functions almost certainly use _explicit binding_ vial `call(...)` or `apply(...)`

#### new Binding

In JS, constructors are just functions that happen to be called with the `new` operator in front of them.
    - They are not attached to classes, nor are they instantiating a class
    - They are not special types of functions
    - They are just regular functions that are, in essence, hijacked by the use of `new` in their invocation

There's really no such thing as "constructor functions" but rather construction calls of functions

When a function is invoked with `new` in front of it, otherwise known as a _constructor call_

  1. A brand new object is created (aka constructed) out of thin air
  2. The newly constructed object is `[[Prototype]]` linked
  3. The newly constructed object is set as the `this` binding for that function call
  4. Unless the function returns its own alternate object, the new invoked function call will _automatically_ return the newly constructed object

```js
function foo(a) {
  this.a = a;
}
var bar = new foo( 2 );
console.log( bar.a ); // 2
```

1. By calling `foo(...)` with `new` in front of it
    - We've constructed a new object
    - Set that new object as the `this` for the call of `foo(..)`

### Everything in Order

1. _explicit_ takes precedence over _implicit_ binding
    - You should ask first if _explicit_ applies before checking for _implicit_
