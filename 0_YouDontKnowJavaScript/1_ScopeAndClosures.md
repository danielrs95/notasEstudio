# Scopes & Closures

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables and later retrieve or modify those values

- Where do those variables live?
- How does our program find them when it needs them?

These questions speak to the need for a well-defined set of rules for storing variables in some location, and for finding those variables at a later time. We'll call that set of rules _scope_

____________________________________________________________________
____________________________________________________________________

## Compiler theory

Javascript is in fact a compiled language

- It is not compiled well in advance as many traditionally compiled languages
- Nor are the results of compilation portable among various distributed systems

In traditional compiled-languages process your code will undergo typically 3 steps _before_ it is executed, roughly called "compilation"

1. _Tokenizing/Lexing_

    Breaking up a string of characters into meaningful (to the language) chunks, called tokens.

    For instance consider the program `var a=2` this will probably be broken into

    - var
    - a
    - =
    - 2
    - ;

    The difference between tokenizing and lexing centers on whether or not these tokens are identified in a stateless or stateful way

    Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether `a` should be considered a distinct token or just part of another token, that would be `lexing`

2. _Parsing_

    Takings a stream(array) of tokens and turning it into a tree of nested elements, which collectively represents the grammatical structure of the program. This tree is called an _AST (abstract syntax tree)_

    - The tree for `var a = 2;` might start with a top-level node called `VariableDeclaration`
    - With a child node called `Identifier` whose value is a
    - And another child called `AssignmentExpression` which itself has a child called `NumericLiteral` whose value is 2

3. _Code generation_

    The process of taking an AST and turning it into executable code, varies depending on the language and the platform it's targeting

    We'll just handwave and say there's a way to take our previously described AST for `var a = 2;` and turn it into a set of machine instructions to actually create a variable called a (including reserving memory,etc) and then store a value into a

____________________________________________________________________
____________________________________________________________________

## Understanding Scope

One way to understand about scope is to think of the process in terms of a conversation about 3 people

1. _Engine_

    Responsible for start-to-finish compilation and execution of our JS program

2. _Compiler_

    One of Engine's friends. Handle all the dirty work of parsing and code-generation

3. _Scope_

    Collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code

### Back and Forth

Engine sees the code `var a = 2;` as two distinct statements

1. One that _Compiler_ will handle during compilation
2. One that _Engine_ will handle during execution

Lets break down how Engine and friends will approach the program `var a=2;`

1. _Compiler_ will perform _lexing_ to break it down to tokens and then parse into a tree (AST)

2. When _Compiler_ gets to code generation, it will treat this program somewhat differently

    A reasonable assumption would be that Compiler will produce code that could be summed up by this pseudocode:

    "Allocate memory for a variable, label it a, then stick the value 2 into that variable"

    That's not quite accurate, _Compiler_ will instead proceed as:

    1. Encountering `var a`, _Compiler_ asks _Scope_ to see if a variable `a` already exists for that particular scope collection.

        1. If so, _Compiler_ ignores this declaration and moves on
        2. Otherwise, _Compiler_ asks _Scope_ to declare a new variable called `a` for that scope collection

    2. _Compiler_ then produces code for _Engine_ to later execute, to handle the `a=2` assignment

        1. The code _Engine_ runs will first ask _Scope_ if there is a variable called `a` accessible in the current scope collection.

        If so, _Engine_ uses that variable. If not, Engine looks elsewhere

If _Engine_ eventually finds a variable, it assigns the value 2 to it. If not, _Engine_ will raise its hand and yell out an error

To summarize, two distinct actions are taken for a variable assignment:

1. _Compiler_ declares a variable (if not previously declared) in the current _Scope_
2. When executing, _Engine_ looks up the variable in _Scope_ and assigns to it, if found

### Compiler Speak

When _Engine_ executes the code that _Compiler_ produced it  has to look up the variable `a` to see if it has been declared, and this look-up is consulting _Scope_

The type of look-up _Engine_ performs affects the outcome of the look-up

In our case, it is said that _Engine_ would be performing an LHS look-up for the variable `a`. The other type of look-up is called RHS

L & R stand for lefthand side and righthand side of an _assignment operation_

An _LHS_ is done when a _variable_ appears on the lefthand side of an assignment
    An _LHS_ is trying to fin the _VARIABLE CONTAINER_ itself, so that it can assign the value to it

An _RHS_ is done when a _variable_ appears on the righthand side of an assignment
    - An _RHS_ is indistinguishable, from simply a look-up of the value of some variable
    - You could think _RHS_ means _go get the value of_

When we say `console.log(a)`

The reference to `a` is an _RHS_, because nothing is being assigned to `a` here. We are looking to retrieve the value of `a` and passed to `console.log(..)`

By contrast: `a=2`

The reference to `a` here is an _LHS_, we don't actually care what the current value is, we want to find the variable as a target for the `=2` assignment

It's better to conceptually think about this topic as:

_Who's the target of the assignment (LHS)?_
_Who's the source of the assignment (RHS)?_

Consider this program, which has both LHS and RHS references

```js
function foo(a) {
  console.log(a)
}

foo(2);
```

1. The last line that invokes `foo(..)` as a function call requires an _RHS_ reference to foo

    - Meaning, go look up the value of `foo` and give it to me
    - `(..)` means the value of `foo` should be executed, so it'd better be a function

2. There is an implied `a=2`, this happens when the value `2` is passed as an argument to the `foo(..)` function

    - The 2 value is _assigned_ to the parameter `a`. To implicitly assign to parameter `a`, an _LHS_ look-up is performed

3. As explained early, when we use the `console.log(a)` there is a _RHS_ look up

____________________________________________________________________
____________________________________________________________________

## Nested Scope

```js
  function foo(a) {
    console.log(a+b)
  }

  var b=2;

  foo(2);
```

The _RHS_ reference for `b` cannot be resolved inside the function `foo`, but it can be resolved in the scope surrounding it (in this case the global)

_Engine_ starts at the currently executing scope, looks for the variable there, then if not found, keep going up one level and so on

If the outermost global scope is reached, the search stops, whether if finds the variable or not

## Errors

```js
  function foo(a) {
    console.log( a + b );
    b = a;
  }

  foo( 2 );
```

When the _RHS_ occurs for `b` the first time, it will not be found.

This is said to be an _undeclared_ variable, because it is not found in the scope

If an _RHS_ fails to find a variable, anywhere in the nested scopes, this results in a `ReferenceError` being thrown by the _Engine_

It's important to note that the error is of the type _ReferenceError_

If the _Engine_ is performing an _LHS_ and it gets to the global scope without finding it, if the program is not in _Strict Mode_, the global scope will create a new variable of that name

If a variable is found for an _RHS_ but you try to do something with its value that's imposible, the _Engine_ throws a _TypeError_

_ReferenceError_ is scope resolution-failure related

_TypeError_ implies that scope resolution was successful, but that there was an illegal/imposible action attempted against the result

____________________________________________________________________
____________________________________________________________________

## Lexical Scope

The first traditional phase of a standard language compiler is called lexing (aka tokenizing).

Lexing examines a string of source code characters and assigns semantic meaning to the tokens as a result of some parsing

Lexical scope is scope that is defined at lexing time, in other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code

```js
  function foo(a) {
    var b = a * 2;

    function bar(c) {
      console.log(a,b,c)
    };

    bar( b * 3 );
  }

  foo(2) // 2, 4, 12
```

Existen 3 scopes anidados en esta función

![2](../images/YDKJS_1.png)

1. Bubble 1, encompasses the global scope and has just one identifier in it `foo`

2. Bubble 2, encompasses the scope of `foo`, which includes the three identifiers: `a`, `bar`, and `b`

3. Bubble 3, encompasses the scope of `bar`, and it includes just one identifier, `c`

- Scope bubbles are defined by where the blocks of scope are written, let's just assume that each function creates a new bubble of scope

- The nested bubbles are strictly nested, no bubble for some function can simultaneously exist inside two other outer scope bubbles

### Look-ups

The structure and relative placement of the scope bubbles fully explains to the engine all the places it needs to look to find an identifier

1. In the previous code, the _Engine_ executes the `console.log(..)` and goes looking for the three references variables `a,b and c`

    - First starts with the innermost scope bubble, the scope of `bar(...)`.
    - It won't find `a` there, so it goes up one level to the scope of `foo(...)`
    - It finds `a` there, same thing for `b`
    - It finds `c` inside of `bar(...)`

2. Had there been a `c` both inside of `bar(..)` and inside of `foo(..)`, the `console.log(..)` statement would have found and used the one in `bar(..)` never getting to `foo(..)

3. _Scope look-up stops once it finds the first match_.

    The same identifier name can be specified at multiple layers of nested scope, which is called _shadowing_ (the inner identifier, shadows the outer)

4. Regardless of shadowing, scope look-up always starts at the innermost scope being executed at the time, and works its way out/up until the first match, and stops

5. Global variables are automatically also properties of the global object (window in browsers, etc)

    - It's possible to reference a global variable not directly by its lexical name, but instead as a property reference of the global scope

    `window.a`

    This technique gives access to a global variable that would be otherwise be inaccessible due to it being shadowed

6. No matter where a function is invoked from, or how, it's lexical scope is only defined by where the function was declared

7. The lexical scope look-up only applies to first-class identifiers such as `a`, `b`, and `c`

    - If theres a reference to `foo.bar.baz`, the lexical scope look-up would apply to finding the `foo` identifier
    - Once `foo` is located, object property-access rules take over to resolve the `bar` and `baz` properties

## Cheating Lexical

JS has 2 mechanisms to cheat lexical scope, but this leads to poorer performance

### eval

The `eval(...)` function in JS takes a string as an argument and treats the contents of the strings as if it had actually been authored code at that point in the program

```js
  function foo(str, a) {
    eval (str); // CHEATING
    console.log( a,b )
  }

  var b = 2;

  foo('var b = 3;', 1); // 1, 3
```

1. The string `"var b=3;"` is treated at the point of the `eval(..)` call as code that was there all along

    - Because this code declares a new variable `b=3`, it modifies the existing lexical scope of `foo(..)`
    - The variable `b` created by `eval(..)` shadows the outer `b=2`

2. `eval(..)` when used in a strict-mode program operates in its own lexical scope, which means declarations made inside of the `eval(..)` do not actually modify the enclosing scope

The other frowned-upon (and now deprecated!) feature in JS that cheats lexical scope is the `with` keyword

### Performance

Both `eval()` and `with` cheat the lexical scope, and they completely cancel the optimization that JS does during the compilation phase and there is no getting around the fact that without the optimizations, code runs slower

____________________________________________________________________
____________________________________________________________________

## Function Versus Block Scope

Scope consists of a series of bubbles, that each act as a container, in which identifiers (variables, functions) are declared

But what exactly makes a new bubble? Is it only the function?

### Scope From Functions

JS has function-based scope, each function you declare creates a bubble for itself

```js
  function foo(a) {
    var b = 2;

    // some code

    function bar() {
      // ....
    }

    var c = 3;
  }
```

- The scope bubble for `foo(...)` includes identifiers `a`, `b`, `c`, and `bar`. It doesn't matter where in the scope a declaration appears, the variable or function belongs to the containing scope bubble

- `bar(..)` has its own scope bubble, so does the global scope, which has just one identifier attached to it `foo`

- Function scope encourages the idea that all variables belongs to the function, and can be used an reused throughout the entirety of the function (and indeed, accessible even to nested scope)

  - This design approach can be quite useful, and certainly can make full use of the dynamic nature of JS variables to take on values of different types as needed

  - If you don't take careful precautions, variables existing across the entirety of a scope can lead to some unexpected pitfalls

### Hiding in Plain Scope

The software design principle _Principle of Least Privilege_, also sometimes called Least Authority or Least Exposure

This principle states that in the design of software, such as the API for a module/object you should expose only what is minimally necessary and hide everything else

```js
  function doSomething(a) {
    b = a + doSomethingElse(a*2)
    console.log(b*3)
  }

  function doSomethingElse(a) {
    return a-1
  }

  var b;
  doSomething(2) //15
```

- `b` and `doSomethingElse(..)` are likely private details of how `doSomething(..)` does its job

- Giving the enclosing scope access to `b` and `doSomethingElse(...)` is unnecessary and dangerous as they may be used in unexpected ways

```js
  function doSomething(a) {
    function doSomethingElse(a) {
      return a-1
    }

    var b;
    b = a + doSomethingElse(a*2)
    console.log(b*3)
  }

  doSomething(2) // 15
```

- `b` and `doSomethingElse(...)` are not accessible to any outside influence

### Global namespaces

An example of likely variable collision occurs in the global scope, multiple libraries loaded into your program can quite easily collide with each other

Such libraries typically will create a single variable declaration, often an object, with a sufficiently unique name in the global scope

This object is then used as a namespace for that library, where all specific exposures of functionality are made as properties off that object(namespace), rather than as top-level lexically scoped identifiers

```js
  var MyLibrary = {
    awesome: "stuff",
    doSomething: function() {
      //...
    },
    doAnother: function() {
      //...
    }
  }
```

### Module management

Another option for collision avoidance is the more modern `module` approach, using any of various dependency managers, using these tools, no libraries ever add any identifiers to the global scope, but are instead required to have their identifiers be explicitly imported into another specific scope through usage of the dependency managers various mechanisms

____________________________________________________________________
____________________________________________________________________

## Function as Scopes

We can take any snippet of code and wrap a function around it, and that effectively hides any enclosed variable or function declaration from the outside scope inside that functions inner scope

```js
  var a=2

  function foo() {
    var a=3
    console.log(a) // 3
  }

  foo()
  console.log(a) // 2
```

This technique works, it is not necessarily very ideal

1. We have to declare a named-function `foo()`, which means that the identifier `foo` itself pollutes the enclosing scope (global in this case)

2. We have to explicitly call the function by name (`foo()`) so that the wrapped code actually executes

3. It would be more ideal if the function didn't need a name (or rather, the name didn't pollute the enclosing scope) and if the function could automatically be executed

```js
  var a=2

  (function foo() {
    var a=3
    console.log(a) // 3
  })() // Immediately Invoked Function Expression

  console.log(a) // 2
```

1. With the Immediately Invoked Function Expression, instead of treating the function as a standard declaration,the function is treated as a function expression

    - In the first snippet, the name `foo` is bound in the enclosing scope and we call it directly with `foo()`

    - In the second snipper, the name `foo` is not bound in the enclosing scope but instead is bound only inside of its own function

        - `(function foo() {...})` as an expression means the identifier `foo` is found _only_ in the scope where the `...` indicates, not in the outer scope

        - Hiding the name `foo` inside itself means it does not pollute the enclosing scope unnecessarily

### Anonymous Versus Named

Function expressions are most often used as callback parameters as

```js
setTimeOut( function() {
  console.log("Wait")
}, 1000);
```

This is called an anonymous function expression because `function()...` has no name identifier on it, however they have several drawbacks to consider

1. Have no useful name to display in stack traces, make debugging harder

2. Without a name, if the function needs to refer to itself, for recursion, etc, the deprecated `arguments.callee` reference is required

    - Another example of self-reference is when an event handler function wants to unbind itself after it fires

3. A name is often helpful in providing more readable/understandable code, helps self-document the code in question

_Inline function expressions_ are powerful and useful, providing a name for your function expression addresses all these drawbacks and has no tangible downsides. The best practice is to always name your function expressions

```js
setTimeOut( function timeOutHandler() {
  console.log("Wait")
}, 1000)
```

### Invoking Function Expressions Immediately

```js
  var a=2

  (function foo() {
    var a=3
    console.log(a) // 3
  })() // Immediately Invoked Function Expression IIFE

  console.log(a) // 2
```

1. The first enclosing `()` pair makes the _function an expression_
2. The second enclosing `()` executes the function
3. Naming and IIFE has all the benefits over anonymous functions expressions

There is a slight variation on the traditional IIFE form, which some prefer

> (function(){...})()
> (function(){...}())

1. In the first form

    - The function expressions is wrapped in `()` and then the invoking `()` pair is on the outside right after it

2. In the second form

    - The invoking `()` pair is moved to the inside of the outer `()` wrapping pair

3. These 2 forms are identical in functionality, it's purely a stylistic choice

Another variation on IIFEs is to use the fact that they are just function calls and pass in arguments

```js
  var a=2

  (function IIFE(global) {

    var a=3
    console.log( a ) // 3
    console.log( global.a ) // 3

  })( window ) // Immediately Invoked Function Expression IIFE

  console.log(a) // 2
```

1. We pass the `window` object reference and name the parameter `global` so we have a clear delineation between global and non global references

Another variation of the IIFE inverts the order of things

- The function to execute is given _second_, after the invocation and parameters to pass it

- This pattern is used in the UMD (Universal Module Definition) project

```js
  var a=2

  (function IIFE(def){
    def(window)
  })(function def(global){

    var a=3
    console.log(a) // 3
    console.log(global.a) // 2
  })
```

1. The `def` _function expression_ is defined in the second-half of the snippet and then passed as a _parameter_ (also called def) to the IIFE function defined in the first half of the snippet

2. The parameter `def` (the function) is invoked, passing `window` as the `global` parameter

## Blocks as Scopes

When considering blocks on JS, they usually declare variables as global (pre ES6 when we had no `let` and `const`), on the surface JS has no facility for block scope until you dig a little further

### with

`with` is an example of (a form of) block scope, in that the scope that is created from the object only exists for the lifetime of that `with` statement and not in the enclosing scope

### try/catch

JS in ES3 specified the variable declaration in the _catch_ of a `try/catch` to be block-scoped to the _catch_ block

```js
try {
  undefined();
} catch (err){
  console.log(err) //works
}

console.log( err ) // ReferenceError
```

- `err` exist only in the `catch` and throws an error when you try to reference it elsewhere
- Many linters still complain if you have 2 or more `catch` clauses in the same scope that each declare their error variable with the same identifier name

  - This is not a redefinition, since variables are safely block-scoped
  - To avoid this unnecessary warnings, call them `err1`,`err2`, etc

### let

ES6 introduces a new keyword `let` which sits alongside `var` as another way to declare variables

`let` attaches the variable declaration to the scope of whatever block (commonly a `{...}` pair) it's contained in

```js
  var foo = true

  if (foo) {
    let bar = foo * 2
    bar = something(bar)
    console.log(bar)
  }

  console.log(bar) // ReferenceError
```

Creating explicit blocks for block-scoping can make more obvious where variables are attached and not

Usually, explicit code is preferable over implicit or subtle code, explicit block-scoping is easy to achieve and fits more naturally with how bloc-scoping works in other languages

```js
  var foo = true

  if (foo) {
    { // EXPLICIT BLOCK
      let bar = foo * 2
      bar = something(bar)
      console.log(bar)
    }
  }

  console.log(bar) // ReferenceError
```

Hoisting is about declarations being taken as existing for the entire scope in which they occur

Declarations made with `let` will not hoist to the entire scope of the block they appear in. Such declarations will not observably _exists_ in the block until the declaration statement

```js
{
  console.log(bar) // ReferenceError
  let bar = 2
}
```

#### Garbage collection

Another reason block-scoping is useful relates to closures and garbage collection to reclaim memory, consider

```js
function process(data) {
  // do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
}, /*capturingPhase=*/false );
```

1. The `click` function click handler callback doesn't need the `someReallyBigData` variable at all

    - That means, after `process(..)` runs, the big memory-heavy, data structure could be garbage collected

    - It's quite likely that the JS engine will still have to keep the structure around since the `click` function has a closure over the entire scope

    - Block-scoping can address this concern, making it clearer to the engine that it does not need to keep `someReallyBigData` around

```js
function process(data) {
  // do something interesting
}

// Anything declared inside this block can go away after
{
  var someReallyBigData = { .. };
  process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
}, /*capturingPhase=*/false );
```

#### let loops

```js
for (let i=0; i<10; i++) {
  console.log(i)
}

console.log(i) // ReferenceError
```

- `let` in the `for` loop header bind the `i` to the `for` loop body
- It rebinds it to each iteration of the loop, making sure to reassign it the value from the end of the previous loop iteration

Here's another way of illustrating the per-iteration binding behavior that occurs

```js
{
  let j;
  for (j=0; j<10; j++) {
    let i=j; // re-bound for each iteration!!
    console.log(i)
  }
}
```

Because `let` declarations attach to arbitrary blocks rather than to the enclosing functions scope (or global) there can be gotchas where existing code has a hidden reliance on function-scoped `var` declarations and replacing `var` with `let` may require additional care

```js
  var foo = true, baz = 10;

  if (foo) {
    var bar = 3;

    if (baz > bar) {
      console.log( baz )
    }

    // ...
  }
```

This code is fairly easily refactored as

```js
var foo = true, baz = 10;

if (foo) {
  var bar = 3;
}

if (baz > bar) {
  console.log(baz)
}
```

But, be careful of such changes when using block-scoped variables

```js
var foo = true, baz=10;

if (foo) {
  let bar = 3;

  if (baz>bar) { // don't forget `bar` when moving
    console.log(baz)
  }
}
```

### const

In addition to `let`, ES6 introduces `const` which also creates a block-scoped variable, but whose value is fixed. Any attempt to change that value at a later time results in an error

```js
var foo = true;

if (foo) {
  var a = 2
  const b = 3 // block-scoped to the containing if

  a = 3
  b = 4 // error
}

console.log( a ) // 3
console.log( b ) // ReferenceError!
```

____________________________________________________________________
____________________________________________________________________

## 4. Hoisting

Variables are attached to different levels of scope depending on where and how they are declared

Function scope and block scope behave by the same rules in this regard: any variable declared within a scope is attached to that scope

But there's a subtle detail of how scope attachment works with declarations that appear in various locations within a scope

### Chicken or the Egg?

There's a temptation to think that all of the code you see in a JS program is interpreted line-by-line, top-down in order, as the program executes

While that is substantially true, there's one part of that assumption that can lead to incorrect thinking about your program

```js
a = 2;

var a;

console.log( a );
```

- Many would expect `undefined`, since the `var a` statement comes after the `a=2`. However, the output will be 2

```js
console.log( a );

var a = 2;
```

- In this case, `undefined` is the output

What's going on here? It would appear we have a chicken-and-the-egg question

Which come first, de declaration (egg) or the assignment (chicken)

### The Compiler Strikes Again

Part of the compilation phase is to find and associate all declarations with their appropriate scopes

The best way to think about things is that all declarations, both variables and functions, are processed first, before any part of the code is executed

When you see `var a = 2;` you see it as one statement, but JS actually thinks of it as two statements:

> var a;
> a = 2;

The first statement, the _declaration_ is processed during the _compilation phase_

The second statement, the _assignment_ is left in place for the _execution phase_

Going back to our first snippet, we should though of as being handled like this

```js
var a;
a = 2;
console.log(a)
```

Similarly, our second snipper

```js
var a;
console.log(a)
a = 2
```

Variables and functions _declarations_ are "moved" from where they appear in the flow of the code to the top of the code, this gives rise to the name _hoisting_

In other words the egg (_declaration_) comes before the chicken (_assignment_)

Only the declarations themselves are hoisted, while any assignments or other executable logic are left in place

```js
foo ();

function foo() {
  console.log( a ) // undefined
  var a = 2
}
```

The function `foo` declaration (which in this case includes the implied value of it as an actual function) is hoisted, such that the call on the first line is able to execute

Hoisting is per-scope, the `foo(...)` function exhibits that the `var a` is hoisted to the top of the definition. The program can perhaps be more accurately interpreted as

```js
function foo() {
  var a
  console.log( a ) // undefined
  a = 2
}

foo ();
```

Function declarations are hoisted, as we just saw. But function expressions are not

```js
foo() // not ReferenceError but TypeError

var foo = function bar() { ... }
```

1. The variable identifier `foo` is hoisted and attached to the enclosing scope (global) of this program

    - `foo()` doesn't fail as a `ReferenceError`
    - `foo` has no value yet (as it would if it had been a true function declaration instead of a expression)
    - `foo()` is attempting to invoke the `undefined` value, which is a `TypeError` illegal operation

2. Recall that even though it's a named function expression, the name identifier is not available in the enclosing scope

```js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
  // ...
}
```

This snippet is more accurately interpreted with hoisting as

```js
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
  var bar = ...self...
  //...
}
```

### Function First

Las declaraciones de funciones y variables son hoisted, es decir, movidas al inicio del código, pero hay un detalle, las funciones toman prioridad sobre las variables

```js
foo(); // 1
var foo;

function foo() {
  console.log(1)
}

foo = function() {
  console.log(2);
}
```

The engine interpreted the code as

```js
function foo() {
  console.log(1)
}

foo() // 1

foo = function() {
  console.log(2)
}
```

- `var foo` was a declaration duplicated and thus _ignored_
- Even though `var foo` was before the `function foo()` declaration, function declarations take priority

While multiple/duplicate `var` declarations are effectively ignored, subsequent function declarations _override_ previous ones

```js
foo(); // 3

function foo() {
  console.log(1)
}

var foo = function() {
  console.log(2);
}

function foo() {
  console.log(3)
}
```

This highlights the fact that duplicate definitions in the same scope are a really bad idea and will often lead to confusing results

Function declarations that appear inside of normal blocks typically hoist to the enclosing scope, rather than being conditional as this code implies

```js
foo(); // "b"
var a = true

if (a) {
  function foo() { console.log("a") }
} else {
  function foo() { console.log("b") }
}
```

## 5. Scope Closure

Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope

```js
function foo() {
  var a = 2;

  function bar() {
    console.log(a) // 2
  }

  bar()
}

foo()
```

1. This code should look familiar from our nested scope
    - Function `bar()` has access to `a` because of lexical scope look-up rules

2. Is this closure? technically, perhaps. The most accurate way to explain `bar()` referencing `a` is via lexical scope look-up rules, and those rules are only an important part of what closure is

3. From a purely academic perspective, what is said of the above snipper is that the function `bar()` has a _closure_ over the scope of `foo()` (and indeed, even over the rest of the scopes)

    - Put slightly differently, it's said that `bar()` _closes_ over the scope of `foo()`

    - Why? Because `bar()` appears nested inside `foo()`. Plain and simple

Closure defined in this way is not directly observable, nor do we se closure exercised in that snippet, we see lexical scope

```js
function foo() {
  var a = 2;

  function bar() {
    console.log(a) // 2
  }

  return bar;
}

var baz = foo()
baz(); // Closure was just observed!
```

1. The function `bar()` has lexical scope access to the inner scope of `foo()`

2. We take `bar()`, the function itself and pass it as a value, we _return_ the function object itself that `bar` references

3. After we execute `foo()` we assign the value it returned (the `bar()` function) to a variable called `baz`

    - Then we invoke `baz()` which of course is invoking our inner function `bar()`

4. `bar()` is executed, but in this case, executed _outside_ of its declared lexical scope

5. After `foo()` executed, normally we would expect that the entirety of the inner scope of `foo()` go away because of the garbage collector

6. The "magic" of closures does not let this happen, the inner scope is in fact _still_ in use, and does not go away. `bar()` is still using it

7. By virtue of where it was declared, `bar()` has a lexical scope closure over the inner scope of `foo()` which keeps that scope alive for `bar()` to reference at any later time

8. `bar()` still has a reference to that scope, and that reference is called _closure_
Pero
9. When `baz` is invoked (thus invoking `bar()`) it has access to author-time lexical scope, so it can access `a`

10. The function is being invoked well outside of its author-time lexical scope. _Closure_ lets the function continue to access the lexical scope it was defined in at author time

Any of the various ways that functions can be passed around as values, and indeed invoked in other locations, all are _examples_ of observing/exercising closure

```js
function foo() {
  var a = 2;

  function baz() {
    console.log(a); // 2
  }

  bar(baz);
}

function bar(fn) {
  fn(); // Exercising the closures
}
```

1. We pass the inner function `baz` over to `bar`
2. Call that inner function (labeled now `fn`)
3. When we do the call, it's closure over `foo()` is observed by accessing `a`

Passing can be indirect

```js
var fn

function foo() {
  var a = 2;

  function baz() {
    console.log(a)
  }

  fn = baz; // Assign baz to global variable
}

function bar() {
  fn(); // look ma, I saw closure
}

foo();
bar(); // 2
```

However we transport an inner function outside of its lexical scop, it will maintain a scope reference to where it was originally declared and wherever we execute him, that closure will be exercised

### Examples

```js
function wait(message) {

  setTimeout( function timer() {
    console.log(message);
  }, 1000);
}

wait("Hello, closure!");
```

1. We take an inner function (`timer`) and pass it to `setTimeout(..)`
2. `timer` has a scope closure over the scope of `wait(..)` keeping and using a reference to the variable `message`

3. 1000 milliseconds after we have executed `wait(..)` and its inner scope should otherwise be gone, that anonymous function still has closure over that scope

4. Deep in the engine

    - The built-in utility `setTimeOut(...)` has reference to some parameter (fn, or func, or something like that)
    - Engine invokes that function which is invoking our inner `time` and the lexical scope is still intact

### Loops and Closure

The most common example used to illustrate closure involves `for` loop

```js
for (var i=1; i<=5; i++){
  setTimeout( function timer(){
    console.log(i)
  }, i*1000)
}
```

- The idea of this code is to print the number 1 to 5, one at a time, one per second, respectively

- If you run the code you get 6, printed five times at 1 time intervals

    1. The 6 comes from the terminating condition of the loop, when 6 > 5
    2. The timeout callback function (`timer`) are all running well after the completion of the loop

- We are trying to _imply_ that each iteration of the loop _captures_ it's own copy of `i` at the time of iteration

  - The way scope works, all five of those functions, though they are defined separately in each loop iteration
  - _Are closed over the same shared global scope_, in fact, only one `i` in it

- Something about loop structure tends to confuse us into _thinking_ there's something else more at work, there's no difference than if each of the five timeout callbacks were just declared one right after the other, with no loop at all

- We _NEED_ more closured scope, we need a new closured scope for each iteration of the loop with a IIFE, remember, IIFE creates scope by declaring a function an immediately executing it

```js
for (var i=1; i<=5; i++){
  (function (){
    setTimeout( function timer(){
      console.log(i)
    }, i*1000)
  })()
}
```

- This still doesn't work

  - Each timeout function callback is indeed closing over its own per-iteration scope created by each IIFE

  - It's not enough to have a scope to close over _if that scope is empty_

  - Our IIFE is just an empty do-nothing scope, it needs something in it to be useful of us

  - It needs its own variable, with a copy of the `i` value at each iteration

```js
for (var i=1; i<=5; i++){
  (function (){
    var j = i
    setTimeout( function timer(){
      console.log(j)
    }, j*1000)
  })()
}
```

A slight variation some prefer

```js
for (var i=1; i<=5; i++){
  (function (j){
    setTimeout( function timer(){
      console.log(j)
    }, j*1000)
  })(i)
}
```

- Since these IIFE are just function, we can pass in `i` and we can call it `j` if we prefer, or even `i` again

- The IIFE inside each iteration created a new scope for each iteration

  - This gave our timeout function callbacks the opportunity to close over a new scope for each iteration
  - This new scope had a variable with the right per-iteration value in it for us to access

#### Block Scoping Revisited

We used an IIFE to create a new scope per-iteration, we created a new scope per-iteration.

Remember `let` hijacks a block and declares a variable right there in the block

Essentially turns a block into a scope that we can close over, so the following code works

```js
for (var i=1; i<=5; i++) {
  let j=i
  setTimeout( function timer() {
    console.log(j)
  }, j*1000)
}
```

There's a special behavior for `let` declarations used in the head of a `for` loop.

The variable will be declared not just once, but _each iteration_. And it will be initialized at each subsequent iteration with the value from the end of the previous iteration

```js
for (let i=1; i<=5; i++) {
  setTimeout( function timer() {
    console.log(i)
  }, i*1000)
}
```

### Modules

There are other code patterns that leverage the power of closure but that do not on the surface appear to be about callbacks, like the _module_

```js
function CoolModule() {
  var something = 'cool';
  var another = [1,2,3];

  function doSomething() {
    console.log(something)
  };

  function doAnother() {
    console.log(another.join(" ! "));
  };
}
```

- We simply have some private data variables `something` and `another` and a couple of inner functions `doSomething()` and `doAnother()` which bot have lexical scope (and thus closure!) over the inner scope of `foo()`

```js
function CoolModule() {
  var something = 'cool';
  var another = [1,2,3];

  function doSomething() {
    console.log(something)
  };

  function doAnother() {
    console.log(another.join(" ! "));
  };

  return {
    doSomething: doSomething,
    doAnother: doAnother,
  };
}

var foo = CoolModule();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

- This pattern in JS is called _module_, the most common way of implementing is called _revealing module_ and it's the variation above

- `CoolModule()` is a function, but _it has to ve invoked_ for there to be a module instance created

- `CoolModule()` return an object, this object has references on it to our inner functions
  - But not to our inner data variables, we keep those hidden an private
  - It's appropriate to think of this object as a _public API for our module_
  - This object return value is ultimately assigned to the outer variable foo and then we can access those methods on the API

- It's not required that we return an actual object, we could just return back an inner function directly

- `doSomething()` and `doAnother()` have closure over the inner scope of the module instance

There are 2 requirements for the module pattern to be exercised

1. There must be an outer enclosing function and it must be invoked at least once (each time creates a new module instance)

2. The enclosing function must return back at least one inner function, so that this inner function has closure over the private scope and can access and/or modify that private state

An object with a function property on it alone is not a module

An object that is returned from a function invocation that only has data properties on it and no closured functions is not really a module

A slight variation of this pattern when you only care to have one instance, a singleton of sorts

```js
var foo = (function CoolModule() {
  var something = 'cool';
  var another = [1,2,3];

  function doSomething() {
    console.log(something)
  };

  function doAnother() {
    console.log(another.join(" ! "));
  };

  return {
    doSomething: doSomething,
    doAnother: doAnother,
  };
})()

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

- We turned the module function into an IIFE, immediately invoked it and assigned its return value directly to our single module instance identifier `foo`

Modules are just functions so they can receive parameters

```js
function CoolModule(id) {
  function identify() {
    console.log(id);
  }

  return {
    identify: identify
  }
}

var foo1 = CoolModule("foo1")
foo1.identify() // foo 1
```

Another slight but powerful variation on the module pattern is to name the object you are returning as your public API

```js
var foo = (function CoolModule(id) {
  function change() {
    // modifying the public API
    publicAPI.identify = identify2;
  }

  function identify1() {
    console.log( id );
  }

  function identify2() {
    console.log( id.toUpperCase() );
  }

  var publicAPI = {
    change: change,
    identify: identify1
  };

  return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

1. By retaining an inner reference to the public API object inside your module instance, you can modify that module instance _from the inside_, including adding and removing methods and properties, and changing their values

#### Modern Modules
