# this & Object Prototypes

## this or That ?

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
