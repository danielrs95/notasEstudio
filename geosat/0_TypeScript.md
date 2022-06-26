# Typescript

1. Open source language, build on top of JavaScript
2. Add static type to the code
   1. Dynamically typed languages -> Types are associated with run-time values and not named explicitly on code
      1. Example: Java, C, C++, Rust, Go

   2. Statically typed language -> Explicitly assign types to variables
       1. JavaScript, Python

3. PROS:
   1. More Robust
   2. Easily spot bugs
   3. Predictability
   4. Readability
   5. Popular

4. CONS:
   1. Mode code to write
   2. More to learn
   3. Required compilation
   4. Not true static typing

The goal of TS is to be a static typechecker for JS programs, a tool that runs before your code runs (static) and ensures that the types of the program are correct (typechecked)

Handbook from TS page aims to :

1. Read and understand commonly-used TS syntax and patterns
2. Explain the effects of important compiler options
3. Correctly predict type system 1. behavior in most cases

When JS code is run, the way JS runtime chooses what to do is by figuring out the `type` of the value, what sorts of behaviors and capabilities it has.

For some values, such as the primitives `string` and `number` we can identify their type using the `typeof` operator, but for functions theres no corresponding runtime mechanism to identify their types

```ts
function fn(x) {
  return x.flip();
}
```

1. This function only work when x has the callable `flip` property
2. This error only surfaces when we actually call the function
3. This behavior is hard to predict

Seeing that way, a `type` is the concept of describing which values can ve passed to `fn` and which will crash

  1. Remember, JS only provides DYNAMIC TYPING, running the code to see what happens

## Static type-checking

Static types systems describe the shapes and behaviors of what our values will be when we run our programs, a type-checker like TS uses that information and tells us when thing might be going off the rails.

```js
const message = "hello!";

message();
// This expression is not callable.
//   Type 'String' has no call signatures.
```

## Non-Exception failures

There are some runtime errors defined on the ECMAScript specification, like trying to call something that isn't callable like `message()` on the past example

You could imagine that accessing a property that doesn't exist on an object should trow an error too, instead, JS gives us different behavior and return the value `undefined`

```js
const user = {
  name: "Daniel",
  age: 26,
}

user.location; // returns undefined
```

Ultimately, TS has to make the call over what code should be flagged as an error in its system, even if it's valid JS that won't throw an error immediately

```ts
const user = {
  name: "Daniel",
  age: 26,
};

user.location;
// Property 'location' does not exist on type '{ name: string; age: number; }'.
```

## Explicit Types

```ts
function greet(person:string, date:Date){
  console.log(`Hello ${person}, today is ${date.toDateString()}`);
}
```

We just added `type annotations` on `person` and `date` to describe what types of values `greet` can be called with

Can be read as: ----> `greet` takes a `person` of type `string` and a `date` of type `Date`

With this, TS can tell us when `greet` was called incorrectly

1. `Date()` in JS returns a `string`, and the second parameter was expecting a date

  ```ts
  function greet(person: string, date: Date) {
    console.log(`Hello ${person}, today is ${date.toDateString()}!`);
  }

  greet("Maddison", Date());
  // Argument of type 'string' is not assignable to parameter of type 'Date'.
  ```

We don't always have to write explicit type annotations, TS can infer the types

## Erased Types

When we compile the function `greet` with `tsc` we get

```ts
"use strict";
function greet(person, date) {
    console.log("Hello ".concat(person, ", today is ").concat(date.toDateString(), "!"));
}
greet("Maddison", new Date());
```

1. Our `person` and `date` parameters no longer have type annotations
2. Our `template string` that used backtick was converted to plain strings

Remember, type annotations aren't part of JS when we compile those annotations get erased -> Type annotations never change the runtime behavior of your program

## Downleveling

The other difference was that our template string was rewritten, this is because by default TypeScript targets ES3, and template strings are from ECMAScript2015 (aka ECMAScript 6, ES2015, ES6)

## Strictness

TypeScript has several type-checking strictness flags that can be turned on or off.

The `strict` flag in the CLI, or `"strict: true"` in a `tsconfig.json` toggles them all on simultaneously, the two biggest ones you should know about are `noImplicitAny` and `strictNullChecks`

### noImplicitAny

Falling back to `any` is the plain JS experience

The more typed your program is, the more validation and tooling you'll get, fewer bugs.

Turning `noImplicitAny` flag will issue an error on any variables whose type is implicitly inferred as `any`

### strictNullChecks

By default, `null` and `undefined` are assignable to any other type, forgetting to handle this cases is the cause of countless bugs

`strictNullChecks` flag makes handling `null` and `undefined` more explicit

## The primitives: `string`, `number`, and `boolean`

1. `string` represents string values
2. `number` is for numbers like, JS does not have a special runtime value for integers, there's no equivalent to `int` or `float`, everything is `number`
3. `boolean` is for the two values `true` and `false`

- The type names `String`, `Number`, and `Boolean` with capital letters are legal but refer to some special built-in types.

---

## Composing Types

With Ts you can create complex types by combining simple ones, there are 2 popular ways to do so: with unions and with generics

### Unions

You can declare that a type could be one of many types

1. A popular use-case for union types is to describe the ser of `string` or `number` literals that a value is allowed to be

  ```ts
  type WindowStates = "open" | "closed" | "minimized";
  type LockStates = "locked" | "unlocked";
  type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;
  ```

  1. Unions provide a way to handle different types too

  ```ts
  function getLength(obj: string | string[]) {
    return obj.length;
  }
  ```

## Generics

1. The identity function will return whatever is passed in

    ```js
    function identity(arg: number): number {
      return arg;
    }

    function identity(arg: any): any {
      return arg;
    }
    ```

2. Using `any` is generic, but we lose information
   1. If we pass a number, the only info we have is that any type could be returned

3. We need to capture the type of the argument and use it to denote what is being returned
   1. We will use a `type variable`, a special kind of variable that works on types rather than values

    ```js
    function identity<Type>(arg: Type): Type {
      return arg;
    }
    ```

4. We added a type variable `Type`
   1. This allows us to capture the type the user provides (e.g. `number`)
   2. We use `<Type>` again as the return type
   3. This function is generic, as it works over a range of types.

```ts
// Explicitly set TYPE to be a string
let output = identity<string>("myString")
```

### Working with generic type variables

When you start using generics, you'll notice that when you create generic functions like `identity` the compiler will enforce that you use any generically typed parameters in the body of the function correctly, you actually treat these parameters as if they could be any and all types

```js
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
// Property 'length' does not exist on type 'Type'.
  return arg;
}
```

1. `Type` stands for any and all types, so a `number` variable does not have a `.length`

2. Lets say we've actually intended this function to work on arrays of `Type` rather than `Type` directly

```ts
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
```

1. `loggingIdentity` is a generic function that
   1. takes a type parameter `Type`
   2. an argument `arg` which is an array of `Type`s and returns an array of `Type`s

    <!-- Can also be written as: -->
    ```ts
      function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
      console.log(arg.length); // Array has a .length, so no more error
      return arg;
    }
    ```

## Generic types

1. We created generic identity functions that worked over a range of types
2. Lets explore the type of the functions themselves

3. The type of generic functions is just like those of non-generic functions, with the type parameters listed first

```js
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Type>(arg: Type) => Type = identity;
```
