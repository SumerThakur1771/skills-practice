# Week 1 - Day 3: Array Methods — Standard Usage + From Scratch

## Concept Summary

Every array method (forEach, map, filter, find, some, every, reduce) shares the same underlying skeleton: loop over the array, call a callback on each item, and do something different with the callback's return value depending on the method.

## The Standard (Built-In) Methods — How to Use Them

### forEach
Loops through every item, runs the callback, returns nothing (`undefined`). Pure "do something for each item."

```js
[1, 2, 3].forEach((item) => console.log(item));
// logs: 1, 2, 3
```

### map
Loops through every item, runs the callback, and collects whatever the callback returns into a **new array** — same length as the original.

```js
const doubled = [1, 2, 3].map((item) => item * 2);
// doubled = [2, 4, 6]
```

### filter
Loops through every item, runs a true/false test callback, and keeps only the items where it returned `true`. Returns a new, possibly shorter array.

```js
const evens = [1, 2, 3, 4].filter((item) => item % 2 === 0);
// evens = [2, 4]
```

### find
Loops through, runs a true/false test callback, returns the **first item** where it's `true`. Stops looking as soon as it finds one. Returns `undefined` if nothing matches.

```js
const firstEven = [1, 3, 4, 5].find((item) => item % 2 === 0);
// firstEven = 4
```

### some
Loops through, runs a true/false callback, returns `true` as soon as **any single item** passes. Returns `false` if none pass.

```js
const hasEven = [1, 3, 5].some((item) => item % 2 === 0);
// hasEven = false
```

### every
Loops through, runs a true/false callback, returns `true` only if **all items** pass. Stops and returns `false` the moment one fails.

```js
const allEven = [2, 4, 6].every((item) => item % 2 === 0);
// allEven = true
```

### reduce
The most flexible one. Loops through the array carrying forward a single "accumulator" value that builds up as it goes. Takes a callback (accumulator, item) and a starting value.

```js
const sum = [1, 2, 3].reduce((accumulator, item) => accumulator + item, 0);
// starts at 0 -> 0+1=1 -> 1+2=3 -> 3+3=6
// sum = 6
```

Traced step by step:
- Before loop: accumulator = 0 (the starting value)
- i=0: accumulator = callback(0, 1) = 0+1 = 1
- i=1: accumulator = callback(1, 2) = 1+2 = 3
- i=2: accumulator = callback(3, 3) = 3+3 = 6
- Loop ends, return 6

Key line to remember: `accumulator = callback(accumulator, item)` — every pass, accumulator gets overwritten with whatever the callback returns.

## Why write these from scratch

Not to replace the built-in methods — you'd never use your own version in real code. The point is proving understanding of the mechanism underneath. Very common live interview question ("implement map from scratch") specifically because it's easy to use these methods without understanding them, but hard to write them without understanding them.

## My From-Scratch Implementations

```js
function myForEach(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i]);
  }
}

function myMap(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(callback(arr[i]));
  }
  return result;
}

function myFilter(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i])) {
      result.push(arr[i]);
    }
  }
  return result;
}

function myFind(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i])) {
      return arr[i];
    }
  }
  return undefined;
}

function mySome(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i])) {
      return true;
    }
  }
  return false;
}

function myEvery(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (!callback(arr[i])) {
      return false;
    }
  }
  return true;
}

function myReduce(arr, callback, startingValue) {
  let accumulator = startingValue;
  for (let i = 0; i < arr.length; i++) {
    accumulator = callback(accumulator, arr[i]);
  }
  return accumulator;
}
```

## Doubts / Questions I had

**Q: Where is the callback's definition inside these from-scratch functions?**

A: There isn't one inside the function itself — `callback` is just a parameter name, a placeholder. The actual behavior of the callback is defined at the call site, when you pass in your own function. Example: `myMap([1, 2, 3], (item) => item * 2)` — inside myMap, `callback` becomes that arrow function. The loop runs `callback(arr[i])`, which actually executes `(item) => item * 2` with `item` set to the current array element. myMap itself never knows what the callback does (double, square, etc.) — it just knows to call whatever function it was given, once per item.

**Q: How does the accumulator in reduce actually work?**

A: See the traced example above under the reduce section — accumulator gets reassigned to whatever the callback returns on every loop pass, starting from the given starting value.

## Key takeaway (in my own words)

Reduce takes a callback and a starting value for the accumulator. The callback takes the accumulator and the current item as arguments, and the accumulator gets reassigned to whatever the callback returns on every cycle. Item is just whichever array element the loop is currently on. All array methods share the same loop-and-callback skeleton — they just differ in what they do with the callback's return value.