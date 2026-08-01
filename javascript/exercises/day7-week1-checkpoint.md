# Week 1 - Day 7: Checkpoint Quiz (var/let/const through Prototypes)

## Result: 6/10 clean on first pass, 4/10 needed correction. Re-drilled the two genuine gaps (var/let scoping terms, TDZ behavior) and passed a 4-question re-check clean. Week 1 JavaScript fundamentals: COMPLETE.

## Question 1: Difference between var and let scope

**Precise answer:** `var` is function-scoped - contained by the nearest enclosing function, ignores block boundaries like if/for. Only attaches to global scope if declared with no function around it at all. `let`/`const` are block-scoped - contained strictly by the nearest `{ }`, whether that's a function, if-block, loop, or bare block.

```js
function example() {
  if (true) {
    var a = 1;  // function-scoped -> survives outside the if-block
    let b = 2;  // block-scoped -> dies at the end of the if-block
  }
  console.log(a); // 1
  console.log(b); // ReferenceError
}
```

**Initial gap:** said "var is not limited to a scope" instead of the precise term "function-scoped."

## Question 2: What is the Temporal Dead Zone (TDZ)?

**Precise answer:** TDZ = the stretch of code from the top of a scope down to the exact line where a let/const declaration actually runs. During that window the variable has been hoisted (JS knows it exists) but is NOT initialized - it's locked. Accessing it during this window throws a `ReferenceError: Cannot access 'x' before initialization`, not `undefined`.

```js
console.log(x); // undefined - var has no TDZ, hoisted AND auto-initialized
var x = 5;

console.log(y); // ReferenceError - y IS in its TDZ right now
let y = 10;
```

**Why "dead":** the variable is completely unusable during that window - not quietly undefined, but throws immediately. This was a deliberate design choice: var's silent undefined behavior was a known flaw that let bugs hide silently; let/const throw loudly instead, on purpose.

**Initial gap:** said accessing a TDZ variable gives `undefined` - corrected to `ReferenceError`.

## Question 3: this inside a regular function object method

```js
const obj = {
  name: "Test",
  regularFn: function () { console.log(this.name); }
};
```

**Answer:** `this` refers to `obj`, because `regularFn` is called as `obj.regularFn()` - regular functions determine `this` based on the call site (how/where they're called), not where they're written. **Correct on first pass.**

## Question 4: Why arrow functions as object methods don't work as expected for this

**Answer:** Arrow functions never bind their own `this` - they always inherit it from the surrounding scope where they were WRITTEN, not where they're called. Since object methods are usually defined in a scope that isn't the object itself (like the top-level module scope), `this` ends up pointing somewhere unexpected, not the object. **Correct on first pass.**

## Question 5: Write map from scratch

```js
function map(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(callback(arr[i]));
  }
  return result;
}
```

**Correct on first pass** - logic fully right. Minor syntax note: initially wrote `int i` instead of `let i` (leftover Java/C++ habit, worth consciously catching going forward).

## Question 6: What does reduce do, and its two arguments

**Answer:** reduce iterates through an array starting from a fixed accumulator value and builds it up through each iteration. Takes two arguments: a callback function and a starting value for the accumulator. The callback itself takes two parameters: accumulator and item (the current array element). **Correct on first pass.**

## Question 7: Difference between spread and rest

**Precise answer:** Rest appears on the LEFT side of `=`, as part of destructuring - it collects whatever wasn't already pulled out by name into a leftover array/object. Spread appears on the RIGHT side, or inside a `{ }`/`[ ]` being constructed - it expands an existing array/object's contents outward into something new.

**Initial gap:** called spread "used for destructuring" - backwards. Destructuring is specifically the rest side (pulling values out); spread is for building/constructing new things.

## Question 8: Rest destructuring output

```js
const { a, ...rest } = { a: 1, b: 2, c: 3 };
console.log(rest);
```

**Answer:** `rest` is `{ b: 2, c: 3 }` - `a` gets pulled out individually by name first, and `...rest` collects only what's left over (b and c), bundled as an object (not an array, since we're destructuring an object here).

**Initial gap:** first answer was `1,2,3` - corrected after tracing through which properties get individually claimed vs left over.

## Question 9: What is a closure, and why does the inner function retain access to outer variables

**Precise answer:** A closure is when an inner function keeps a live connection to its outer function's variables even after the outer function has finished executing. The specific mechanism: JS only cleans up (garbage collects) variables when NOTHING still references them. If the returned inner function's code still references an outer variable (e.g. `count++`), JS sees that active reference and keeps that variable alive in memory specifically so the inner function can keep using it - instead of deleting it like normal.

**Initial gap:** described general "scope" access rather than the specific cause-and-effect: inner function actively referencing outer variables is WHY JS keeps them alive rather than garbage collecting them. Needed two rounds of prompting to land on "the inner function still points to those variables, that's why the reference is maintained."

## Question 10: What is the prototype chain, and why does it exist

**Answer:** An object connected to (pointing at) another object is that object's prototype, and can use the prototype's properties/methods as if they were its own. Purpose: avoid every single object/array needing its own private copy of shared methods - e.g. `.map()`/`.filter()` are defined once on `Array.prototype` and shared across every array, rather than being redefined on each one. **Correct on first pass.**

## Re-drill: var/let scoping + TDZ (4 question re-check, all correct)

1. **Q: Precise scoping term for var vs let?** A: var is function-scoped, let is block-scoped. Correct.
2. **Q: What does accessing a let variable before its declaration throw?** A: ReferenceError, due to TDZ. Correct.
3. **Q: Predict output:**
   ```js
   function test() {
     if (true) {
       var count = 10;
     }
     console.log(count);
   }
   test();
   ```
   A: `10` - var ignores the if-block boundary (function-scoped). Correct.
4. **Q: Predict output:**
   ```js
   console.log(typeof score);
   let score = 100;
   ```
   A: Initially said "number" - self-corrected to ReferenceError, since `score` is in its TDZ at that point (hoisted but not initialized) - even `typeof`'s normal safety net (returning "undefined" for genuinely undeclared variables) doesn't apply here because the variable does exist, just locked.

## Key takeaways for Week 1

- var = function-scoped, hoisted + auto-initialized to undefined
- let/const = block-scoped, hoisted but locked in TDZ until declaration line runs - throws ReferenceError if touched early, even with typeof
- this in regular functions = determined by call site; arrow functions = inherited from where written, never changes
- All array methods share a loop + callback skeleton, differing only in what they do with the callback's return value
- Spread = expanding outward (building new things, right side of =). Rest = collecting leftovers (destructuring, left side of =)
- Closures persist because the inner function actively references outer variables, which prevents JS from garbage collecting them
- Prototype chain = objects linked to other objects for shared method/property access, avoiding duplicate copies of the same methods everywhere