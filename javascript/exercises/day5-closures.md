# Week 1 - Day 5: Closures

## WHY closures exist

When a function finishes running, its local variables normally get cleaned up - JS frees that memory since nothing needs it anymore. But sometimes a function needs to remember data between separate calls, without exposing that data as a global variable anyone can mess with.

A closure is what happens when an inner function keeps access to variables from the scope it was created in, even after the outer function has already finished executing. JS specifically keeps that memory alive because the inner function still holds a reference to it - if nothing referenced those variables anymore, JS would clean them up like normal.

## The problem it solves

Without closures, persisting state between function calls would require global variables - risky because any code anywhere can accidentally read or overwrite them. Closures give private state: data that only specific functions can see and change, nothing else can touch directly.

## Example

```js
function makeCounter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

`makeCounter()` runs once, creates `count = 0`, and returns an inner function. Normally `count` would be destroyed once `makeCounter` finishes - but the inner function still references it, so JS keeps it alive in a private bubble only that specific returned function can access.

## Doubts / Questions I had

**Q: Why doesn't calling counterA() twice give a fresh count each time?**

A: Creating the closure and calling the closure are two separate steps that happen at different times.

`const counterA = makeCounter();` runs `makeCounter()` exactly ONCE. During that one run, `count` is created (set to 0), the inner function is created and returned, and that inner function gets stored into `counterA`. After this line, `makeCounter()` is done - it does not run again.

Every later call to `counterA()` does NOT re-run `makeCounter()`. It calls the specific inner function that was returned back when `makeCounter()` ran that one time. That inner function still has a live connection to the exact same `count` variable created back then. So each `counterA()` call reads and mutates that same shared `count` - not a fresh one.

```js
const counterA = makeCounter(); // runs makeCounter() ONCE - creates count #1
const counterB = makeCounter(); // runs makeCounter() AGAIN - creates a NEW, separate count #2

counterA(); // 1
counterA(); // 2
counterB(); // 1 - totally separate closure, own private count
```

Mental model: `makeCounter()` is a machine that, each time you run it, builds a brand new sealed box with a `count` variable locked inside, and hands you the only key (the inner function) to open and increment that specific box. Calling `counterA()` repeatedly just reuses the same box you already have the key to - it doesn't build a new one.

**Q: Why do we call counterA() with parentheses instead of just counterA?**

A: `counterA` (no parentheses) refers to the function itself - just points at it, doesn't run it. `counterA()` (with parentheses) means "actually execute this function right now" - that's what triggers count++ and return count to actually happen.

`counterA` being declared with `const` just means the variable name can't be reassigned to something else later - it says nothing about what kind of value it holds. Here it happens to hold a function (since that's what makeCounter returned), not a number. So we still need `()` to actually run that stored function and get a number out of it.

```js
const counterA = makeCounter(); // calls makeCounter ONCE, stores the returned FUNCTION into counterA
counterA();                     // calls the function stored inside counterA
```

Two separate function calls at two separate moments.

## My own closure, written from scratch

```js
function makeCounter() {
  let counter = 0;
  return function () {
    counter++;
    return counter;
  };
}
```

Verified behavior:
```js
const counterA = makeCounter();
counterA(); // 1
counterA(); // 2
counterA(); // 3

const counterB = makeCounter();
counterB(); // 1 - separate closure, own counter, unrelated to counterA's
```

## Key takeaway (in my own words)

A closure is the inner function keeping access to variables from its outer scope, even though the outer function has fully finished executing - the memory is kept alive specifically because the inner function still holds a reference to it, allowing that inner function to keep reading and updating those variables across multiple calls.