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

```js
// `eventLoop` is an array that acts as a queue (first-in, first-out)
var eventLoop = [ ];
var event;

// keep going "forever"
while (true) {
 // perform a "tick"
 if (eventLoop.length > 0) {
  // get the next event in the queue
  event = eventLoop.shift();

  // now, execute the next event
  try {
   event();
  }
  catch (err) {
   reportError(err);
  }
 }
}
```

1. Simplified pseudocode
2. There's a continuos loop represented by the `while` loop, each iteration of this loops is called a _tick_

3. For each tick, if there's an event on queue, it's taken off and executed. These events are the function callbacks

4. `setTimeout(..)` doesn't put your callback on the event loop queue

5. `setTimeout(..)` set a timer, when the timer expires, the environment places the callback into the event loop. Some future tick will pick it up

### Parallel Threading

_Async_ and _parallel_ are different, _async_ is about the gap between _now_ and _later_

Parallel is about things being able to occur simultaneously

The most common tools for parallel computing are processes and threads

Processes and threads execute independently and may execute simultaneously: on separate processors, or even separate computers, but multiple threads can share the memory of a single process

An event loop breaks its work into tasks and executes them in serial, disallowing parallel access and changes to shared memory

Parallelism and serialism can coexist in cooperating event loops in separate threads

The interleaving of parallel threads of execution and the interleaving of asynchronous events occur at very different levels

```js
function later() {
  answer = answer * 2;
  console.log( "Meaning of life:", answer );
}
```

1. The entire `later()` would be regarded as a single event loop queue entry

2. When thinking about a thread this code would run on, there's many low-level operations

    - `answer = answer * 2` requires first loading the current value of `answer`, putting 2 somewhere, then perform the multiplication, put the value back in `answer`

In a single-threaded environment, it really doesn't matter that the items in the thread queue are low-level operations, because nothing can interrupt the thread

If you have parallel system, where two different threads are operating in the same program, you could have unpredictable behavior

```js
var a = 20;

function foo() {
  a = a + 1;
}

function bar() {
  a = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

1. In JS single-threaded behavior, if `foo()` runs before `bar()` the result is `a=42`, but if `bar()` runs before `foo()` the result in `a=41`

JS never shares data across threads, but that doesn't mean JS is always deterministic.

### Run-to-completion

Because of JS single-threading, the code inside of `foo()` is atomic, which means that once `foo()` starts running, the entirety of its code will finish before any of the code in `bar()` can run. This is called "run-to-completion"

```js
var a = 1;
var b = 2;

function foo() {
  a++;
  b = b * a;
  a = b + 3;
}

function bar() {
  b--;
  a = 8 + b;
  b = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

1. Because neither `foo()` or `bar()` can be interrupted, there's only 2 possible outcomes.

      Chunk 1 is synchronous (happens _now_), chunks 2 and 3 are asynchronous (happens _later_), which means their execution will be separated by a gap of time

    - Chunk 1

        ```js
        var a = 1;
        var b = 2;
        ```

    - Chunk 2 `foo()`

        ```js
        a++;
        b = b * a;
        a = b + 3;
        ```

    - Chunk 3 `bar()`

        ```js
        b--;
        a = 8 + b;
        b = a * 2;
        ```

    Chunk 2 and 3 may happen in either-first order, so there are 2 possible outcomes

      ```js
      // Outcome 1
      var a = 1;
      var b = 2;

      // foo()
      a++;
      b = b * a;
      a = b + 3;

      // bar()
      b--;
      a = 8 + b;
      b = a * 2;

      a; // 11
      b; // 22
      ```

      ```js
      // Outcome 2
      var a = 1;
      var b = 2;

      // bar()
      b--;
      a = 8 + b;
      b = a * 2;

      // foo()
      a++;
      b = b * a;
      a = b + 3;

      a; // 183
      b; // 180
      ```

Two outcomes from the same code means we still have nondeterminism, but it's at the function (event) ordering level rather than at the statement ordering level

In other words, it's more deterministic than threads would have been

### Concurrency

Concurrency is when two or more "processes" are executing simultaneously over the same period, regardless of whether their individual operations happen in parallel or not

You can think of concurrency as process-level or task-level parallelism, as opposed to operation-level parallelism (separate-processor threads)

### Noninteracting

As two or more "processes" are interleaving their steps/events concurrently within the same program, they don't necessarily need to interact with each other if the tasks are unrelated

If they don't interact, nondeterminism is acceptable

```js
var res = {};

function foo(results) {
  res.foo = results;
}

function bar(results) {
  res.bar = results;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

1. `foo()` and `bar()` are two concurrent processes, and it's nondeterminate which order the will be fired in
    - It doesn't matter what order they fire in, they act independently and don't need to interact
    - The code will always work correctly, regardless of the ordering

### Interaction

More commonly, concurrent "processes" will by necessity interact, indirectly through scope and/or the DOM. You need to coordinate these interactions to prevent "race conditions" as described earlier

```js
var res = [];

function response(data) {
  res.push( data );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

1. The concurrent processes are the two `response()` calls that will be made to handle the ajax responses, they can happen in either first order
2. Sometimes `url1` will end up on `res[0]`, or on `res[1]` depending on which call finishes first
3. This is a race condition bug, and nondeterminism

To address this race condition, you can coordinate ordering interaction

```js
var res = [];

function response(data) {
  if (data.url == "http://some.url.1") {
    res[0] = data;
  }
  else if (data.url == "http://some.url.2") {
    res[1] = data;
  }
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

### Jobs

As of ES6, there's a new concept layered on top of the event loop queue, called the job queue

Currently there's no exposed API to work with it, the Job queue is a queue hanging off the end of every tick in the event loop queue

Certain async-implied actions that may occur during a tick of the event loop will not cause a whole new event to be added to the event loop queue, but will instead add an item (aka Job) to the end of the current ticks Job queue

It's like saying, "oh here's this other thing I need to do _later_, but make sure it happens right away before anything else can happen" _Later, but as soon as possible_

Let's imagine an API for scheduling Jobs (directly, without hacks) and call it `schedule(...)`

```js
console.log( "A" );

setTimeout( function(){
  console.log( "B" );
}, 0 );

// theoretical "Job API"
schedule( function(){
  console.log( "C" );

  schedule( function(){
    console.log( "D" );
  } );
} );
```

1. You might expect to print `A B C D`
2. Will print `A C D B`. Jobs happen at the end of the current event loop tick

## Callbacks

Callbacks are by far the most common way that asynchrony in JS programs is expressed and managed. The callback is the most fundamental async pattern in the language

Callbacks are not without their shortcomings.

### Trying to Save Callbacks

There are several variations of callback design that have attempted to address some of the trust issues

Regarding more graceful error handling, some API designs provide for split callbacks, one for success notification, one for error notification

```js
function success(data) {
  console.log( data );
}

function failure(err) {
  console.error( err );
}

ajax( "http://some.url.1", success, failure );
```

1. In API's of this design, often `failure()` is optional, and if not provided it will be assumed you want the errors swallowed

2. This split-callback design is what the ES6 Promises API uses

Another common callback pattern is called _error-first style_ (also _Node style_, is the convention used across nearly all Nodejs APIs).

The first argument of a single callback is reserved for and error object (if any).

- If success, this argument will be empty/falsy and any subsequent arguments will be the success data
- If an error, the first argument is set truthy and usually nothing else is passed

```js
function response(err,data) {
  // error?
  if (err) {
    console.error( err );
  }
  // otherwise, assume success
  else {
    console.log( data );
  }
}

ajax( "http://some.url.1", response );
```

The story with callbacks is that they can do pretty much anything you want, but you have to be willing to work hard to get it

### 2. Review

1. Callbacks are the fundamental unit of asynchrony in JS.

2. Our brains plan things out in sequential, blocking, single-threaded semantic ways

    - Callbacks express asynchronous flow in a rather nonlinear, nonsequential way
    - This makes reasoning properly about such code much harder
    - Bad to reason about code is bad code that leads to bad bugs

3. We need a way to express asynchrony in a more synchronous, sequential, blocking manner, just like our brains

4. Callbacks suffer from _inversion of control_ they implicitly give control over to another party (often a third-party utility) to invoke the _continuation_ of your program

    - This control transfer leads us to a list of trust issues, such as whether the callbacks is called more times than we expect

5. Inventing ad hoc logic to solve these trust issues is possible, but it's more difficult than it should be and produces clunkier and harder to maintain code

6. We need a generalized solution to all of the trust issues, that can be reused for many callbacks as we want without the extra boilerplate

## Promises

The issue we want to address first is the _inversion of control_

Remember, we wrap up the _continuation_ of our program in a callback function, and hand that callback to another party and hope that it will do the right thing

We want to say "here's what happens _later_ after the current step finishes"

What if we could uninvert that _inversion of control_ ?

What if instead of handing the continuation of our program, we could expect it to return us a capability to know when its task finishes, and then our code decide what to do next?

### What is a Promise

#### Future Value

> Imagine this scenario: I walk up to the counter at a fast-food restaurant, and place an order for a cheeseburger. I hand the cashier $1.47. By placing my order and paying for it, I've made a request for a value back (the cheeseburger). I've started a transaction.
>
> But often, the cheeseburger is not immediately available for me. The cashier hands me something in place of my cheeseburger: a receipt with an order number on it. This order number is an IOU ("I owe you") promise that ensures that eventually, I should receive my cheeseburger.
>
> So I hold onto my receipt and order number. I know it represents my future cheeseburger, so I don't need to worry about it anymore -- aside from being hungry!
>
> While I wait, I can do other things, like send a text message to a friend that says, "Hey, can you come join me for lunch? I'm going to eat a cheeseburger."
>
> I am reasoning about my future cheeseburger already, even though I don't have it in my hands yet. My brain is able to do this because it's treating the order number as a placeholder for the cheeseburger. The placeholder essentially makes the value time independent. It's a future value.
>
> Eventually, I hear, "Order 113!" and I gleefully walk back up to the counter with receipt in hand. I hand my receipt to the cashier, and I take my cheeseburger in return.
>
> In other words, once my future value was ready, I exchanged my value-promise for the value itself.
>
> But there's another possible outcome. They call my order number, but when I go to retrieve my cheeseburger, the cashier regretfully informs me, "I'm sorry, but we appear to be all out of cheeseburgers." Setting aside the customer frustration of this scenario for a moment, we can see an important characteristic of future values: they can either indicate a success or failure.
>
> Every time I order a cheeseburger, I know that I'll either get a cheeseburger eventually, or I'll get the sad news of the cheeseburger shortage, and I'll have to figure out something else to eat for lunch.
>
> Note: In code, things are not quite as simple, because metaphorically the order number may never be called, in which case we're left indefinitely in an unresolved state. We'll come back to dealing with that case later.

##### Values Now and Later

Imagine if there was a way to say add x and y, but if either of them isn't ready yet, just wait until they are. Add them as soon as you can

This could be done with callbacks

```js
function add(getX,getY,cb) {
  var x, y;

  getX( function(xVal){
    x = xVal;
    // both are ready?
    if (y != undefined) {
      cb( x + y );  // send along sum
    }
  } );

  getY( function(yVal){
    y = yVal;
    // both are ready?
    if (x != undefined) {
      cb( x + y );  // send along sum
    }
  } );
}

// `fetchX()` and `fetchY()` are sync or async
// functions
add( fetchX, fetchY, function(sum){
  console.log( sum ); // that was easy, huh?
} );
```

1. We treated `x` and `y` as future values, and we express an operation `add(..)` that from the outside does not care whether both are variables right away or not

    - It normalizes the _now_ and _later_, we can rely on a predictable outcome of the `add(..)`

`add(..)` behaves the same _now_ and _later_

This is just a first tiny step toward realizing the benefits of reasoning about _future values_ without worrying about the time aspect of when it's available or not

##### Promise Value

Let see how we can express `x+y` example via `Promise`

```js
function add(xPromise,yPromise) {
  // `Promise.all([ .. ])` takes an array of promises,
  // and returns a new promise that waits on them
  // all to finish
  return Promise.all( [xPromise, yPromise] )

  // when that promise is resolved, let's take the
  // received `X` and `Y` values and add them together.
  .then( function(values){
  // `values` is an array of the messages from the
  // previously resolved promises
    return values[0] + values[1];
  } );
}

// `fetchX()` and `fetchY()` return promises for
// their respective values, which may be ready
// *now* or *later*.
add( fetchX(), fetchY() )

// we get a promise back for the sum of those
// two numbers.
// now we chain-call `then(..)` to wait for the
// resolution of that returned promise.
.then( function(sum){
  console.log( sum ); // that was easier!
} );
```

1. There are 2 layers of Promises in this snippet

2. `fetchX()` and `fetchY()` are called directly, and the values they return (promises!!!) are passed into `add(..)`

    - The underlying values those promises represent may be ready _now_ or _later_, but each promise normalizes the behavior to be the same regardless

    - We now reason about `X` and `Y` values it a _time-independent_ way. They are _future values_

3. The second layer is the promise that `add(..)` creates via `Promise.all([..])` and returns, which we _WAIT_ on by calling `then(...)`

    - When the `add(..)` operation completes, our `sum` _future value_ is ready and we can print it out

    - We hide inside of `add(..)` the logic for waiting on the `X` and `Y` _future values_

4. Inside `add(..)`, the `Promise.all([..])` call create a promise, that is waiting on `promiseX` and `promiseY` to resolve

    - The chained call to `.then(..)` creates another promise, which the `return values[0] + values[1]` immediately resolves with the result of the addition

    - The `then(..)` call we chain off the end of the `add(..)` call is actually operating on that second promise returned, rather than the first one create by `Promise.all([..])`

    - Though we are not chaining off the end of that second `then(..)`, it too has created another promise, had we chosen to observe/use it.

It's possible that the resolution of a Promise is rejection instead of fulfillment, a rejection value can either be set directly by the program logic, or it can result implicitly from a runtime exception

With Promises, the `then(..)` call can actually take two functions, the first for fulfillment and the second for rejection

```js
add( fetchX(), fetchY() )
.then(
  // fulfillment handler
  function(sum) {
    console.log( sum );
  },

  // rejection handler
  function(err) {
    console.error( err ); // bummer!
  }
);
```

1. If something went wrong getting `X` or `Y`, or something failed during the addition, the promise that `add(..)` returns is rejected, and the second callback error handler passed to `then(..)` will receive the rejection value from the promise

Because Promises encapsulate the time-dependent state -- waiting on the fulfillment or rejection of the underlying value -- from the outside, the Promise itself is _time-independent_, and thus Promises can be composed in predictable ways regardless of the timing or outcome underneath

Once a Promise is resolved, it stays that way forever, it becomes an immutable value, and can then be observed as many times as necessary

That's one of the most powerful and important concepts to understand about Promises.

### Completion Event

There's another way to think of the resolution of a Promise: as a flow-control mechanism -- a temporal this-then-that -- for two or more steps in an asynchronous task

Let's imagine calling a function `foo(..)` to perform some task. We don't know about any of its details, nor do we care. It may complete right away, or take a while

We just simply need to know when `foo(..)` finishes so that we can move on to our next task. We did like a way to be notified of `foo(..)` completion so that we can continue

With _callbacks_ the "notification" would be our callback invoked by the task `foo(..)`.

With _Promises_, we turn the relationship around, and expect that we can listen for an event from `foo(..)`, and when notified, proceed accordingly

Consider some pseudocode

```js
foo(x) {
  // start doing something that could take a while
}

foo( 42 )

on (foo "completion") {
  // now we can do the next step!
}

on (foo "error") {
  // oops, something went wrong in `foo(..)`
}
```

1. We call `foo(..)` and then we set up two event listeners

    - In essence, `foo(..)` doesn't even appear to be aware that the calling code has subscribed to these events

    - This makes a very nice _separation of concerns_

The more natural way we could express in JS

```js
function foo(x) {
  // start doing something that could take a while

  // make a `listener` event notification
  // capability to return

  return listener;
}

var evt = foo( 42 );

evt.on( "completion", function(){
  // now we can do the next step!
} );

evt.on( "failure", function(err){
  // oops, something went wrong in `foo(..)`
} );
```

1. `foo(..)` expressly creates an event subscription capability to return back, and the calling code receives and registers the 2 event handlers against it

2. The inversion from normal callback-oriented code should be obvious and it's intentional.

    - Instead of passing the callbacks to `foo(..)`, it returns an event capability we call `evt`, which receives the callbacks

Remember, callbacks themselves represent an _inversion of control_, inverting the callback pattern is actually an _inversion of inversion_.Restoring control back to the calling code where we wanted it to be in the first place

One important benefit is that multiple separate parts of the code can be given the event listening capability, and they can all independently be notified of when `foo(..)` completes

```js
var evt = foo( 42 );

// let `bar(..)` listen to `foo(..)`'s completion
bar( evt );

// also, let `baz(..)` listen to `foo(..)`'s completion
baz( evt );
```

_Uninversion of control_ enables a nicer separation of concerns, `bar(..)` and `baz(..)` don't need to be involved in how `foo(..)` is called

Similarly, `foo(..)` doesn't need to know or care that `bar(..)` and `baz(..)` exist or are waiting to be notified when `foo(..)` completes

Essentially, this `evt` object is a neutral third-party negotiation between the separate concerns

#### Promise "Events"

The `evt` listening capability is an analogy for a Promise

In a Promise-based approach, the `foo(..)` would create and return a `Promise` instance, and that promise would then be passed to `bar(..)` and `baz(..)`

```js
function foo(x) {
  // start doing something that could take a while

  // construct and return a promise
  return new Promise( function(resolve,reject){
    // eventually, call `resolve(..)` or `reject(..)`,
    // which are the resolution callbacks for
    // the promise.
  } );
}

var p = foo( 42 );

bar( p );

baz( p );
```

1. The pattern shown with `new Promise( function(..){...} )` is generally called the _revealing constructor_

2. The function passed is executed immediately (not async deferred, as callbacks to `then(..)`), and it's provided two parameters `resolve` and `reject`

3. These are the resolution functions for the promise

The internals of `bar(..)` and `baz(..)` could be something like

```js
function bar(fooPromise) {
  // listen for `foo(..)` to complete
  fooPromise.then(
    function(){
      // `foo(..)` has now finished, so
      // do `bar(..)`'s task
    },
    function(){
      // oops, something went wrong in `foo(..)`
    }
  );
}

// ditto for `baz(..)`
```

Promise resolution can just be a flow-control signal, as used in the previous snippet

```js
function bar() {
  // `foo(..)` has definitely finished, so
  // do `bar(..)`'s task
}

function oopsBar() {
  // oops, something went wrong in `foo(..)`,
  // so `bar(..)` didn't run
}

// ditto for `baz()` and `oopsBaz()`

var p = foo( 42 );

p.then( bar, oopsBar );

p.then( baz, oopsBaz );
```

1. Instead of passing the `p` promise to `bar(..)` we use the promise control when `bar(..)` will get executed, if ever. The primary difference is in the error handling

In the first approach, `bar(..)` is called regardless of whether `foo(..)` succeeds or fails, and it handles its own fallback logic if `foo(..)` failed

In the second snippet, `bar(..)` only gets called if `foo(..)` succeeds, otherwise `oopsBar(..)` gets called

In either case, the promise `p` that comes from `foo(..)` is used to control what happens next

### Thenable Duck Typing

An important detail is how to know for sure if some value is genuine Promise or not, Or more directly, is it a value that will behave like a Promise?

The way to recognize a Promise, would be to define something called a "thenable" as any object or function which has a `then(..)` method on it

It is assumed that any such value is a Promise-conforming thenable

The general term for "type checks" that make assumptions about a values "type" based on its shape (what properties are present) is called _duck typing_

If it looks like a duck, and quacks like a duck, it must be a duck

The duck typing check for a thenable would roughly be

```js
if (
  p !== null &&
  (
    typeof p === "object" ||
    typeof p === "function"
  ) &&
  typeof p.then === "function"
) {
  // assume it's a thenable!
}
else {
  // not a thenable
}
```

1. If you try to fulfill a Promise with any object/function value that happens to have a `then(..)` function on it, but you weren't intending it to be treated as a Promise/thenable you're out of luck, it will automatically be recognized as thenable and treated with special rules

### Promise Trust

Whereas the _future values_ and _completion events_ analogies play out explicitly in the code patterns we've explored, it won't be entirely obvious why or how Promises are designed to solve all of the _inversion control_ trust issues

Let's start by reviewing the trust issues with  callbacks-only coding, when you pass a callback to a utility `foo(..)` it might:

1. Call the callback too early
2. Call the callback too late (or never)
3. Call the callback too few or too many times
4. Fail to pass along any necessary environment/parameters
5. Swallow any errors/exceptions that may happen

The characteristics of Promises are intentionally designed to provide useful, repeatable answers to all these concerns

#### Calling Too Early

This is a concern of whether code can introduce Zalgo-like effects, where sometimes a task finishes synchronously and sometimes asynchronously, which can lead to race conditions

Promises by definition cannot be susceptible to this concern, because even an immediately fulfilled Promise cannot be _observed_ synchronously

When you call `then(..)` on a Promise, even if that Promise was already resolved, the callback you provide to `then(..)` will _always_ be called asynchronously

#### Calling Too Late

A Promise `then(..)` registered observation callbacks are automatically scheduled when either `resolve(..)` or `reject(..)` are called by the Promise creation capability

Those scheduled callbacks will predictably be fired at the next asynchronous moment

It's not possible for synchronous observation, so it's not possible for a synchronous chain of tasks to run in such way to in effect delay another callback from happening as expected

That is, when a Promise is resolved, all `then(..)` registered callbacks on it will be called, in order, immediately at the next asynchronous opportunity (Jobs ES6), and nothing inside of one of those callbacks can affect/delay the calling of the other callbacks

```js
p.then( function(){

  p.then( function(){
    console.log( "C" );
  } );

  console.log( "A" );
} );

p.then( function(){
  console.log( "B" );
} );
// A B C
```

1. `C` cannot interrupt and precede `B`, by virtue of how Promises are defined to operate

##### Promise Scheduling Quirks

There're lots of nuances of scheduling where the relative ordering between callbacks chained off two separate Promises is no reliably predictable

If 2 promises, `p1` and `p2` are both already resolved, it should be true that `p1.then(..); p2.then(..)` would end up calling the callbacks `p1` before `p2`

There are subtle cases where that might not be true

```js
var p3 = new Promise( function(resolve,reject){
  resolve( "B" );
} );

var p1 = new Promise( function(resolve,reject){
  resolve( p3 );
} );

var p2 = new Promise( function(resolve,reject){
  resolve( "A" );
} );

p1.then( function(v){
  console.log( v );
} );

p2.then( function(v){
  console.log( v );
} );

// A B  <-- not  B A  as you might expect
```

1. `p1` is resolved not with an immediate value, but with another promise `p3`

2. The specified behavior is to _unwrap_ `p3` into `p1` but asynchronously, so `p1` callbacks are _behind_ `p2` callbacks in the asynchronous job queue

#### Never Calling the Callback

Nothing (not even a JS error) can prevent a Promise from notifying you of its resolution (if it's resolved).

If you register both fulfillment and rejection callbacks for a Promise, and the Promise gets resolved, one of 2 callbacks will always be called

What if the Promise itself never gets resolved either way? Promises provide an answer for, using a higher level abstraction called a race

```js
// a utility for timing out a Promise
function timeoutPromise(delay) {
  return new Promise( function(resolve,reject){
    setTimeout( function(){
      reject( "Timeout!" );
    }, delay );
  } );
}

// setup a timeout for `foo()`
Promise.race( [
  foo(),          // attempt `foo()`
  timeoutPromise( 3000 )  // give it 3 seconds
] )
.then(
  function(){
    // `foo(..)` fulfilled in time!
  },
  function(err){
    // either `foo()` rejected, or it just
    // didn't finish in time, so inspect
    // `err` to know which
  }
);
```

1. We can ensure a signal as to the outcome of `foo()`, to prevent it from hanging our program indefinitely

### Calling Too Few or Too Many Times

By definition, _one_ is the appropriate number of times for the callback to be called. The _too few_ case would be zero calls, the same as never case

The _too many_ case is easy to explain, Promises are defined so that they can only be resolved once

If for some reason the Promise creation code tries to call `resolve(..)` or `reject(..)` multiple times, the Promise will accept only the first resolution and will silently ignore any subsequent attempts

Because a Promise can only be resolved once, any `then(..)` registered callbacks will only ever be called once (each)

### Failing to Pass Along Any Parameters/Environment

Promises can have, at most, one resolution value, fulfillment or rejection

If you don't explicitly resolve with a value either way, the value is `undefined`.

Whatever the value, it will always be passed to all registered callbacks, either now or in the future

If you call `resolve(..)` or `reject(..)` with multiple parameters, all subsequent parameters beyond the first will be silently ignored
    - This constitutes an invalid usage of the Promise mechanism

If you want to pass along multiple values, you must wrap them in another single value that you pass, like an `array` or `object`

### Swallowing Any Errors/Exceptions

If you reject a Promise with a reason (aka error message), that value is passed to the rejection callback

If at any point in the creation of a Promise, or in the observation of its resolution, a JS exception error occurs (`TypeError`, `ReferenceError`), that exception will be caught, and it will force the Promise in question to become rejected

```js
var p = new Promise( function(resolve,reject){
  foo.bar();  // `foo` is not defined, so error!
  resolve( 42 );  // never gets here :(
} );

p.then(
  function fulfilled(){
    // never gets here :(
  },
  function rejected(err){
    // `err` will be a `TypeError` exception object
    // from the `foo.bar()` line.
  }
);
```

1. The JS exception from `foo.bar()` becomes a Promise rejection that you can catch and respond to

2. Promises turn even JS exceptions into asynchronous behavior, thereby reducing the race condition

What happens if a Promise is fulfilled, but there's a JS exception error during the observation (in a `then(..)` registered callback)

Even those aren't lost

```js
var p = new Promise( function(resolve,reject){
  resolve( 42 );
} );

p.then(
  function fulfilled(msg){
    foo.bar();
    console.log( msg );  // never gets here :(
  },
  function rejected(err){
    // never gets here either :(
  }
);
```

1. It looks like the exception from `foo.bar()` got swallowed. It didn't, we have failed to listen for it

2. The `p.then(..)` call itself returns another promise, and it's _that_ promise that will be rejected with the `TypeError` exception

3. Why couldn't it just call the error handler we have defined here?

    - It would violate the fundamental principle that Promises are _immutable_ once resolved

    - `p` was already fulfilled to the value `42`, it can't later be changed to a rejection just because there's an error in observing `p`'s resolution

### Trustable Promise ?

Promises don't get rid of callbacks at all, they just change where the callback is passed to.

Instead of passing a callback to `foo(..)`, we get something back from`foo(..)` and we pass the callback to that _something_

How can we be sure the _something_ we get back is in fact a trustable Promise?

Promises have a solution to this issue, `Promise.resolve(..)`

If you pass an immediate, non-Promise, non-thenable value to `Promise.resolve(..)` you get a Promise that's fulfilled with that value

```js
var p1 = new Promise( function(resolve,reject){
  resolve( 42 );
} );

var p2 = Promise.resolve( 42 );
```

1. `p1` and `p2` will behave basically identically

If you pass a genuine Promise to `Promise.resolve(..)` you just get the same promise back

```js
var p1 = Promise.resolve( 42 );

var p2 = Promise.resolve( p1 );

p1 === p2; // true
```

Even more importantly, if you pass a non-PRomise thenable value to `Promise.resolve(..)` it will attempt yo unwrap that value, until a concrete final non-Promise-like value is extracted

```js
var p = {
  then: function(cb) {
    cb( 42 );
  }
};

// this works OK, but only by good fortune
p
.then(
  function fulfilled(val){
    console.log( val ); // 42
  },
  function rejected(err){
    // never gets here
  }
);
```

1. `p` is a thenable, but it's not a genuine Promise

If you had something like this, it would not work

```js
var p = {
  then: function(cb,error_cb) {
    cb( 42 );
    error_cb( "evil laugh" );
  }
};

p
.then(
  function fulfilled(val){
    console.log( val ); // 42
  },
  function rejected(err){
    // oops, shouldn't have run
    console.log( err ); // evil laugh
  }
);
```

1. `p` is thenable but it'snot a well behaved of a promise

We can pass either versions of `p` to `Promise.resolve(..)` and we will get the normalized, safe result

```js
Promise.resolve( p )
.then(
  function fulfilled(val){
    console.log( val ); // 42
  },
  function rejected(err){
    // never gets here
  }
);
```

1. `Promise.resolve(..)` will accept any thenable, and will unwrap it to its non-thenable value

2. You get a real, genuine Promise, one that you can trust

If we're calling a `foo(..)` utility and we are not sure we can trust its return value to be a Promise, but we know it's a thenable

```js
// don't just do this:
foo( 42 )
.then( function(v){
  console.log( v );
} );

// instead, do this:
Promise.resolve( foo( 42 ) )
.then( function(v){
  console.log( v );
} );
```

1. Another beneficial side effect of wrapping `Promise.resolve(..)` around any functions return value is that is an easy way to normalize that function call into a well-behaving async task

### Chain Flow

Promises are not just a mechanism for a single-step _this-then-that_ sort of operation

That's the building block, but we can string multiple Promises together to represent a sequence of async steps

The key to making this work is built on two behaviors intrinsic to Promises:

1. Every time you call `then(..)` on a Promise, it creates and return a new Promise, which we can _chain_ with

2. Whatever value you return from the `then(..)` calls fulfillment callback, is automatically set as the fulfillment of the _chained_ Promise (from the first point)

```js
var p = Promise.resolve( 21 );

var p2 = p.then( function(v){
  console.log( v );  // 21

  // fulfill `p2` with value `42`
  return v * 2;
} );

// chain off `p2`
p2.then( function(v){
  console.log( v );  // 42
} );
```

1. By returning `v * 2` we fulfill the `p2` promise that the first `then(..)` call created and returned

2. When `p2` `then(..)` call runs, it's receiving the fulfillment from the `return v * 2` statement

    - `p2.then(..)` creates yet another promise that we could store in `p3`

It;s annoying to have to create an intermediate variable `p2`, we can easily chain these together

```js
var p = Promise.resolve( 21 );

p
.then( function(v){
  console.log( v );  // 21

  // fulfill the chained promise with value `42`
  return v * 2;
} )
// here's the chained promise
.then( function(v){
  console.log( v );  // 42
} );
```

1. The first `then(..)` is the first step in an async sequence, and the second `then(..)` is the second step.

2. What if we want step 2 to wait for step 1 to do something asynchronous?. We're using an immediate `return` statement, which immediately fulfills the chained promise

The key to making a Promise sequence truly async capable at every step is to recall how `Promise.resolve(..)` operates when what you pass to it is a Promise or thenable instead of a final value

`Promise.resolve(..)` directly returns a received genuine Promise, or it unwraps the value of a received thenable

The same sort of unwrapping happens if you `return` a thenable or Promise from the fulfillment or rejection handler

```js
var p = Promise.resolve( 21 );

p.then( function(v){
  console.log( v );  // 21

  // create a promise and return it
  return new Promise( function(resolve,reject){
    // fulfill with value `42`
    resolve( v * 2 );
  } );
} )
.then( function(v){
  console.log( v );  // 42
} );
```

1. Even when we wrapped `42` in a promise that we returned, it got unwrapped and ended up as the resolution of the chained promise

2. If we introduce asynchrony to that wrapping promise, everything still nicely works the same

```js
var p = Promise.resolve( 21 );

p.then( function(v){
  console.log( v );  // 21

  // create a promise to return
  return new Promise( function(resolve,reject){
    // introduce asynchrony!
    setTimeout( function(){
      // fulfill with value `42`
      resolve( v * 2 );
    }, 100 );
  } );
} )
.then( function(v){
  // runs after the 100ms delay in the previous step
  console.log( v );  // 42
} );
```

1. We can construct a sequence of however many async steps we want, and each step can delay the next step as necessary

Let's consider making Ajax requests:

```js
// assume an `ajax( {url}, {callback} )` utility

// Promise-aware ajax
function request(url) {
  return new Promise( function(resolve,reject){
    // the `ajax(..)` callback should be our
    // promise's `resolve(..)` function
    ajax( url, resolve );
  } );
}
```

We first define a `request(..)` utility that constructs a promise to represent the completion of the `ajax(..)` call

```js
request( "http://some.url.1/" )
.then( function(response1){
  return request( "http://some.url.2/?v=" + response1 );
} )
.then( function(response2){
  console.log( response2 );
} );
```

1. Using the Promise that `request(..)` return we create the first step in our chain by calling it with the first URL

    - We chain off that returned promise with the first `then(..)`

2. Once `response1` comes back, we use that value to construct a second URL and make a second `request(..)` call

    - That second `request(..)` promise is returned so that the third step in our async flow control waits for that Ajax call to complete

    - Finally, we print `response2` once it returns

The Promise chain in not only a flow control that express a multistep async sequence, but also acts as a message channel to propagate messages from step to step

- If something went wrong in one of the steps of the Promise chain, an error/exception is on a per-Promise basis

  - It's possible to catch such an error at any point in the chain, and that catching acts sort of "reset" the chain back to normal operation an that point

```js
// step 1:
request( "http://some.url.1/" )

// step 2:
.then( function(response1){
  foo.bar(); // undefined, error!

  // never gets here
  return request( "http://some.url.2/?v=" + response1 );
} )

// step 3:
.then(
  function fulfilled(response2){
    // never gets here
  },
  // rejection handler to catch the error
  function rejected(err){
    console.log( err );  // `TypeError` from `foo.bar()` error
    return 42;
  }
)

// step 4:
.then( function(msg){
  console.log( msg );    // 42
} );
```

1. When the error occurs in step 2, the rejection handler in step 3 catches it

    - The return value, if any, from that rejection handler fulfills the promise for the next step (4) such that the chain is now back in a fulfillment state

2. When returning a promise from a fulfillment handler, it's unwrapped and can delay the next step

    - If the `return 42` in step 3 instead returned a promise, that promise could delay step 4.

    - A thrown exception inside either the fulfillment or rejection handler of a `then(..)` call causes the next chained promise to be immediately rejected with that exception

If you call `then(..)` on a promise, and you only pass a fulfillment handler to it, an assumed rejection handler is substituted

```js
var p = new Promise( function(resolve,reject){
  reject( "Oops" );
} );

var p2 = p.then(
  function fulfilled(){
    // never gets here
  }
  // assumed rejection handler, if omitted or
  // any other non-function value passed
  // function(err) {
  //     throw err;
  // }
);
```

1. The assumed rejection handler simply rethrows the error, which ends up forcing `p2` to reject with the same reason

If a proper valid function is not passed as the fulfillment handler parameter to `then(..)`, there's also a default handler

```js
var p = Promise.resolve( 42 );

p.then(
  // assumed fulfillment handler, if omitted or
  // any other non-function value passed
  // function(v) {
  //     return v;
  // }
  null,
  function rejected(err){
    // never gets here
  }
);
```

1. The default fulfillment simple passes whatever value it receives along to the next step

2. The `then(null, function(err){..})` patter -- only handling rejections, if any, but letting fulfillments pass through -- has a shortcut in the API `catch(function (err){..})`

#### Terminology: Resolve, Fulfill and Reject

Let's consider the `Promise(..)` constructor

```js
var p = new Promise( function(X,Y){
  // X() for fulfillment
  // Y() for rejection
} );
```

`resolve()` and `reject()` is the standard

Let's turn our attention to the callbacks provided to `then(..)`. A good option is `fulfilled(..)` and `rejected(..)`

```js
function fulfilled(msg) {
  console.log( msg );
}

function rejected(err) {
  console.error( err );
}

p.then(
  fulfilled,
  rejected
);
```

### Error Handling

The most natural form of error handling for most developers is the synchronous `try..catch` construct

That is synchronously only, so it fails to help in async code

```js
function foo() {
  setTimeout( function(){
    baz.bar();
  }, 100 );
}

try {
  foo();
  // later throws global error from `baz.bar()`
}
catch (err) {
  // never gets here
}
```

Consider

```js
var p = Promise.resolve( 42 );

p.then(
  function fulfilled(msg){
    // numbers don't have string functions,
    // so will throw an error
    console.log( msg.toLowerCase() );
  },
  function rejected(err){
    // never gets here
  }
);
```

1. `msg.toLowerCase()` throw an error but

    - The error is for the `p` promise, which has already been fulfilled with value `42`

    - The `p` promise is immutable, so the only promise that can be notified of the error is the one returned from `p.then(..)`, which we don't capture

To avoid losing an error to the silence of a forgotten Promise, some say that a best practice is to place a `catch(..)` at the end of the chain

```js
var p = Promise.resolve( 42 );

p.then(
  function fulfilled(msg){
    // numbers don't have string functions,
    // so will throw an error
    console.log( msg.toLowerCase() );
  }
)
.catch( handleErrors );
```

1. Because we didn't pass a rejection handler to `then(..)`, the default handler was substituted, which propagates the error

2. Both errors that come into `p` and error that come after `p` in its resolution will filter down to the final `handleErrors(..)`

3. If `handleErrors(..)` has an error, there's still another unattended promise

    - The promise that `catch(..)` returns!!! which we don't capture and don't register a rejection handler for

### Promise Patterns

#### Promise.all([..])

Allows to do multiple Promises at a time

```js
// `request(..)` is a Promise-aware Ajax utility,
// like we defined earlier in the chapter

var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

Promise.all( [p1,p2] )
.then( function(msgs){
  // both `p1` and `p2` fulfill and pass in
  // their messages here
  return request(
    "http://some.url.3/?v=" + msgs.join(",")
  );
} )
.then( function(msg){
  console.log( msg );
} );
```

1. `Promise.all([..])` expects a single argument, an `array` consisting generally of Promise instances

2. The promise returned from `Promise.all([..])` will receive a fulfillment message that is an `array` of all the fulfillment messages from the passed in promises

    - The order of the msg is the same as the order in the initial `array` of Promises

3. The main promise returned from `Promise.all([..])` will only be fulfilled if and when all promises are fulfilled

    - If any promise is rejected, the main promise is immediately rejected

    - Always attach a rejection/error handler to every promise

#### Promise.race([..])

Sometimes you only want to respond to the first Promise to cross the finish line, letting the other Promises fall away

`Promise.race([..])` also expects a single `array` containing one or more Promises, thenables or immediate values

Will fulfill if and when any Promise resolution is a fulfillment and it will reject if and when any Promise resolution is a rejection

### Promise Limitations

#### Sequence Error Handling

Because a Promise chain is nothing more than is constituent Promises wired together, there's no entity to refer to the entire chain as a single _thing_, there's no external way to observe any errors that may occur

If you construct a Promise chain that has no error handling in it, any error anywhere in the chain will propagate indefinitely down the chain, until observed by registering a rejection handler at some step.

In that specific case, having a reference to the _last_ promise in the chain is enough (`p` in the snippet)`, because you can register a rejection handler there and it will be notified of any propagated errors

```js
// `foo(..)`, `STEP2(..)` and `STEP3(..)` are
// all promise-aware utilities

var p = foo( 42 ).then( STEP2 ).then( STEP3 );
```

1. `p` here is pointing to the las promise, the one that comes from the `then(STEP3)` call

2. No step is doing its own error handling. You could register a rejection error handler on `p`

    `p.catch(handlerErrors)`

    - If any step of the chain in fact does its own error handling, your `handleErrors` won't be notified

    - The complete lack of ability to be notified of already _handled_ rejection errors is a limitation that restricts capabilities in some use cases

It's basically the same limitation that exist with a `try..catch` that can catch an exception and simply swallow it

#### Single Value

Promises only have a single fulfillment value or a single rejection reason

The typical advice is to construct a values wrapper to contain the multiple messages

##### Splitting Values

Imagine you have a utility `foo(..)` that produces two values asynchronously

```js
function getY(x) {
  return new Promise( function(resolve,reject){
    setTimeout( function(){
      resolve( (3 * x) - 1 );
    }, 100 );
  } );
}

function foo(bar,baz) {
  var x = bar * baz;

  return getY( x )
  .then( function(y){
    // wrap both values into container
    return [x,y];
  } );
}

foo( 10, 20 ).then( function(msgs){
  var x = msgs[0];
  var y = msgs[1];

  console.log( x, y );  // 200 599
});
```

First lets rearrange what `foo(..)` returns so that we don't have to wrap `x` and `y` into a single `array` value to transport through one Promise.

```js
function foo(bar,baz) {
  var x = bar * baz;

  // return both promises
  return [
    Promise.resolve( x ),
    getY( x )
  ];
}

Promise.all(
  foo( 10, 20 )
)
.then( function(msgs){
  var x = msgs[0];
  var y = msgs[1];

  console.log( x, y );
} );
```

1. This approach more closely embraces the Promise design theory

    - It's now easier in the future to refactor to split the calculation of `x` and `y` into separate functions

    - It's cleaner and more flexible to let the calling code decide how to orchestrate the two promises (using `Promise.all([..])`, but not the only option) rather than to abstract such details away inside of `foo(..)`

##### Unwrap/Spread Arguments

The `var x = ...` and `var y = ...` assignments are still awkward overhead. We can employ some functional trickery in a helper utility

```js
function spread(fn) {
  return Function.apply.bind( fn, null );
}

Promise.all(
  foo( 10, 20 )
)
.then(
  spread( function(x,y){
    console.log( x, y );  // 200 599
  } )
)
```

Can even use inline functional magic to avoid the extra helper

```js
Promise.all(
  foo( 10, 20 )
)
.then( Function.apply.bind(
  function(x,y){
    console.log( x, y );  // 200 599
  },
  null
) );
```

With ES6 we can use destructuring

```js
Promise.all(
  foo( 10, 20 )
)
.then( function(msgs){
  var [x,y] = msgs;

  console.log( x, y );  // 200 599
} );
```

Also we can destructure parameters

```js
Promise.all(
  foo( 10, 20 )
)
.then( function([x,y]){
  console.log( x, y );  // 200 599
} );
```

#### Single Resolution

Promises can only be resolved once (fulfillment or rejection).

There's a lot of async cases that fit into a different model, one that's more akin to events and/or streams of data

Imagine a scenario where you might want to fire off a sequence of async steps in response to a stimulus (like an event) that can in fact happen multiple times, like a button click

This probably won't work the way you want

```js
// `click(..)` binds the `"click"` event to a DOM element
// `request(..)` is the previously defined Promise-aware Ajax

var p = new Promise( function(resolve,reject){
  click( "#mybtn", resolve );
} );

p.then( function(evt){
  var btnID = evt.currentTarget.id;
  return request( "http://some.url.1/?id=" + btnID );
} )
.then( function(text){
  console.log( text );
} );
```

1. The behavior only works if your application calls for the button to be clicked just one.

    - If the button is clicked a second time, the `p` promise has already been resolved, so the second `resolve()` call would be ignored

You probably need to invert the paradigm, creating a whole new Promise chain for each event firing

```js
click( "#mybtn", function(evt){
  var btnID = evt.currentTarget.id;

  request( "http://some.url.1/?id=" + btnID )
  .then( function(text){
    console.log( text );
  } );
} );
```

1. A whole new Promise sequence will be fired off for each `click` event on the button

2. Beyond of having to define the entire Promise chain inside the event handler, this design violates the idea of separation of concerns/capabilities

    - You might want to define your event handler in a different place in your code from where you define the _response_ to the event (the Promise chain)

##### Inertia

Promises offer a different paradigm, consider a callback-based scenario like the following

```js
function foo(x,y,cb) {
  ajax(
    "http://some.url.1/?x=" + x + "&y=" + y,
    cb
  );
}

foo( 11, 31, function(err,text) {
  if (err) {
    console.error( err );
  }
  else {
    console.log( text );
  }
} );
```

1. Promises don't just magically adjust to the code, you need to make them fit with your expertise

2. We need an Ajax utility that is Promise-aware instead of callback-based

    - We can call it `request(..)`

    - The overhead of having to manually define Promise-aware wrappers for every callback-based utility makes it less likely you'll choose to refactor to Promise-aware coding at all

Imagine a helper like this

```js
// polyfill-safe guard check
if (!Promise.wrap) {
  Promise.wrap = function(fn) {
    return function() {
      var args = [].slice.call( arguments );

      return new Promise( function(resolve,reject){
        fn.apply(
          null,
          args.concat( function(err,v){
            if (err) {
              reject( err );
            }
            else {function foo(x,y,cb) {
  ajax(
    "http://some.url.1/?x=" + x + "&y=" + y,
    cb
  );
}

foo( 11, 31, function(err,text) {
  if (err) {
    console.error( err );
  }
  else {
    console.log( text );
  }
} );
              resolve( v );
            }
          } )
        );
      } );
    };
  };
}
```

1. It takes a function that expects an error-first style callback as its last parameter, and returns a new one that automatically creates a Promise to return and substitutes the callback for you, wired up to the Promise fulfillment/rejection

This is how it would be use

```js
var request = Promise.wrap(ajax)
request("http://some.url.1/")
.then(...)
...
```

1. `Promise.wrap(..)` does not produce a Promise.

    - Produces a function that will produce Promises.

    - A Promise-producing function could be seen as a _Promise factory_ or _Promisory_

2. Wrapping a callback-expecting function to be a Promise-aware function is sometimes referred to as _lifting_ or _promisifying_

3. `Promise.wrap(ajax)` produces an `ajax(..)` promisory we call `request(..)`, and that promisory produces Promises for Ajax responses

Back to the example, we need a promisory for both `ajax(..)` and `foo(..)`

```js
// make a promisory for `ajax(..)`
var request = Promise.wrap( ajax );

// refactor `foo(..)`, but keep it externally
// callback-based for compatibility with other
// parts of the code for now -- only use
// `request(..)`'s promise internally.
function foo(x,y,cb) {
  request(
    "http://some.url.1/?x=" + x + "&y=" + y
  )
  .then(
    function fulfilled(text){
      cb( null, text );
    },
    cb
  );
}

// now, for this code's purposes, make a
// promisory for `foo(..)`
var betterFoo = Promise.wrap( foo );

// and use the promisory
betterFoo( 11, 31 )
.then(
  function fulfilled(text){
    console.log( text );
  },
  function rejected(err){
    console.error( err );
  }
);
```

1. We are refactoring `foo(..)` to use our new `request(..)` promisory

2. We could just make `foo(..)` a promisory itself, instead of remaining callback-based and needing to make and use the subsequent `betterFoo(..)` promisory

```js
// `foo(..)` is now also a promisory because it
// delegates to the `request(..)` promisory
function foo(x,y) {
  return request(
    "http://some.url.1/?x=" + x + "&y=" + y
  );
}

foo( 11, 31 )
.then( .. )
..
```

#### Promise Uncancelable

Once you create a Promises and register a fulfillment and/or rejection handler for it, there's nothing external you can do to stop that progression

Consider

```js
var p = foo( 42 );

Promise.race( [
  p,
  timeoutPromise( 3000 )
] )
.then(
  doSomething,
  handleError
);

p.then( function(){
  // still happens even in the timeout case :(
} );
```

1. The timeout was external to the promise `p`, so `p` keep going, which we probably don't want

One option is to invasive define your resolution callback

```js
var OK = true;

var p = foo( 42 );

Promise.race( [
  p,
  timeoutPromise( 3000 )
  .catch( function(err){
    OK = false;
    throw err;
  } )
] )
.then(
  doSomething,
  handleError
);

p.then( function(){
  if (OK) {
    // only happens if no timeout! :)
  }
} );
```

1. This is ugly, works but's it is not ideal

2. _Cancellation_ is a functionality that belongs at a higher level of abstraction on top of Promises

A single Promises is not really a flow-control mechanism

A chain of Promises taken collectively together is a flow control expression, and thus it's appropriate for cancellation to be defined at that level of abstraction

No individual Promise should be cancelable, but it's sensible for a sequence to be cancellable, because you don't pass around a sequence as a single immutable value like you do with a Promise

## Generators

Chapter 2 we identifies 2 key drawbacks to expressing async flow control with callbacks

1. Callback-based async doesn't fit how our brain plans out steps of a task
2. Callbacks aren't trustable or composable because of _inversion of control_

Chapter 3, detailed how Promises uninvert the _inversion of control_ of callbacks, restoring trustability/composability

Now let's express async flow control in a sequential, synchronous-looking fashion.  This is possible with ES6 generators

### Breaking Run-to-Completion

Once a function starts executing, it runs until it completes, and no other code can interrupt and run in between

ES6 introduces a new type of function that does not behave with the run-to-completion behavior. This new type of function is called a _generator_

```js
var x = 1;

function foo() {
  x++;
  bar();        // <-- what about this line?
  console.log( "x:", x );
}

function bar() {
  x++;
}

foo();          // x: 3
```

1. `bar()` runs in between `x++` and `console.log(x)`. But what if `bar()` wasn't there ?

    - Obviously the result would be `2` instead of `3`

2. What if `bar()` wasn't present, but it could still somehow run between the `x++` and `console.log(x)`

In _preemptive_ multithread languages, it would be possible for `bar()` to interrupt and run at exactly the right moment between those statements

```js
var x = 1;

function *foo() {
  x++;
  yield; // pause!
  console.log( "x:", x );
}

function bar() {
  x++;
}
```

Now, how we can run the code in that previous snippet such that `bar()` executes at the point of the `yield` inside of `*foo()`

```js
// construct an iterator `it` to control the generator
var it = foo();

// start `foo()` here!
it.next();
x;            // 2
bar();
x;            // 3
it.next();        // x: 3
```

1. `it=foo()` does not execute the `*foo()` generator yet, but it constructs an _iterator_ that will control its execution

2. The first `it.next()` starts the `*foo()` generator, and runs the `x++` on the first line of `*foo()`

3. `*foo()` pauses at the `yield` statement at which point that first `it.nexT()` call finishes

    - At the moment `*foo()` is stull running and active, but it is paused

4. We call `bar()` which increments `x`

5. The final `it.next()` call resumes the `*foo()` generator from where it was paused, and runs the `console.log(..)`

Clearly `*foo()` started but did not run to completion, it paused at the `yield`. we resumed `*foo()` later, and let it finish, but that wasn't even required

A generator is a special kind of function that can start and stop one or more times, and doesn't necessarily ever have to finish

### Input and Output

A generator function is a special function with the new processing model we just alluded to.

It's still a function, still accepts arguments and still return a value

```js
function *foo(x,y) {
  return x * y;
}

var it = foo( 6, 7 );

var res = it.next();

res.value;    // 42
```

1. We pass the arguments `6` and `7`, `*foo()` returns the value `42` back to the calling code

2. Now we see a difference with how the generator is invoked compared to a normal function. `foo(6,7)` looks familiar

    - Subtly, `*foo(..)` generator hasn't actually run yet as it would have with a function

3. We are just creating an _iterator object_ which we assign to the variable `it`, to control the `*foo(..)` generator

    - Then we call `it.next()`, which instructs the `*foo(...)` generator to advance from its current location, stopping either at the next `yield` or end of the generator

4. The result of that `next(..)` call  is an object with a `value` property on t holding whatever value (if anything) was returned from `*foo(...)`

    - In other words, `yield` caused a value to be sent out from the generator during the middle of its execution, kind of like an intermediate `return`

#### Iteration Messaging

In addition to generators accepting arguments and having return values, there's even more input/output messaging capability built into them, via `yield` and `next(..)`

```js
function *foo(x) {
  var y = x * (yield);
  return y;
}

var it = foo( 6 );

// start `foo(..)`
it.next();

var res = it.next( 7 );

res.value;    // 42
```

1. We pass `6` as the parameter `x`. Then we call `it.next()`, and it starts up `*foo(..)`

2. Inside `*foo(..)`, the `var y = x ..` statement starts to be processed, but find `yield`

    - At that point, pauses `*foo(..)` (in the middle of the assignment statement)

    - Essentially request the calling code to provide a result value for the `yield` expression?

    - Next, we call `it.next(7)`, which is passing `7` value back in to be that result of the paused `yield` expression

3. At this point, the assignment statement is `var y = 6 * 7`.

    - `return y` returns `42` as the result of the `it.next(7)` call

Depending on your perspective, there's a mismatch between the `yield` and the `next(..)` call.

In general, you're going to have one more `next(..)` call than you have `yield` statements

1. The first `next(..)` always starts a generator, and runs to the first `yield`
2. The second `next(..)` fulfills the first paused `yield` expression
3. The third `next(..)` would fulfill the second `yield` and so on

##### Tale of Two Questions

Which code you're thinking about primarily will affect whether there's a mismatch or not

```js
var y = x * (yield);
return y;
```

1. The _first_ yield is asking: "What value should I insert here?

    - The _first_ `next()` already run to get the generator up to this point

    - The __second__ `next(...)` call must answer posted by the __first__ `yield`

    - Here's the mismatch!  second-to-first

Let's flip the perspective, look at it from the iterators point of view

Messages can go in both directions:

1. `yield` can send out messages in response to `next(..)` calls
2. `next(..)` can send values to a paused `yield` expression

```js
function *foo(x) {
  var y = x * (yield "Hello");  // <-- yield a value!
  return y;
}

var it = foo( 6 );

var res = it.next();  // first `next()`, don't pass anything
res.value;        // "Hello"

res = it.next( 7 );    // pass `7` to waiting `yield`
res.value;        // 42
```

1. We don't pass a value to the first `next()`. Only a paused `yield` could accept such a value passed by a `next(..)`

2. The __first__ `next()` call is asking: What next value does `*foo(..)` has to give me

    - The answer is made by the __first__ `yield "hello"`

    - There's still an extra `next()`

    - The final `it.next(7)` is asking again what next value the generator will produce. There's no more `yield`, so the `return` answer the question

#### Generator'ing Values

#### Producers and Iterators

Imagine you're producing a series of values where each value has a definable relationship to the previous value

To do this, you'r going to need a stateful producer that remembers the las value it gave out

You can implement something like that using a function closure

```js
var gimmeSomething = (function(){
  var nextVal;

  return function(){
    if (nextVal === undefined) {
      nextVal = 1;
    }
    else {
      nextVal = (3 * nextVal) + 6;
    }

    return nextVal;
  };
})();

gimmeSomething();    // 1
gimmeSomething();    // 9
gimmeSomething();    // 33
gimmeSomething();    // 105
```

This task is a very common design pattern, usually solved by iterators

An iterator is a well-defined interface for stepping through a series of values from a producer. The JS interface for iterators, is to call `next()` each time you want the next value from the producer

We could implement the standard iterator interface for our number series producer:

```js
var something = (function(){
  var nextVal;

  return {
    // needed for `for..of` loops
    [Symbol.iterator]: function(){ return this; },

    // standard iterator interface method
    next: function(){
      if (nextVal === undefined) {
        nextVal = 1;
      }
      else {
        nextVal = (3 * nextVal) + 6;
      }

      return { done:false, value:nextVal };
    }
  };
})();

something.next().value;    // 1
something.next().value;    // 9
something.next().value;    // 33
something.next().value;    // 105
```

1. `[..]` syntax is called __computed property name__, is a ES6 feature. It's a way in an object literal definition to specify an expression and use the result of that expression as the name for the property

2. `Symbol.iterator` is one of ES6 predefined special `Symbol` values

3. `next()` call returns an object with two properties: `done` is a `boolean` signaling the iterators complete status. `value` holds the iteration value

ES6 adds `for...of` loop, a standard iterator can automatically be consumed with native loop syntax

```js
for (var v of something) {
  console.log( v );

  // don't let the loop run forever!
  if (v > 500) {
    break;
  }
}
// 1 9 33 105 321 969
```

1. `for...of` automatically calls `next()` for each iteration, it does not pass any values to the `next()` and will terminate on receiving a `done:true`

You could manually loop over iterators, calling `next()` and checking `done:true`

```js
for (
  var ret;
  (ret = something.next()) && !ret.done;
) {
  console.log( ret.value );

  // don't let the loop run forever!
  if (ret.value > 500) {
    break;
  }
}
// 1 9 33 105 321 969
```

1. This approach is uglier, but let's you pass values to the `next(...)` if necessary

2. Regular `objects` do not come with a default iterator the way `arrays` do.

    - If you want to iterate over the properties of an object (with no particular guarantee of ordering). `Object.keys(..)` returns an `array`

    - `for ( var k of Object.keys(obj)) {....` will loop over an objects keys

    - Similar to a `for...in` loop, but `Object.keys(..)` does not include properties from the `[[Prototype]]` chain

##### Iterables

The `something` object in our example is called an _iterator_, as it has the `nexT()` method on its interface

Other term is _iterable_, which is an _object_ that _contains_ an _iterator_ that can iterate over its values

As of ES6, the way to retrieve an _iterator (the method next)_ from an _iterable (the object)_ is that:

1. The _iterable (object)_ must have a function on it, with the name being the special ES6 symbol value `Symbol.iterator`

2. When this function is called, it returns an _iterator (next())_. Generally each call should return a fresh new _iterator_

```js
[Symbol.iterator]: function(){ return this; }
```

1. That code is making the `something` value -- the interface of the `something` iterator -- ALSO an _iterable_. Then we pass `something` to the `for..of` loop

```js
for (var v of something) {
  ..
}
```

1. `for..of` expects `something` to be an __iterable__. It looks for and calls its `Symbol.iterator` function.
2. That function simple `return this`, so it just gives itself back

##### Generator Iterator

A generator can be treated as a producer of values that we extract one at a time through an _iterator_ interfaces `next()` calls

A generator is not technically an _iterable_, but very similar, when you execute the generator, you get an _iterator_ back

```js
function *foo(){ .. }

var it = foo();
```

We can implement the `something` example like this

```js
function *something() {
  var nextVal;

  while (true) {
    if (nextVal === undefined) {
      nextVal = 1;
    }
    else {
      nextVal = (3 * nextVal) + 6;
    }

    yield nextVal;
  }
}
```

1. `while...true` is OK if it has a `yield`, as the generator will pause at each iteration.

2. Because the generator pauses at each `yield` the state (scope) of the function `*something()` is kept around. Meaning there's no need for the closure boilerplate to preserve variable state across calls

We can use our the new `*something()` generator with a `for...of` loop

```js
for (var v of something()) {
  console.log( v );

  // don't let the loop run forever!
  if (v > 500) {
    break;
  }
}
// 1 9 33 105 321 969
```

1. We called `*something()` to get its _iterator_ for the `for..of`

2. We could not say `for (var v of something)...` because `something` here is a generator which is not an __iterable__

    - We have to call `something()` to construct a producer for the `for...of` loop to iterate over

3. The `something()` call produces an __iterator__ but the `for..of` wants an __iterable (object)__

    - The generators __iterator__ also has a `Symbol.iterator` function on it, which basically does a `return this`

    - In other words, a generators __iterator__ is also an __iterable__

###### Stopping the Generator

Abnormal completion (early termination) of the `for..of` loop, generally caused by a `break`, `return` or an uncaught exception sends a signal to the generators _iterator_ for it to terminate

`for..of` will automatically sends this signal, you can send manually to an _iterator_ by calling `return(..)`

If you specify a `try...finally` clause inside the generator, it will always be run even when the generator is externally completed. Useful to clean up resources like database connections

```js
function *something() {
  try {
    var nextVal;

    while (true) {
      if (nextVal === undefined) {
        nextVal = 1;
      }
      else {
        nextVal = (3 * nextVal) + 6;
      }

      yield nextVal;
    }
  }
  // cleanup clause
  finally {
    console.log( "cleaning up!" );
  }
}
```

```js
var it = something();
for (var v of it) {
  console.log( v );

  // don't let the loop run forever!
  if (v > 500) {
    console.log(
      // complete the generator's iterator
      it.return( "Hello World" ).value
    );
    // no `break` needed here
  }
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

1. `it.return(..)` immediately terminates the generator, which runs the `finally` clause

    - Sets the returned `value` to whatever you passed in to `return(...)`
    - No need to include `break` because the generator _iterator_ is set to `done:true` so the `for..of` will terminate on its next iteration
