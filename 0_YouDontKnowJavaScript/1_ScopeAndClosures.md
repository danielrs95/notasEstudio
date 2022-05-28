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

When _Engine_ executes the code that _Compiler_ produced it  has to look up the variable `a` to see if it has been declared, and this is consulting _Scope_
