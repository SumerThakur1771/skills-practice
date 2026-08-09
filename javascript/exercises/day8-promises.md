# Week 2 - Day 8: Promises

## WHY promises exist

Some operations take time to finish - fetching data from a server, reading a file, waiting on a timer. JS doesn't pause and wait around for these; it keeps running other code while the slow thing happens in the background. Promises are the tool JS gives to say "run this code once that slow thing eventually finishes" instead of constantly checking "is it done yet?"

## What a Promise is

An object representing a value that doesn't exist yet, but will (or won't) exist eventually. Always ends up in one of three states:

- Pending - still waiting, not resolved or rejected yet
- Fulfilled - the operation succeeded, now has a value
- Rejected - the operation failed, now has an error

## Creating a basic Promise

```js
const myPromise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("It worked!");
  } else {
    reject("It failed!");
  }
});
```

`resolve` and `reject` are two functions handed automatically inside the Promise constructor. Calling `resolve(value)` moves the promise to fulfilled with that value. Calling `reject(error)` moves it to rejected with that error.

## Using a Promise - .then and .catch

```js
myPromise
  .then((result) => {
    console.log(result); // runs if resolve() was called
  })
  .catch((error) => {
    console.log(error); // runs if reject() was called instead
  });
```

## Chaining .then

Each `.then` in a chain receives whatever the PREVIOUS `.then` returned - like a relay passing a baton.

```js
const myPromise = new Promise((resolve) => resolve(5));

myPromise
  .then((value) => {
    console.log(value); // 5
    return value * 2;
  })
  .then((value) => {
    console.log(value); // 10 - what the previous .then returned
    return value + 1;
  })
  .then((value) => {
    console.log(value); // 11
  });
```

## .catch placement in a chain

A single `.catch` at the end catches an error from ANY step earlier in the chain - no need for one after every `.then`.

## Promise.all

Runs multiple promises in PARALLEL and waits until all of them finish before continuing.

```js
const p1 = new Promise((resolve) => resolve("first"));
const p2 = new Promise((resolve) => resolve("second"));
const p3 = new Promise((resolve) => resolve("third"));

Promise.all([p1, p2, p3]).then((results) => {
  console.log(results); // ["first", "second", "third"]
});
```

Takes an array of promises, gives back a single promise that resolves once every one has finished, with results in the same order they were passed in. If even one rejects, the whole Promise.all immediately rejects too.

## My practice exercise - written and debugged myself

Goal: checkAge(age) function that returns a Promise - resolves "You are an adult" if age >= 18, rejects "You are a minor" if under 18.

**Final correct code:**

```js
function checkAge(age) {
  const promise = new Promise((resolve, reject) => {
    if (age >= 18) {
      resolve("You are an adult");
    } else {
      reject("You are a minor");
    }
  });
  return promise;
}

checkAge(20)
  .then((value) => {
    console.log(value); // "You are an adult"
  })
  .catch((error) => {
    console.log(error);
  });

checkAge(15)
  .then((value) => {
    console.log(value);
  })
  .catch((error) => {
    console.log(error); // "You are a minor"
  });
```

## Doubts / Questions I had

**Q: Why doesn't function parameter syntax allow let/const, e.g. `function checkAge(let age)`?**

A: Function parameters are bare names, never prefixed with let/const/var. Fixed `function checkAge(let age)` to `function checkAge(age)`.

**Q: Why can't I call `.then`/`.catch` on a variable created inside the function, from outside the function?**

A: Initially built the promise and called `.then`/`.catch` on a local variable called `promise` OUTSIDE the function - invalid, since `promise` was declared inside `checkAge` and doesn't exist in the outer scope (basic scope rule from Day 1). The function needs to `return` the promise, and the CALLER attaches `.then`/`.catch` to the function's return value, not to some internal variable name that only exists inside the function.

**Q: Why doesn't calling `.then` directly on the function name work?**

A: Initially forgot to actually CALL the function with `()` and an argument before chaining `.then` - `checkAge` is a function, calling `.then` on it directly (without calling it first) doesn't work since `.then` only exists on the PROMISE it returns, not on the function itself. Also typo'd the function name (`checkage` vs `checkAge`) along the way - JS is case-sensitive, names must match exactly.

## Key takeaway (in my own words)

A function that creates a Promise needs to return that promise so whoever calls the function can attach `.then`/`.catch` to the actual returned promise - internal variables inside the function aren't accessible from outside it. The function builds and returns the promise; the caller is responsible for handling what it resolves or rejects with.