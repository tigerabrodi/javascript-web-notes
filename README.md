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

# Microtasks and the JavaScript runtime environment

Understanding JavaScript's runtime is key when debugging or optimizing task timing and scheduling. Originally, JavaScript was single-threaded, suitable for the less complex computing era. However, as computers evolved into multi-core systems and JavaScript usage expanded, enhancements were necessary.

To overcome single-threaded limitations, Web APIs like `setTimeout()` and `setInterval()` were introduced. These allow task scheduling and support multi-threaded development. Understanding these features, including `queueMicrotask()`, is crucial to grasp JavaScript's approach to code scheduling and execution in today's complex applications.

## Execution context

In JavaScript, code runs in an execution context, of which there are three types:

1. **Global Context:** For code outside any function.
2. **Local Context:** Each function runs in its own context.
3. **Eval Context:** Created by using `eval()`.

Each context is a scope level. When code starts, a new context is created and then destroyed when the code exits. Consider a JavaScript program with multiple function calls:

- The global context is created at program start.
- Calling a function like `greetUser()` creates a new local context.
- Inside `greetUser()`, calling another function, say `localGreeting()`, creates another context.
- After `localGreeting()` finishes, its context is destroyed, returning to the `greetUser()` context.
- After `greetUser()` finishes, its context is destroyed, returning to the global context.
- This process repeats for each function call.

Each context manages its own variables and tracks the next execution line. This system handles local/global variables and function calls/returns.

Note on recursive functions: Each call creates a new context, which requires more memory for each level of recursion.

## Run JavaScript Run

JavaScript's runtime uses agents to execute code, each with its own execution contexts, a main thread, worker threads, and task and microtask queues. Here's a breakdown:

**Event Loops:**

- Agents run on event loops, handling user events and enqueuing tasks.
- Each agent's event loop manages tasks, microtasks, rendering, and painting.
- The main thread, shared by a website's code and the browser's UI, runs on this loop.
- There are three types of event loops: Window, Worker, and Worklet.

**Tasks vs. Microtasks:**

- Tasks: Scheduled JavaScript, executed one per event loop iteration.
- Microtasks: Run after each task, executed continuously until the queue is empty.

**Problems:**

- Code running on the main thread can slow down or stall the browser.
- Intensive tasks can cause sluggish performance.

**Solutions:**

- **Web Workers:** Run scripts in separate threads, reducing main thread workload.
- **Asynchronous JavaScript:** Using promises, the main code runs while waiting for tasks.
- **Microtasks:** Schedule tasks before the next event loop iteration, helping with performance. `queueMicrotask()` exposes this mechanism, standardizing microtask handling across browsers.

In summary, JavaScript's runtime model, with its event loops, task and microtask queues, and solutions like web workers and `queueMicrotask()`, addresses the challenges of single-threaded execution and enhances performance and reliability.

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

# Microtask in JS with queueMicrotask()

## Microtasks

The difference between tasks and microtasks in JavaScript may seem small, but it's significant:

1. **Microtask Queue Processing:** After each task, the event loop checks for microtasks. If found, it runs all microtasks in the queue. This means microtasks are processed several times within a single event loop iteration, not just once like tasks.

2. **Continuous Execution of Microtasks:** If a microtask adds more microtasks to the queue (using `queueMicrotask()`), these new microtasks are executed before moving on to the next task. The event loop keeps running microtasks until the queue is empty, even if new ones are added during the process. This means normal tasks can't run until the microtask queue is empty, once it has started processing.

## Using Microtasks

Enqueueing microtasks should be done judiciously, primarily in frameworks, libraries, or specific scenarios where no other solution fits. The `queueMicrotask()` method offers a standard way to schedule microtasks, avoiding the issues that arise when using promises for this purpose, like exception handling differences and additional overhead.

**When to Use Microtasks:**

1. **Consistent Ordering:** Microtasks ensure consistent task ordering, even with synchronous results, minimizing user-perceived delays.
2. **Conditional Promises:** In scenarios where promises are used conditionally, microtasks can help maintain consistent operation order. For example, balancing a conditional statement where one branch uses promises and the other doesn't.

3. **Batching Operations:** Microtasks are useful for batching multiple requests into a single operation. This can optimize performance by reducing overhead and delays from multiple individual operations.

Example of Batching with Microtasks:

```javascript
const messageQueue = [];

let sendMessage = (message) => {
  messageQueue.push(message);
  if (messageQueue.length === 1) {
    queueMicrotask(() => {
      const json = JSON.stringify(messageQueue);
      messageQueue.length = 0;
      fetch("url-of-receiver", json);
    });
  }
};
```
