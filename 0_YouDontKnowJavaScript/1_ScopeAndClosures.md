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
