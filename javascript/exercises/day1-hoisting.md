# Week 1 - Day 1: var/let/const, Hoisting, Temporal Dead Zone

## Concept Summary

Hoisting is JavaScript's behavior of scanning a scope before running any code, and setting aside memory for variable/function declarations ahead of time.

- var: hoisted AND initialized to `undefined` immediately. Function-scoped (ignores block boundaries like if/for — only the function itself is the boundary).
- let/const: hoisted but NOT initialized. Sit in the "Temporal Dead Zone" (TDZ) until their declaration line actually runs. Block-scoped (confined to the nearest `{ }`, whether that's an if block, loop, or bare block).
- const: same block scoping as let, but cannot be reassigned after declaration.

## Exercise 1

```js
console.log(a);
var a = 5;

console.log(b);
let b = 10;
```

**Answer:**

- `console.log(a)` → `undefined` (var hoisted + auto-initialized)
- `console.log(b)` → `ReferenceError: Cannot access 'b' before initialization` (TDZ)

## Exercise 2

```js
if (true) {
  var x = 1;
  let y = 2;
}
console.log(x);
console.log(y);
```

**Answer:**

- `console.log(x)` → `1` (var is function-scoped, ignores the if-block boundary)
- `console.log(y)` → `ReferenceError: y is not defined` (let is block-scoped, doesn't exist outside the if-block)

## Exercise 3

```js
function test() {
  console.log(typeof c);
  const c = 5;
}
test();
```

**Answer:**

`ReferenceError: Cannot access 'c' before initialization` — even `typeof`, which normally safely returns "undefined" for undeclared variables, cannot touch a variable that's in the TDZ.

## Doubts / Questions I had

**Q: What is hoisting, exactly?**

A: JS scans a scope before executing it and sets aside memory for var/let/const/function declarations ahead of time. var gets auto-initialized to undefined at this point; let/const are reserved but left uninitialized (TDZ) until their actual line runs.

**Q: What's the difference between block scope and function scope?**

A: Function scope (var) means a variable is accessible anywhere inside the function it was declared in, regardless of nested `{ }` like if/for blocks — the only boundary is the function itself. Block scope (let/const) means the variable only exists inside the nearest `{ }` it was declared in — once you exit that block, it's gone.

## Key takeaway (in my own words)

Hoisting means JS sets aside memory for var/let/const/function declarations before running the code — var gets a default value of undefined right away, while let/const exist but are locked (TDZ) until their actual line runs, which is why using them early throws an error instead of just giving undefined.
