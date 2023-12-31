# Event loop

JavaScript uses an event loop to run code and manage events. This approach, different from languages like C and Java, processes tasks in a queue.

## Stack

```js
function foo(b) {
  const a = 10;
  return a + b + 11;
}

function bar(x) {
  const y = 3;
  return foo(x * y);
}

const baz = bar(7); // assigns 42 to baz
```

In JavaScript, function calls create a stack of frames:

1. Calling `bar` makes a frame with its arguments and variables.
2. `bar` calls `foo`, adding a new frame on top for `foo`'s arguments and variables.
3. When `foo` finishes, its frame is removed.
4. When `bar` finishes, the stack is empty.

Arguments and variables exist outside the stack, so nested functions can use them even after their outer function ends.

## Heap

Objects are allocated in a heap which is just a name to denote a large (mostly unstructured) region of memory.

## Queue

JavaScript's runtime uses a message queue where each message is linked to a function. During the event loop, it processes messages starting with the oldest. Each message is removed from the queue, and its function is called, creating a new stack frame. This continues until the stack is empty, then moves to the next message.

## Run to completion

In JavaScript, the "run-to-completion" model means each message in the queue is fully processed before the next one starts. This ensures that a function, once started, runs entirely without interruption, unlike languages like C where threads can be paused.

However, if a message processing takes too long, it can delay user interactions in web applications, leading to warnings like "a script is taking too long to run". To avoid this, it's best to keep message processing brief and split larger tasks into smaller messages.

## Adding messages

In web browsers, when an event happens and there's a listener for it, a message is added to the queue. If there's no listener, the event is ignored. For example, a click event adds a message if there's a click listener.

`setTimeout` queues a message after a specified delay (minimum time, not guaranteed). If the queue is empty and the stack is clear, it processes after the delay. If the queue has messages, `setTimeout` waits its turn.

Here's an example showing `setTimeout` doesn't run immediately after its delay:

```javascript
const seconds = new Date().getTime() / 1000;

setTimeout(() => {
  // prints after 2 seconds, not immediately after 500 milliseconds.
  console.log(`Ran after ${new Date().getTime() / 1000 - seconds} seconds`);
}, 500);

while (true) {
  if (new Date().getTime() / 1000 - seconds >= 2) {
    console.log("Good, looped for 2 seconds");
    break;
  }
}
```

## Zero delay

Setting a zero delay in `setTimeout` doesn't mean the callback will execute immediately after 0 milliseconds. Instead, it waits for the current queue of tasks to finish. The zero delay is the minimum, not a guaranteed time for the callback to run.

## Never Blocking

JavaScript's event loop model means it never gets blocked. I/O operations use events and callbacks, allowing other tasks like user input to be processed while waiting for things like IndexedDB queries or fetch() requests.

# Promise

A Promise is like a placeholder for a value that might not be known right away. It's used in asynchronous programming, where operations don't finish immediately. A Promise can end in three ways:

1. **Pending**: The initial state, where the outcome isn't known yet.
2. **Fulfilled**: The operation was successful.
3. **Rejected**: The operation failed.

When a Promise is either fulfilled or rejected, it triggers specific functions set up earlier. These functions handle the successful or failed result. Even if the Promise is already resolved when you set up these handlers, they will still work properly, avoiding any timing issues.

Once a Promise is either fulfilled or rejected, it's considered "settled" and doesn't stay in the pending state.

## Resolved

The term "resolved" in promises means it's set to its final state: either done (fulfilled), failed (rejected), or matching another promise's state. For instance:

```javascript
new Promise((resolveOuter) => {
  resolveOuter(
    new Promise((resolveInner) => {
      setTimeout(resolveInner, 1000);
    })
  );
});
```

In this code, the outer promise resolves immediately but waits for the inner promise, which takes 1 second to complete. Once a promise is resolved, its state can't change.

## Further action after promise settles

In promises, `.then()`, `.catch()`, and `.finally()` are used for further actions after the promise settles. These methods return new promises, allowing for chaining.

- `.then()` takes two functions: one for success (fulfilled) and one for failure (rejected). It returns a new promise, enabling chaining. For example:

```javascript
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("foo");
  }, 300);
});

myPromise
  .then(handleFulfilledA, handleRejectedA)
  .then(handleFulfilledB, handleRejectedB)
  .then(handleFulfilledC, handleRejectedC);
```

If `.then()` lacks a rejection function, the chain continues normally. You can skip rejection functions until a final `.catch()`.

- Handling rejections in every `.then()` affects the chain. If you need immediate error handling, throw an error. Otherwise, it's simpler to use a final `.catch()`, which is like `.then()` but only for rejections.

```javascript
myPromise
  .then(handleFulfilledA)
  .then(handleFulfilledB)
  .then(handleFulfilledC)
  .catch(handleRejectedAny);
```

## Handling multiple async tasks

The Promise class in JavaScript provides four static methods for handling multiple asynchronous tasks:

1. **Promise.all()**: Succeeds if all promises in the iterable succeed. Fails if any promise fails.
2. **Promise.allSettled()**: Succeeds when all promises in the iterable are either fulfilled or rejected.
3. **Promise.any()**: Succeeds if any promise in the iterable succeeds. Fails if all promises fail.
4. **Promise.race()**: The result follows the first promise in the iterable to either fulfill or reject.

These methods work with an iterable of promises and return a new promise. They also support subclassing, allowing use with Promise subclasses. To subclass, the constructor must have the same format as the Promise constructor and include a static resolve method like `Promise.resolve()`.

Remember, JavaScript is single-threaded, so it can only run one task at a time. While promises may seem concurrent, they're actually executed one after another. True parallel execution in JavaScript requires worker threads.

# User Agent

A user agent is a software that acts for a user, like a web browser or a bot. It sends a User-Agent HTTP header, or UA string, with each web request, identifying the browser, version, and operating system. Some may send a fake UA string, a practice known as user agent spoofing.

You can access the UA string in JavaScript with `NavigatorID.userAgent`. An example UA string is: "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:35.0) Gecko/20100101 Firefox/35.0".
