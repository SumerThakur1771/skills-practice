# Week 2 - Day 9: Async/Await

## WHY async/await exists

Promise chains with .then().then().then() work fine, but get messy with several steps - nested logic, awkward error handling. async/await is newer syntax that lets asynchronous code be written to LOOK like normal, synchronous, top-to-bottom code, while still working exactly like Promises underneath. It doesn't replace Promises - it's a cleaner way to USE them.

## The two keywords

**async** - put before a function definition, means "this function always returns a Promise," even if you just return a plain value inside it.

```js
async function greet() {
  return "hello";
}
greet().then((value) => console.log(value)); // "hello" - auto-wrapped in a Promise
```

**await** - can ONLY be used inside an async function. Pauses execution at that line until the Promise on the right resolves, then gives the resolved value directly - no .then() needed.

## Why await requires an async function - the actual rule

await literally PAUSES execution of the function it's inside, until the Promise resolves. That pausing only makes sense within a function designed to handle being paused/resumed - which is exactly what marking a function async sets up. A normal synchronous function (or top-level script code) isn't built to pause like that. Using await outside an async function throws a SyntaxError.

Contrast with .then() - .then() doesn't pause anything, it just says "whenever this resolves, run this callback," and the surrounding code keeps running immediately without waiting. That's why .then() can be called anywhere, no special wrapping needed.

## The critical two-job split (this was my main point of confusion)

There are two separate jobs that are easy to mix up:

- JOB 1: A function that CREATES and returns a Promise - does NOT need to be async, does NOT use await inside it. Just builds `new Promise(...)` and returns it.
- JOB 2: A function that CONSUMES/uses a Promise - THIS is the one that needs to be async, so it can use await inside to pause and grab the resolved value.

## Example - rewriting checkAge with async/await

```js
// JOB 1: creates and returns a promise. NOT async, no await inside. UNCHANGED from Day 8.
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

// JOB 2: consumes/uses that promise. THIS one is async, uses await inside.
async function runAgeCheck() {
  try {
    const result = await checkAge(20);
    console.log(result);
  } catch (error) {
    console.log(error);
  }
}

runAgeCheck();
```

Only the function that actually uses the word await needs to be marked async. The function being awaited (checkAge) doesn't need to change at all, regardless of whether someone consumes it with .then() or with await later.

.catch() becomes a regular try/catch block - same error-handling pattern as normal synchronous code. If the awaited Promise rejects, the catch block runs.

## My practice exercise - written and debugged myself

Goal: rewrite ageCheck (same logic as Day 8's checkAge) and write an async wrapper using await + try/catch.

**Mistake made and fixed:** Initially placed `return promise;` INSIDE the Promise executor callback `(resolve, reject) => { ... return promise; }` instead of inside `ageCheck` itself, after the Promise construction finished. This meant `ageCheck` never actually returned anything (would be `undefined`), and the return inside the executor did nothing meaningful since the executor's return value is ignored by Promise. Fixed by moving `return promise;` to after the closing `});` of the Promise construction, as `ageCheck`'s own return statement.

**Final correct code:**

```js
function ageCheck(age) {
  const promise = new Promise((resolve, reject) => {
    if (age >= 18) {
      resolve("You are an adult");
    } else {
      reject("You are a minor");
    }
  });
  return promise;
}

async function runAgeCheck() {
  try {
    const result = await ageCheck(20);
    console.log(result);
  } catch (error) {
    console.log(error);
  }
}

runAgeCheck();
```

## Doubts / Questions I had

**Q: Why is it called "synchronous-looking," and what makes async/await actually better than Promises - is "too many .then() is confusing" the whole reason?**

A: Important distinction first - async/await code is NOT actually synchronous. Under the hood it's still fully asynchronous, still using Promises, still non-blocking. "Synchronous-looking" refers only to how the code READS (top to bottom, like normal step-by-step code), not how it actually executes. JS still pauses that specific function at the await line and lets other code run elsewhere meanwhile.

The full list of real reasons async/await is preferred (interview-ready):

1. **Readability / avoiding "callback hell" or promise chain sprawl** - long .then().then().then() chains get visually nested and hard to trace, especially with conditional logic mixed in. Async/await reads like a normal function, top to bottom.

2. **Unified error handling** - .then() chains use .catch(), a DIFFERENT mechanism from how errors are handled in regular synchronous code (try/catch). Async/await lets you use the SAME try/catch pattern used everywhere else in JS - one mental model instead of two separate systems.

3. **Easier to mix with normal control flow (loops, conditionals)** - genuinely technical advantage, not just style. Writing a for loop where each iteration needs to wait for an async result is awkward with pure .then() chains (often requires manually building a promise chain). With await, it just works naturally inside a normal for loop:
   ```js
   async function processAll(items) {
     for (const item of items) {
       const result = await doSomethingAsync(item);
       console.log(result);
     }
   }
   ```

4. **Easier debugging** - stack traces and breakpoints behave more predictably since code structure matches actual execution order, versus .then() callbacks which can make stack traces harder to trace back to their origin.

**Common misconception to avoid in an interview:** async/await is NOT faster than Promises and does NOT change how concurrency actually works. It's syntax sugar over the exact same Promise mechanism - same performance, same underlying behavior underneath. Saying "async/await is faster" would be a wrong answer.

**Interview-ready one-liner:** "Async/await is syntax sugar over Promises - same behavior underneath, not actually synchronous, still non-blocking. It's preferred because it reads top-to-bottom like normal code instead of nested .then() chains, it lets you use standard try/catch for errors instead of .catch(), and it's much easier to combine with normal control flow like loops and conditionals."

## Key takeaway (in my own words)

A function that builds and returns a Promise doesn't need to be async or use await - it just constructs the Promise and returns it, same as always. Only the function that wants to WAIT for that promise's result needs to be marked async, so it can use await inside a try/catch to pause until the promise resolves or rejects.