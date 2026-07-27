# Week 1 - Day 3: Array Methods from Scratch

## Concept Summary

Every array method (forEach, map, filter, find, some, every, reduce) shares the same
underlying skeleton: loop over the array, call a callback on each item, and do
something different with the callback's return value depending on the method.

- forEach: calls callback on each item, returns nothing
- map: collects callback's return values into a new array
- filter: keeps items where callback returns true, into a new array
- find: returns first item where callback is true, stops early
- some: returns true as soon as one item passes, stops early
- every: returns false as soon as one item fails, stops early
- reduce: carries an accumulator through the loop, reassigning it to whatever
  the callback returns on each pass

## Why write these from scratch

Not to replace the built-in methods, but to prove understanding of the
mechanism underneath them. Very common live interview question ("implement
map from scratch") specifically because it's easy to use these methods
without understanding them, but hard to write them without understanding
them.

## My implementations

\`\`\`js
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
\`\`\`

## Doubts / Questions I had

**Q: Where is the callback's definition inside these functions?**

A: There isn't one inside the function itself — `callback` is just a parameter
name, a placeholder. The actual behavior of the callback is defined at the
call site, when you pass in your own function. Example:
`myMap([1, 2, 3], (item) => item * 2)` — inside myMap, `callback` becomes
that arrow function. The loop runs `callback(arr[i])`, which actually
executes `(item) => item * 2` with `item` set to the current array element.
myMap itself never knows what the callback does (double, square, etc.) — it
just knows to call whatever function it was given, once per item.

**Q: How does the accumulator in reduce actually work?**

A: Traced through `myReduce([1, 2, 3], (acc, item) => acc + item, 0)` step
by step:
- Before loop: accumulator = 0 (the starting value)
- i=0: accumulator = callback(0, 1) = 0+1 = 1
- i=1: accumulator = callback(1, 2) = 1+2 = 3
- i=2: accumulator = callback(3, 3) = 3+3 = 6
- Loop ends, return 6

The key line is `accumulator = callback(accumulator, arr[i])` — every pass,
accumulator gets overwritten with whatever the callback returns. It's just a
normal variable reassignment inside a loop, not a special mechanism.

## Key takeaway (in my own words)

Reduce takes a callback and a starting value for the accumulator. The
callback takes the accumulator and the current item as arguments, and the
accumulator gets reassigned to whatever the callback returns on every cycle.
Item is just whichever array element the loop is currently on.