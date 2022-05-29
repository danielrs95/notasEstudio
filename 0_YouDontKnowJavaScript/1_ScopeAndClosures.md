# Scopes & Closures

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables and later retrieve or modify those values

- Where do those variables live?
- How does our program find them when it needs them?

These questions speak to the need for a well-defined set of rules for storing variables in some location, and for finding those variables at a later time. We'll call that set of rules _scope_

________________________________________________________________________
________________________________________________________________________

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

________________________________________________________________________
________________________________________________________________________

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

________________________________________________________________________
________________________________________________________________________

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

________________________________________________________________________
________________________________________________________________________

## Lexical Scope

The first traditional phase of a standard language compiler is called lexing (aka tokenizing).

Lexing examines a string of source code characters and assigns semantic meaning to the tokens as a result of some parsing

Lexical scope is scope that is defined at lexing time, in other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code
