# Week 1 - Day 2: Functions, Function Expressions, Arrow Functions, `this` Binding

## Concept Summary

- Function declarations: fully hoisted, callable before their line in the file.
- Function expressions: assigned to a variable. The variable name is hoisted, but the function body isn't attached until that line runs.
- Arrow functions: shorter syntax, but the key difference is `this` binding, not syntax.
- Regular functions: `this` is determined dynamically by how the function is called (the call site) — e.g. `obj.method()` binds `this` to `obj`.
- Arrow functions: do NOT have their own `this`. They inherit `this` from the surrounding scope where they were written (lexical scoping), and that binding never changes based on how they're called. Even `.call()`/`.apply()`/`.bind()` can't override it.

## Exercise 1 — Regular fn vs arrow fn `this`

```js
const obj = {
  name: "Sumer",
  regularFn: function () {
    console.log(this.name);
  },
  arrowFn: () => {
    console.log(this.name);
  },
};

obj.regularFn();
obj.arrowFn();
```

**Answer:**

- `obj.regularFn()` → `"Sumer"` — called as `obj.regularFn()`, so `this` refers to `obj`.
- `obj.arrowFn()` → `undefined` — the arrow function has no `this` of its own, so it looks up to the scope where it was written (top-level, outside `obj`). At the top level, `this` refers to the global object (`window` in a browser), which has no `name` property, so `this.name` is `undefined`. It does not throw an error — it just resolves to nothing meaningful.

**Rule of thumb:** Method attached to an object where `this` should mean "the object it's called on" → regular function. Callback where you want `this` to stay locked to the surrounding scope no matter how/when it's called later → arrow function.

## Exercise 2 — Counter object with `this`

Goal: object with a `count` property and an `increment` method (regular function) that increments it.

**Mistakes made along the way and fixes:**

1. Initially used `let count = 0;` and `let increment = function(){...}` inside the object literal — wrong, because object literal properties don't use `let`/`const`/`var`, they're just `key: value` pairs.
2. Then used `=` between key and value (`count = 0`) — wrong, object literals use `:` not `=`. `=` is for variable assignment, `:` is for object literal key-value pairs.
3. Initially wrote `count++` inside `increment` — wrong, because there's no standalone `count` variable, only `counter.count`. Needed `this.count++` to reference the object's own property from inside its method.
4. Also mixed semicolons between object properties instead of commas — object literal entries are comma-separated, not semicolon-separated.

**Final correct code:**

```js
const counter = {
  count: 0,
  increment: function () {
    this.count++;
  },
};
counter.increment();
counter.increment();
counter.increment();
console.log(counter.count);
```

**Output:** `3` — each `counter.increment()` call runs with `this` bound to `counter` (since it's called as `counter.increment()`), so `this.count++` increments `counter.count` from 0 → 1 → 2 → 3.

## Key takeaway (in my own words)

Regular functions decide what `this` means based on how they're called, so they're the right choice for object methods where `this` needs to point to the object itself. Arrow functions permanently keep the `this` from wherever they were written, so they're better for callbacks where you want `this` to stay the same no matter how or when they later get called.