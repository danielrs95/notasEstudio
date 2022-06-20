# Types & Grammar

ECMAScript types are

1. `undefined`
2. `null`
3. `boolean`
4. `string`
5. `number`
6. `object`
7. `symbol`

All of these types except `object` are called _primitives_

## 1. Types

### Values as Types

In JS, variables _don't_ have types, _values have types_. Variables can hold any value, at any time

JS doesn't have type enforcement, a variable can, in one assignment hold a `string` and in the next a `number`

#### undefined vs 'undeclared'

Variables that have no value _currently_ actually have the `undefined` value

It's tempting to think of the word `undefined` as a synonym for _undeclared_, however, in JS these two concepts are different

1. _undefined_ variable declared in the accessible scope, but at the moment has no other value in it
2. _undeclared_ variable that has not been formally declared in the accessible scope

```js
var a;

a; // undefined
b; // ReferenceError: b is not defined
```

1. The message error the browser assign can be misleading
    - A better option could be 'b is not declared'

There's a special behavior associated with `typeof` as it relates to undeclared variables

```js
var a;

typeof a; // undefined
typeof b; // undefined
```

The `typeof` returns undefined even for _undeclared_ variables. Notice that there was no error thrown when we executed `typeof b` even when b is an undeclared variable.

- This is a _special safety_ guard in the behavior of `typeof`

#### typeof Undeclared

You have to take care in how you check for the global variables so you don't throw a `ReferenceError`. The safety guard on `typeof` is our friend in this case

```js
// oops, this would throw an error!
if (DEBUG) {
  console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
  console.log( "Debugging is starting" );
}
```

This check is useful even if you are doing a feature check for a built-in API, you may find it helpful to check without throwing an error

```js
if (typeof atob === "undefined") {
  atob = function() { /*..*/ };
}
```

## 2. Values

### Arrays

JS `arrays` are just containers for any type of value

`arrays` are numerically indexed, but the tricky thing is that they also are objects that can have `string` keys/properties added to them (but which don't count toward the `length` of the `array`)

```js
var a = [ ];

a[0] = 1;
a["foobar"] = 2;
a.length; // 1
a["foobar"]; // 2
a.foobar; // 2
```

Be aware that if a `string` value intended as a key can be coerced to a standard base-10 `number` then it is assumed that you wanted to use it as a `number` index rather than as a `string` key

```js
var a =[]

a['13'] = 42;
a.length; // 14
```

#### Array-Likes

Sometimes you have array-like data and you need to turn it into a true array to make use of the array utilities like `forEach()`

```js
function foo() {
  var arr = Array.prototype.slice.call(arguments)
  arr.push("bam")
  console.log(arr)
}

foo( "bar", "baz" ); // ["bar","baz","bam"]
```

1. If `slice()` is called without any other parameters, have the effect of duplicating the `array`
2. With ES6 there's a built-in utility called `Array.from(..)` that can do the same task
    > var arr = Array.from(arguments)

### Strings

It's a common believe that `strings` are just `arrays` of characters

This is not true, the similarity is mostly just skin-deep

```js
var a = "foo";
var b = ["f","o","o"];

a.length; // 3
b.length; // 3

a.indexOf( "o" ); // 1
b.indexOf( "o" ); // 1

var c = a.concat( "bar" ); // "foobar"
var d = b.concat( ["b","a","r"] ); // ["f","o","o","b","a","r"]

a === c; // false
b === d; // false

a; // "foo"
b; // ["f","o","o"]
```

1. Strings do have a shallow resemblance to `arrays`
2. Both of them have a `length` property, and `indexOf()` method and a `concat()` method

They look like "array of characters" but not exactly

```js
a[1] = "O";
b[1] = "O";

a; // "foo"
b; // ["f","O","o"]
```

1. JS `strings` are immutable, while `arrays` are quite mutable
2. The `a[1]` access form was not always valid
    - The correct approach has been `a.charAt(1)`

3. A consequence of immutable `strings` is that none of the `string` methods that alter its contents can modify in place, it create and return new `string`

### Numbers

JS has just one numeric type `number`, this type includes integer values and fractional decimal numbers

The implementations of `numbers` in JS is based on the "IEEE 754" standard, often called floating-point

#### Numeric Syntax

Number literals are expressed in JS generally as base-10 decimal literals

The leading portion of a decimal value, if 0, is optional. Similarly, the trailing portion of a decimal value after the . if 0, is optional

```js
var a = 0.42;
var b = .42;

var a = 42.0;
var b = 42.;
```

`number` values can be boxed with the `Number` object wrapper, values can access methods that are built into the `Number.prototype`

```js
var a = 42.59;
a.toFixed( 0 ); // "43"
a.toFixed( 1 ); // "42.6"
```

#### Small Decimals Values

The most infamous side effect of using binary floating point numbers (true for all languages that use IEEE 754) is

```js
0.1 + 0.2 === 0.3; // false
```

1. The representations for 0.1 and 0.2 in binary point are not exact

#### Safe Integer Ranges

Because of how `numbers` are represented, there is a range of safe values for the whole `number` "integers", and it's significantly less than `Number.MAX_VALUE`

This value is 2^53 and is automatically predefined in ES6 as `Number.MAX_SAFE_INTEGER`. There is also a minimum value -2^53 and it's defined as `Number.MIN_SAFE_INTEGER`

JS programs are confronted with this numbers when dealing with 64-bit IDs from databases, etc

#### Testing for Integers

With ES6 you can use `Number.isInteger(...)`

```js
Number.isInteger( 42 ); // true
Number.isInteger( 42.000 ); // true
Number.isInteger( 42.3 ); // false

Number.isSafeInteger( Number.MAX_SAFE_INTEGER ); // true
Number.isSafeInteger( Math.pow( 2, 53 ) ); // false
Number.isSafeInteger( Math.pow( 2, 53 ) - 1 ); // true
```

#### Special Values

There are several special values spread across the various types

##### The nonvalue Values

1. For `undefined` type, there is only one value: `undefined`
2. For `null`, there is only one value: `null`

Both are often taken to be interchangeable as empty or non values

1. `null` is an empty value
2. `undefined` is a missing value

Or

1. `undefined` hasn't had a value yet
2. `null` had a value and doesn't anymore

Regardless of how you choose to _define_ and use these two values:

1. `null` is a special keyword, not an identifier, thus you cannot treat it as a variable to assign to
2. `undefined` is an identifier

##### Undefined

In `non-strict` mode it's possible to assign a value to the globally provided `undefined` identifier

```js
function foo() {
  undefined = 2; // really bad idea
}
foo();

function foo() {
  'use strict'
  undefined = 2; // TypeError!
}
foo();
```

In both strict modes you can create a local variable of the name `undefined`, _terrible idea_

```js
function foo() {
  'use strict'
  var undefined = 2;
  console.log(undefined); // 2
}
foo();
```

##### void operator

Another way to get `undefined` value is with the `void` operator

The expression `void ____` "voids" out any value, so that the result of the expression is always the `undefined` value.
    - It doesn't modify the existing value
    - It just ensures that no value comes back from the operator expression

```js
var a = 42;
console.log( void a, a ); // undefined 42
```

By convention (from C-language) to represent `undefined` value standalone by using `void` you use `void 0`, but there's no practical difference between

  > void 1
  > void 2
  > void true

`void` can be useful if you need to ensure that an expression has no result value

```js
function doSomething() {
  // note: APP.ready is provided by our application
  if(!APP.ready) {
    // try again later
    return void setTimeout( doSomething,100 );
  }

  var result;

  // do some other stuff
  return result
}
```

1. `setTimeout(..)` returns a numeric value, but we want to `void` that out so that the return value of our function doesn't give a false positive, there other way to express the same functionality

```js
if (!APP.ready) {
  // try again later
  setTimeout( doSomething,100 )
  return;
}
```

#### Special Numbers

Any mathematic operation you perform without both operands being `numbers` (or values that can be interpreted as regular `numbers` in base 10 or 16) will result in the operation failing to produce a valid `number` in which case you will get the `NaN` value

`NaN` stands for not a number, this label is very poor and misleading

```js
var a = 2 / "foo"; // NaN
typeof a === "number"; // true
```

- The type of 'not a number' is number

`NaN` is a very special value, it's never equal to another `NaN` value (it's never equal to itself) so `NaN !== NaN`

We test for a `NaN` number with `isNan(...)`, but this has a fatal flaw. It test if the thing passed is either not a `number` or a `number`

```js
var a = 2 / "foo";
var b = "foo";

a; // NaN
b; "foo"

window.isNaN( a ); // true
window.isNaN( b ); // true--ouch!
```

1. This bug is 19 years old
2. ES6 added a utility to solve this with `Number.isNaN(...)`

### Value Versus Reference

In many languages, values can either be assigned/passed by value-copy or by reference-copy depending on the syntax you use

In JS there are no pointers, and reference work a bit differently, you cannot have a reference from one JS variable to another variable

A reference in JS _points_ at a _shared_ value, so if you have 10 different references, they are all always distinct references to a single shared value, _none of them are references/pointers to each other_

In JS the _type_ of the value controls whether that value will be assigned by value-copy or by reference-copy

```js
var a = 2;
var b = a; // `b` is always a copy of the value in `a`
b++;
a; // 2
b; // 3

var c = [1,2,3];
var d = c; // `d` is a reference to the shared `[1,2,3]` value
d.push( 4 );
c; // [1,2,3,4]
d; // [1,2,3,4]
```

1. Simple values (aka scalar primitives) are _always_ assigned/passed by value-copy

    - `null`
    - `undefined`
    - `string`
    - `number`
    - `boolean`
    - `symbol`

2. Compound values, `objects` (including `arrays`, and all boxed object wrappers) and `function`s _always_ create a copy of the reference on assignment or passing

3. Because 2 is a scalar primitive, `a` holds one initial copy of that value, and `b` is assigned another copy of the value
    - When changing `b` you are not changing the value in `a`

4. Both `c` and `d` are separate references to the same shared value `[1,2,3]`, which is compound value, which is a compound value

    - Neither `c` or `d` "OWNS" the `array` value, both are equal references to the value
    - When modifying the `array`, it's affecting the shared value and both references will reference the newly modified value

Since references point to the values themselves and not to the variables, you cannot use one reference to change where another reference is pointer

```js
var a = [1,2,3];
var b = a;
a; // [1,2,3]
b; // [1,2,3]

// later
b = [4,5,6];
a; // [1,2,3]
b; // [4,5,6]
```

1. `b=[4,5,6]` we are not affecting where `a` is still referencing the first `array`, to do that `b` would have to be a pointer to `a` rather than a reference to the `array` but that capability do not exist on JS!

The most common way such confusion happens is with function parameters

```js
function foo(x) {
  x.push(4);
  x; // [1,2,3,4]

  // later
  x = [4,5,6];
  x.push(7);
  x; // [4,5,6,7]
}

var a = [1,2,3];
foo(a);
a; // [1,2,3,4] not [4,5,6,7]
```

1. When we pass in `a` it assign a copy of the `a` reference to `x`
2. `x` and `a` are separate references pointing at the same [1,2,3]
3. We can use that reference to mutate the value itself
4. When we make `x=[4,5,6]` we are NOT affecting where the initial reference `a` is pointing
5. There is no way to use the `x` reference to change where `a` is pointing. We could only modify the contents of the shared value that both `a` and `x` are pointing to

You cannot control/override value-copy versus reference, this is controlled by the type of the underlying value

To effectively pass a compound value (like an `array`) by value-copy, you need to manually make a copy of it, so that the reference passed doesn't still point to the original

```js
foo(a.slice());
```

1. `slice(..)` with no parameters by default makes an entirely new copy of the `array`

## 3. Natives
