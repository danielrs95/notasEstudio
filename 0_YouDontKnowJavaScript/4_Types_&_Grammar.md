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

A list of the most commonly used natives

- `String()`
- `Number()`
- `Boolean()`
- `Array()`
- `Object()`
- `Function()`
- `RegExp()`
- `Date()`
- `Error()`
- `Symbol()`

These natives, are actually built-in functions

`String()` look like the `String(...)` constructor from JAVA, you can do things like

```js
var s = new String("Hello world");
console.log(s.toString()); // "Hello world"
```

It is true that each of these natives can be used as a native constructor. But, what's being constructed may be different than you think

```js
var a = new String("abc");

typeof a; // "object"

a instanceof String; // true

Object.prototype.toString.call(a); // "[Object String]"
```

1. The result of the constructor form of value creation (`new String("abc")`) is an object wrapper around the primitive (`"abc"`)

2. `typeof` shows that these objects are not their own special types, they are subtypes of the `object` type

3. `new String("abc")` creates a string wrapper object around `"abc"`, not just the primitive `"abc"` value itself

### Internal `[[Class]]`

Values that are `typeof` of object are additionally tagged with an internal `[[Class]]` property (like an internal classification).

This property cannot be accessed directly, but can generally be revealed indirectly by borrowing the default `Object.prototype.toString(...)` against the value

```js
Object.prototype.toString.call([1,2,3]); // "[object Array]"
Object.prototype.toString.call(/regex-literal/i); // "[object RegExp]"
```

In most cases, this internal `[[Class]]` value corresponds to the built-in native constructor that's related to the value, but that's not always the case

```js
Object.prototype.toString.call(null); // "[object Null]"
Object.prototype.toString.call(undefined); // "[object Undefined]"
```

There are no `Null()` or `Undefined()` native constructors, but "`Null`" and "`Undefined`" are the internal `[[Class]]` values exposed

For the other simple primitives like `string`, `number` and `boolean`, another behavior actually kicks in, usually called "boxing"

```js
Object.prototype.toString.call("abc"); // "[object String]"
Object.prototype.toString.call(42); // "[object Number]"
Object.prototype.toString.call(true); // "[object Boolean]"
```

1. Each of the simple primitives are automatically boxed by their respective object wrappers

### Boxing Wrappers

Primitive values don't have properties or methods, so to access `.length` or `.toString()` you need an object wrapper around the value

JS automatically box the primitive value

```js
var a = "abc";

a.length; // 3
a.toUpperCase(); // ABC
```

In general there's basically no reason to use the object form directly, it's better to just let the boxing happen implicitly where necessary

### Unboxing

If you have an object wrapper and you want to get the underlying primitive value out, you can use the `valueOf()` method

```js
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

Unboxing can also happen implicitly, when using an object wrapper value in a way that requires the primitive value (Coercion)

```js
var a = new String( "abc" );
var b = a + ""; // `b` has the unboxed primitive value "abc"

typeof a; // object
typeof b; // string
```

### Natives as Constructors

#### Array(...)

```js
var a = new Array( 1, 2, 3 );
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

1. The `Array(..)` constructor does not require the `new` keyword in front of it

2. Never use Array with a single argument, this creates an array of that size with "empty" slots but it's really buggy

#### Date(..) and Error(..)

The `Date(..)` and `Error(..)` native constructors are much more useful, because there is no literal form for either

To create a date object value, you mues use `new Date()`

The `Error()` constructor behaves the same with the `new` keyword present or omitted

The main reason you'd want to create an error object is that it captures the current execution stack context into the object. This stack context includes the function call stack and the line number where the error object was created

You would tipically use such an error object with the `throw` operator

```js
function foo(x) {
  if (!x) {
    throw new Error("x don't exist");
  }
};
```

1. Error object instances generally have at least a `message`
2. It's usually best to just call `toString()` on the error object to get a friendly formatted error message

#### Symbol(...)

Symbols are special "unique" values that can be used as properties on objects with little fear of any collision

Symbols can be used as property names, but you cannot see or access the actual value of a symbol from your program, nor from the developer console

To define your own custom symbols, use the `Symbol(..)` native, this constructor is unique, you're not allowed to use `new` with it

```js
var mysym = Symbol( "my own symbol" );
mysym; // Symbol(my own symbol)
mysym.toString(); // "Symbol(my own symbol)"
typeof mysym; // "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

#### Native Prototypes

Each of the built-in native constructors has its own `.prototype` object
These objects contain behavior unique to their particular object subtype

## 4. Coercion

### Converting Values

Converting a value from one type to another is called _type casting_ when done explicitly and _coercion_ when done implicitly

JS coercions always results in one of the scalar

```js
var a = 42;
var b = a + ""; // implicit coercion
var c = String( a ); // explicit coercion
```

### Abstract Value Operations

We need to learn the basic rules that govern how values become either a `string`, `number` or `boolean`.

ES5 defines several abstract operations with the rules of value conversion

#### ToString

When any `non-string` value is coerced to a `string` representation, the conversion is handled by the `ToString` abstract operation in section 9.8 of the specification

Built-in primitive values have natural stringification

- `null` becomes null
- `undefined` becomes undefined
- `true` becomes true
- `numbers` are generally expressed in the natural way you'd expect
  - Very small or large numbers are represented in exponent form

- For regular objects, unless you specify your own, the default `toString()` (located in `Object.prototype.toString()`) will return the internal `[[Class]]`

- Arrays have an overridden default `toString()` that stringifies as the (string) concatenation of all its values with `,` in between each value

    ```js
    var a = [1,2,3];
    a.toString(); // "1,2,3"
    ```

##### JSON stringification

Another task that seems awfully related to `ToString` is when you use the  `JSON.stringify(...)` utility to serialize a value to a JSON-compatible `string` value

This stringification is not exactly the same thing as coercion, but it's related to th `ToString` rules above

For most simple values, JSON stringification behaves basically the same as `toString()` conversions, except that the serialization result is _always_ a string

```js
JSON.stringify( 42 );// "42"
JSON.stringify( "42" ); // ""42"" (a string with a
                      // quoted string value in it)
JSON.stringify( null ); // "null"
JSON.stringify( true ); // "true"
```

Any _JSON-safe_ value can be stringified by `JSON.stringify(..)`. _JSON-safe_ is any value that can be represented validly in a JSON representation

Values that are not JSON-safe `undefined`, `function`, `symbol` and `object` with circular references

`JSON.stringify(..)` will automatically omit `undefined`, `function`, and `symbol` values when it comes across them.

If such a value is found in an `array`, that value is replaced by `null`
If found as a property of an `object` that property will simply be excluded

```js
JSON.stringify( undefined ); // undefined
JSON.stringify( function(){} ); // undefined

JSON.stringify(
  [1,undefined,function(){},4]
); // [1,null,null,4]

JSON.stringify(
  { a:2, b:function(){} }
);// {"a":2}
```

JSON stringification has the special behavior that if an `object` value has a `toJSON()` method defined, this method will be called first to get a value to use for serialization

```js
var o = { };

var a = {
  b: 42,
  c: o,
  d: function(){}
};

// create a circular reference inside `a`
o.e = a;

// would throw an error on the circular reference
// JSON.stringify( a );

// define a custom JSON value serialization
a.toJSON = function() {
  // only include the `b` property for serialization
  return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

1. `toJSON()` should be interpreted as to a JSON-safe value suitable for stringification, not to a JSON string

An optional second argument can be passed to `JSON.stringify(..)` that is called _replacer_ and can be an `array` or a `function`
    - It's used to customize the recursive serialization of an `object` by providing a filtering mechanism for which properties should and should not be included

If _replacer_ is an `array` it should be an array of `strings`, each of which will specify a property name that is allowed to be included in the serialization of the `object`. If a property exists that isn't in this list, it will be skipped

If _replacer_ is a `function`, it will be called once for the `object` itself, and then once for each property in the `object`. Each time is passed 2 arguments, _key_ and _value_. To skip a _key_ in the serialization, return `undefined`

```js
var a = {
  b: 42,
  c: "42",
  d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
  if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

1. In the `function` replacer case, the key argument `k` is `undefined` for the first call (where the `a` object itself is being passed)

2. The `if` statement filters out the property `c`

3. Stringification is recursive, so the [1,2,3] array  has each of its values passed as `v` to _replacer_ with indexes (0,1,2) as `k`

A third optional argument can also be passed to `JSON.stringify(..)` called _space_, which is used as indentation for prettier human-friendly

1. Can be a positive integer to indicate how many space characters should be used at each indentation level
2. Can be a `string` in which case up to the first 10 characters of its value will be used for each indentation level

```js
var a = {
  b: 42,
  c: "42",
  d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//   "b":42,
//   "c":"42",
//   "d": [
//     1,
//     2,
//     3
//   ]
// }"

JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

#### ToNumber

Follows ES5 spec section 9.3

- `true` becomes 1
- `false` becomes 0
- `undefined` becomes `NaN`
- `null` becomes 0

For `string` value essentially works for the most part like the rules/syntax for numeric literals. If it fails, the result is `NaN`

Objects (and arrays) will first be converted to their primitive value equivalent, and the resulting value is coerced to a `number` according to the rules

To convert to this primitive value equivalent, the `ToPrimitive` abstract operation will consul the value in question using the internal `DefaultValue` to see if it has a `valueOf()` method
    - If `valueOf()` is available and it returns a primitive value, that value is used for the coercion
    - If not, `toString()` will provide the value for the coercion, if present
    - If neither operation can provide a primitive value, a `TypeError` is thrown

```js
var a = {
  valueOf: function(){
    return "42";
  }
};

var b = {
  toString: function(){
    return "42";
  }
};

var c = [4,2];

c.toString = function(){
  return this.join( "" ); // "42"
};

Number( a ); // 42
Number( b ); // 42
Number( c ); // 42
Number( "" ); // 0
Number( [] ); // 0
Number( [ "abc" ] ); // NaN
```

#### ToBoolean

JS has keywords `true` and `false`, and the behave like `boolean` values

It's a common misconception that the values `1` and `0` are identical to `true`/`false`. You can coerce `1` to `true` and viceversa. But they're not the same

##### Falsy values

All of JS values can be divided into two categories

1. Values that will become `false` if coerced to `boolean`
2. Everything else

The falsy values are

1. `undefined`
2. `null`
3. `false`
4. `+0`, `-0` and `NaN`
5. `""`

JS doesn't define a "truthy" list per se, the spec implies that _anything not explicitly on the falsy list is therefore truthy_

##### Falsy objects
