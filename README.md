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
