# Review

1. __Scope__

    - Where and how a variable can be looked up
    - Functions are unit of scope, variables and functions declared inside another function are essentially hidden from any of the enclosing scopes
    - `{..}`, `try/catch` and `let` has block scope, attach themselves to the `{}`

    - _Hoisting_ -> Declarations and functions are "moved" to the top before executing the code. Assignments, and assignments of functions expressions are not hoisted

    - __Closure__ is when a function can remember and access its lexical scope even when it's invoked outside its lexical scope

        ```js
        function foo() {
          var a = 2;

          function bar() {
            console.log(a)
          }

          return bar
        }

        var baz = foo();

        baz(); // 2 -> Closure!!
        ```

2. __this__

    1. Called with `new` -> Use newly created object
    2. Called with `call` or `apply` or `bind`. use the specified object
    3. Called with a context object owning the call, use that context object
    4. Default: `undefined` in `strict mode` `global` object otherwise
