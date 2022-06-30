# Async & Performance

## 1. Asynchrony: Now & Later

### A Program in Chunks

Your program is almost certainly comprised of several chunks, only one of which is going to execute _now_ and the rest will execute _later_

The most common unit of _chunk_ is the `function`

1. _later_ doesn't happen strictly and immediately after _now_
    - Tasks that cannot complete _now_, by definition, going to complete asynchronously, and thus, we will not have blocking behavior as you might expect

```js
// ajax(..) is some arbitrary Ajax function given by a library
var data = ajax( "http://some.url.1" );

console.log( data );
// Oops! `data` generally won't have the Ajax results
```

1. Ajax request don't complete synchronously
    - `ajax(..)` function does not yet have a value to return to be assigned to `data`
    - If `ajax(..)` _could_  block until the response, then the assignment would work

2. We make an asynchronous Ajax request _now_ and we won't get the result back until _later_

A way of _waiting_ from _now_ until _later_ is to use a function, commonly called a _callback_ function

```js
// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", function myCallbackFunction(data){

  console.log( data ); // Yay, I got's me some `data`!

} );
```

Another example

```js
function now() {
  return 21;
}

function later() {
  answer = answer * 2;
  console.log( "Meaning of life:", answer );
}

var answer = now();

setTimeout( later, 1000 ); // Meaning of life: 42
```

1. There are two chunks to this program, the stuff that will run _now_  and the stuff that will runt _later_

2. _now_

    ```js
    function now() {
      return 21;
    }

    function later() { .. }

    var answer = now();

    setTimeout( later, 1000 );
    ```

3. _later_

    ```js
    answer = answer * 2;
    console.log( "Meaning of life:", answer );
    ```

4. The _now_ chunk runs right away. `setTimeout(..)` sets up an event to happen _later_, so the contents of the `later()`function will be executed at a later time

Any time you wrap a portion of code into a `function` and specify that it should be executed in response to some event (timer, mouse click, Ajax response, etc) you are creating a _later_ chunk of your code, thus introducing asynchrony to your program

### Event Loop

Until ES6, JS itself has never had any direct notion of asynchrony built into it

The JS engine itself has never done anything more than execute a single chunk of your program at any given moment

Asked by who? by the environment in which is running

- For example, when JS makes an Ajax request to fetch some data from a server

  - You set the "response" code in a function
  - The JS engine tells the hosting environment: "I'm going to suspend execution for now, but when you _finish_ the _request_ and you have some _data_, _call_ this function _back_"
