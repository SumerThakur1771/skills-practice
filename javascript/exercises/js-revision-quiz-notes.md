# JS Revision Quiz — Full Notes (Q1–Q21)

Verbatim from the actual revision sessions — self-test format: cover the "A:" line, answer cold, then check. Wrong-answer corrections are kept in, since those are exactly the spots worth double-checking before the real interview.

---

## Q1: `var` vs `let` scoping

**Q:** What's the difference between `var` and `let` in terms of scope?

**A:** `var` is **function-scoped** — contained by the nearest enclosing function (or global if declared outside any function). It ignores block boundaries (`if`/`for`). `let` is **block-scoped** — contained by the nearest `{ }`, whether that's a function, an `if`, a loop, or a bare block. It's narrower than `var`, not "the same level."

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

**⚠️ Repeated mistake flagged — double-check this before the interview:** answered "var = global level, let = function level" — backwards on both counts. This same exact mix-up (function-scoped vs block-scoped, not "which level") showed up in an earlier checkpoint too. Deliberately re-verify this one out loud before saying it in a real interview.

---

## Q2: Temporal Dead Zone (TDZ)

**Q:** What is the Temporal Dead Zone (TDZ), and what happens if you try to access a `let`/`const` variable before its declaration line runs?

**A:** TDZ is the zone from the **top of the scope** down to the **exact line where the `let`/`const` declaration actually runs** — not "declaration onward." During that window, the variable has been hoisted (JS knows it exists) but is NOT initialized. Accessing it during that window throws a `ReferenceError`.

```js
console.log(x); // ReferenceError - x is in its TDZ right now
let x = 5;
console.log(x); // 5 - past the declaration line, TDZ is over
```

**⚠️ Flagged gap:** answered the direction backwards — said TDZ runs *from the declaration up till where it's used*. Correct direction is *top of scope → declaration line*, i.e., TDZ ends once the declaration executes, it doesn't begin there. Second real gap on the same concept (direction of the zone) — worth double-checking before the interview.

---

## Q3: `this` in regular functions vs arrow functions

**Q:** What does `this` refer to inside a regular function used as an object method, versus inside an arrow function used the same way?

**A:** For a **regular function** method, `this` is determined by the **call site** — whatever object is directly before the dot when called.

```js
const obj = {
  name: "Sumer",
  regularFn: function () { console.log(this.name); }
};
obj.regularFn(); // "Sumer" - called as obj.regularFn(), so this = obj
```

For an **arrow function**, `this` is NOT determined by how it's called — it never has its own `this`. It looks to wherever it was physically *written* (lexical scope) and borrows `this` from there, permanently.

```js
const obj = {
  name: "Sumer",
  arrowFn: () => { console.log(this.name); }
};
obj.arrowFn(); // undefined - this comes from where it was WRITTEN (top-level), not from obj
```

Key clarifying point worked through: object literal `{ }` braces do **not** create their own scope — they're invisible for `this` purposes. An arrow function written inside an object behaves exactly as if the object wrapper wasn't there at all — it copies whatever `this` would be at that exact point in the surrounding code.

**The clean one-liner for interviews:** *"Regular function's `this` = whoever calls it (the object before the dot). Arrow function's `this` = whatever the surrounding code's `this` already was, regardless of how or by what it gets called."*

**Doubts worked through (multiple passes needed):** Initial answer mixed up "scope" (variable lookup) with "`this` binding" (call-site mechanism) — these are different things. Second attempt said arrow function's `this` refers to "the parent of that object" — corrected: objects don't have a "parent" in this sense; it's about surrounding *code* scope, not object hierarchy. Confirmed understanding by stating it independently: "it doesn't consider `this` pointing towards the object that called it, rather it refers to the level above it, and in here that happens to be global" — this phrasing was confirmed correct.

---

## Q4: Closures (why, not just what)

**Q:** What is a closure, and why does the inner function keep access to the outer function's variables even after the outer function has finished running?

**A (definition — confirmed correct on first try):** A closure is when an inner function keeps access to its outer function's variables even after the outer function has finished running.

**A (the "why" — needed correction):** Normally, when a function finishes running, JS garbage-collects its local variables since nothing needs them. But JS only garbage-collects a variable when **nothing still references it**. If the returned inner function's code still actively references an outer variable (like `count` in `count++`), JS sees that live reference and specifically keeps the variable alive in memory instead of deleting it — precisely so the inner function can keep using it.

```js
function makeCounter() {
  let count = 0;
  return function () {
    count++; // this reference is WHY count survives
    return count;
  };
}
```

**⚠️ Flagged gap:** initial "why" answer was "the variable still might be called in there... then the program removes access to it" — vague, not the actual mechanism. Correct mechanism: *the inner function's active reference is specifically what prevents garbage collection* — it's not "might be used," it's "JS detects a live reference and refuses to delete it."

---

## Q5: Prototype chain

**Q:** What is the prototype chain, and why does it exist?

**A:** The problem it solves: without prototypes, every array would need its own private copy of `.map()`, `.filter()`, `.push()`, etc. — massive memory waste. Prototypes let objects **link to** another object and borrow its methods. `.map()` is defined once on `Array.prototype`; every array just points to that shared object.

The "chain": when you call a method, JS first checks if the object has it **directly on itself**. If not, it checks the object it's linked to (its prototype). If not there, it keeps walking up the chain until found or until reaching `null`.

```js
const arr = [1, 2, 3];
arr.map(x => x);
// JS checks: does arr itself have .map? No.
// -> checks arr's prototype (Array.prototype): does IT have .map? Yes -> uses it.
```

**Confirmed understanding:** `arr`'s prototype is `Array.prototype` — the shared object every array links to, where `.map()`/`.filter()`/`.push()` actually live, defined once and reused by every array instance.

**Note:** first reaction was "totally forgot this one" — rebuilt from scratch successfully once walked through the memory-waste problem it solves.

---

## Q6: `map` from scratch

**Q:** Write `map` from scratch — what does it need to do?

**A (final, correct):**
```js
function map(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(callback(arr[i]));
  }
  return result;
}
```

**⚠️ Mistakes worked through on the way there:**
1. First attempt used Java/C++ syntax (`int result = new int[]`, `int i`) — not valid JS. Corrected to `const result = []` and `let i`.
2. First attempt also only took `arr` as a parameter (no `callback`), and just copied items unchanged (`result.push(arr[i])`) — didn't actually transform anything. The core confusion: not understanding *why* a callback is needed.
3. **Key clarifying trace that fixed it:** given `map([1,2,3], item => item * 2)`, `result.push(arr[i])` pushes `1` unchanged (just a copy), while `result.push(callback(arr[i]))` actually *calls* the callback on `1`, runs `1 * 2`, and pushes the *result* (`2`) instead. This is what finally made "why callback" click — confirmed in own words: *"map walks through every element and applies whatever transformation function the caller provides, collecting the results into a new array."*

**Style note:** `const result = new Array()` works but `const result = []` (array literal) is the more idiomatic/common style.

---

## Q7: Spread vs Rest

**Q:** What's the difference between spread and rest — same syntax (`...`), different jobs?

**A:** **Spread** expands/combines — used on the right side of `=`, for combining arrays/objects or making copies:
```js
const heroes = [...marvel, ...dc]; // combines two arrays
const copy = [...original];        // copies
```

**Rest** collects leftover properties into a new, independent variable — used in destructuring:
```js
const car = { name: "honda", year: "2023", model: "sports" };
const { name, ...rest } = car;
// name = "honda"
// rest = { year: "2023", model: "sports" }  <- brand new standalone object
```

**⚠️ Corrections made:**
1. Spread part was correct immediately (combining/copying, right side of `=`).
2. Rest part had syntax confusion — originally described it as `const {name, ...} = const vehicle` with leftover properties "assigned to object vehicle." Corrected: you destructure *from* an existing object (`car`); the leftover properties get collected into a **brand-new variable** you name yourself after `...` (commonly `rest`, but any name works) — nothing is being assigned into a pre-existing `vehicle`.
3. Follow-up misconception: thought `rest` was "nested in the full destructured object." Corrected: `name` and `rest` are two completely separate, independent, standalone variables produced side-by-side by the same line — nothing is nested inside anything.

---

## Q8: `reduce` — arguments and mechanism

**Q:** What does `reduce` do, and what are its two arguments?

**A:** `reduce`'s two top-level arguments are **(1) a callback function** and **(2) a starting value for the accumulator**:
```js
arr.reduce(callback, startingValue);
```
The accumulator itself is NOT a separate top-level argument — it lives **inside the callback** as the callback's own first parameter:
```js
const sum = [1, 2, 3].reduce((accumulator, item) => accumulator + item, 0);
```
The accumulator gets reassigned every iteration to whatever the callback returns, carrying that new value into the next iteration.

**⚠️ Correction made:** first answer described reduce's parameters as "accumulator and a callback function" as if the accumulator were a separate top-level argument — it's not; only its *starting value* is passed to `reduce` directly, the accumulator variable itself only exists inside the callback.

**Confirmed correct restatement (own words):** *"reduce's two top-level arguments are the callback and the starting value, callback itself takes (accumulator, item), and accumulator changes every iteration to callback's return — e.g. if callback is accumulator + item, then for each item it becomes previous accumulator + that item, which becomes the accumulator going into the next iteration."*

---

## Q9: `filter` from scratch

**Q:** Write `filter` from scratch (cold, no reference) — what does it need to do differently from `map`?

**A (final, correct):**
```js
function filter(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i])) {
      result.push(arr[i]);
    }
  }
  return result;
}
```

**⚠️ Bugs worked through:**
1. First attempt: `if (callback) { result.push(arr[i]); }` — this checks whether `callback` *itself* (the function reference) is truthy, which is always true since a function always exists. It never actually *calls* `callback` on the item, so every item would incorrectly pass.
2. Second attempt fixed the logic (`callback(arr[i])`) but had a missing closing parenthesis: `if(callback(arr[i]){...}`.
3. Third attempt was fully correct.

**Key distinction from `map`, confirmed:** `map` always pushes (the *transformed* value, unconditionally); `filter` conditionally pushes (the *original*, unchanged value, based on a true/false test).

---

## Q10: `some` vs `every` (+ building both from scratch)

**Q:** What's the difference between `some` and `every`?

**A:** `some`: returns `true` as soon as ANY item passes; `false` only if NONE pass. `every`: returns `true` only if ALL items pass; `false` as soon as ANY ONE fails.

```js
[1, 3, 5].some((n) => n % 2 === 0);   // false - no even numbers
[1, 3, 4].some((n) => n % 2 === 0);   // true - 4 passes

[2, 4, 6].every((n) => n % 2 === 0);  // true - all pass
[2, 4, 5].every((n) => n % 2 === 0);  // false - 5 fails
```

**Note:** said "totally forgot" these initially, but the logical description given up front ("some returns false if all are false else true, every needs every value true else false") was actually already correct, just loosely phrased.

**`some` from scratch (correct after one syntax fix — missing paren):**
```js
function some(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i])) {
      return true;
    }
  }
  return false;
}
```

**`every` from scratch (fully correct first try, just misnamed the function `some` instead of `every`):**
```js
function every(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (!callback(arr[i])) {
      return false;
    }
  }
  return true;
}
```
Logic: early-return `false` the instant any item fails (`!callback(...)` is true); if the loop completes without that ever triggering, every item passed → return `true`.

---

## Q11: Promises — definition and states

**Q:** What is a Promise, and what are its three possible states?

**A (correct on first try):** A Promise is an object representing a value that will be completed/available at some point, not instantly. Three states: **pending** (not yet settled), **fulfilled/resolved** (completed successfully), **rejected** (failed). Note: "fulfilled" and "resolved" are used interchangeably for the success state in interviews.

---

## Q12: Why `await` requires `async`

**Q:** Why does `await` only work inside an `async` function — what's the actual rule, and why does it exist?

**A:** `await` literally **pauses execution of the function it's inside**, at that exact line, until the Promise resolves — then resumes with the resolved value. That pausing/resuming mechanism only works inside a function specifically built to support it, which is what `async` sets up. Using `await` outside an `async` function throws a `SyntaxError` — not a style choice, it's enforced because the pause mechanism requires it.

**Contrast with `.then()`:** `.then()` doesn't pause anything — it schedules "whenever this resolves, run this callback" and the rest of the code continues immediately. That's why `.then()` works anywhere, no special function needed.

**Note:** this exact explanation exists already in earlier Day 9 notes ("Why await requires an async function - the actual rule," near word-for-word) — flagged as a retention gap (forgotten after a few weeks), not a teaching gap. Worth a quick re-read of that file before the interview since it didn't lock in the first time.

---

## Q13: Microtask queue vs macrotask queue

**Q:** What's the actual difference between the microtask queue and the macrotask queue in the event loop, and which one runs first?

**A:** Three separate places code can live: the **call stack** (synchronous code, one line at a time), the **macrotask queue** (`setTimeout`, click events — one at a time), and the **microtask queue** (Promise `.then()`/`.catch()`/`async-await` continuations). Key rule: **once the call stack is empty, JS fully drains the ENTIRE microtask queue before picking even one task from the macrotask queue.**

**Trace example, worked through correctly:**
```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
```
Correctly predicted the output order as `1, 4, 3, 2` cold, before explanation.

Why: `1` and `4` run synchronously on the call stack immediately. `setTimeout` schedules into the macrotask queue even at `0ms`. `.then()` schedules into the microtask queue. Once the stack is empty, the full microtask queue drains first (`3`), then the macrotask queue is checked (`2`).

**⚠️ Initial conceptual answer needed a full rebuild:** described microtask queue as "simple callstack" and macrotask queue as a "priority queue" — mixed up the call stack with the queues entirely. After the trace example, correctly restated: call stack runs synchronous code, microtask queue = Promise/.then/.catch/async-await callbacks, macrotask queue = setTimeout/click events, microtask queue fully drains before macrotask queue is touched. (One final wording slip: said "callback runs the synchronous task" — should be "call stack runs synchronous code," not "callback.")

---

## Q14: `export default` vs named export

**Q:** What's the difference between `export default` and a named `export` in ES modules?

**A (correct on first try):** `export default` lets the importing file choose ANY name for the import. A named export must be imported using that exact same name (unless renamed with `as`).

```js
// math.js
export default function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
```
```js
// app.js
import whateverNameIWant from "./math.js";   // default - any name works
import { subtract } from "./math.js";         // named - must match exactly
```

**Extra rule:** a file can only have **one** `export default`, but unlimited named exports.

**`as` renaming example:**
```js
import { subtract as minus } from "./math.js";
minus(5, 2); // 3
```
`as` renames the import only on the importing side — the original file still exports it under its original name.

---

## Q15: Custom error classes

**Q:** What's the difference between a regular `Error` and a custom error class (e.g. `class AuthError extends Error`), and why would you bother creating one?

**A:** `AuthError extends Error` inherits everything `Error` has (`.message`, `.stack`) while adding its own extra properties/behavior. The practical reason: custom error classes let you **distinguish between different kinds of errors using `instanceof`**, and attach extra context a generic `Error` doesn't carry.

```js
class AuthError extends Error {
  constructor(message) {
    super(message);        // passes message up to the base Error class
    this.name = "AuthError";
    this.statusCode = 401; // extra property generic Error doesn't have
  }
}

try {
  throw new AuthError("Invalid credentials");
} catch (error) {
  if (error instanceof AuthError) {
    console.log("Handle auth-specific logic:", error.statusCode);
  } else {
    console.log("Some other kind of error:", error.message);
  }
}
```

**Confirmed via follow-up doubt:** if you `throw new AuthError(...)`, `error instanceof AuthError` is `true` (and `error instanceof Error` is also `true`, since `AuthError` inherits from `Error`). If you'd thrown a plain `new Error(...)` instead, `error instanceof AuthError` would be `false`, but `error instanceof Error` would still be `true`. So `instanceof` lets you catch broadly (`Error`) or narrowly (your specific class) depending on need.

**Separate doubt resolved — `constructor`/`super` mechanics:** `constructor` is the function that runs when `new ClassName(...)` is called — it sets up the new object. `super(message)` is mandatory before using `this` in a subclass constructor; it delegates to the parent class's (`Error`'s) own setup code, which is what makes `.message` and `.stack` actually work correctly — without it, `this` isn't properly initialized and JS throws a `ReferenceError`. `this.name = "AuthError"` is a separate, manual step on top (not something `super()` handles) — it's what makes the error display as `"AuthError: ..."` instead of generic `"Error: ..."` when logged.

---

## Q16-Q21: DOM Fundamentals

**Q16: Difference between `querySelector` and `querySelectorAll`? Select the first `<li>` inside `<ul class="list">`.**
A: `querySelector` returns the first match (or `null`); `querySelectorAll` returns all matches as a `NodeList`. Scoped selector: `document.querySelector(".list li")` — the space means "descendant of."
**⚠️ Flagged:** `document.querySelector("li")` alone is NOT scoped — it searches the whole document, not just inside that `<ul>`.

**Q17: `.list` vs `#list` as selectors — what's the real distinction?**
A: `id` should be unique (one element per page); `class` is reusable (many elements can share it). This governs *how many elements exist* with that identifier — it's unrelated to whether you use `querySelector` vs `querySelectorAll` (separate, orthogonal concepts).
**⚠️ Flagged:** initial answer conflated "which selector to use" with "how many results come back," implying class = multiple results, id = single result — corrected: that distinction is controlled entirely by `querySelector` vs `querySelectorAll`, not by `.` vs `#`.

**Q18: Two `<ul class="list">` elements exist. What does `querySelector(".list li")` return vs `querySelectorAll(".list li")`?**
A (correct): `querySelector` → first `<li>` in document order (from the first matching `<ul>` only). `querySelectorAll` → every `<li>` from both `<ul>`s, combined into one `NodeList`.

**Q19: Difference between `NodeList` and a regular array? Can you call `.map()` on a NodeList?**
A (correct): `NodeList` is a collection of actual DOM elements; no `.map()` available directly (correctly guessed). Added detail: `NodeList` *does* have `.forEach()` built in, but not `.map()`/`.filter()`/`.reduce()`. Convert via `Array.from(nodeList)` or `[...nodeList]` for full array methods.

**Q20: `textContent` vs `innerHTML` — and the security risk?**
A (mostly correct): `textContent` = raw text only; `innerHTML` = text + markup, parsed and potentially executed as real HTML.
**⚠️ Flagged:** said "innerHTML shows hidden text too" — not accurate; corrected to "text only" vs "text + markup as a string," nothing about hidden text. Security risk is XSS: inserting untrusted input via `innerHTML` lets attacker-supplied `<script>`/`onerror` tags actually execute (e.g., stealing `document.cookie`). Fix: use `textContent` for any user-provided content.

**Q21: Event bubbling vs capturing — which does `addEventListener` use by default? What does `stopPropagation()` do?**
A: Capturing = top-down (document → target); bubbling = bottom-up (target → document), fires after target is reached. Default is bubbling. Pass `true` as third arg of `addEventListener` for capturing.
`stopPropagation()` stops the event from continuing to travel further — doesn't stop the browser's default action (that's `preventDefault()`, unrelated).
**Confirmed correctly, own words:** bubbling always happens regardless of listeners — "traveling through" an ancestor and "triggering a listener on" it are separate; no listener = event passes through silently, nothing happens, until it reaches the top. Event delegation is a *technique* built on the bubbling *mechanism*, not the same thing — delegation specifically relies on bubbling, not capturing.
**Real use case for capturing (from doubt):** a global analytics listener attached with `true` guarantees it fires before any child's `stopPropagation()` call (during bubbling) could block it, since capturing runs first, top-down, before the event reaches its target.

---

## Micro-Gaps (explain-first track, not quiz-style)

### `==` vs `===` and type coercion
`===` checks value AND type, no conversion. `==` allows coercion before comparing. Classic surprises: `null == undefined` → `true`; `0 == "0" == "" == false` → all compare as `0` after ToNumber conversion. `===` is the safe default.

### Debounce (built from scratch)
```js
function debounce(fn, delay) {
  let timerId;
  return function (...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => fn(...args), delay);
  };
}
```
Fires once, `delay` ms after the LAST call in a burst. Each call *schedules* a future execution and cancels the previous *not-yet-run* scheduled execution — `fn` is never actually invoked until a timer survives uninterrupted to completion. Use case: search-as-you-type, resize-end.

### Throttle (built from scratch)
```js
function throttle(fn, limit) {
  let inThrottle = false;
  return function (...args) {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => { inThrottle = false; }, limit);
    }
  };
}
```
Fires immediately, then "locks the door" for `limit` ms — any calls during the lock are thrown away entirely (not queued). Once unlocked, next call fires immediately and relocks. Result: fires at a steady max rate throughout continuous activity, never stalls to zero like debounce would. Use case: scroll-position tracking, continuous drag/mousemove handlers.

**Debounce vs throttle, one line each:** debounce = only the last call in a burst survives, after a quiet period (can mean *zero* fires during continuous activity). Throttle = calls get through at a steady max rate the whole time, dropping everything in between (never stalls to zero).

### Event delegation
One listener on a parent, relying on bubbling + `event.target` to identify which child was actually clicked. `event.target` is fixed at click time (the physical element clicked), regardless of where the listener is attached. `tagName` is always uppercase (`"LI"`, not `"li"`).

### Shallow vs deep copy
Spread (`{...obj}`) copies one level only — nested objects/arrays remain shared by reference. `structuredClone(obj)` performs a true deep copy, recursively, with no shared references at any level.

### Optional chaining (`?.`) and nullish coalescing (`??`)
`?.` stops a property/method/index access chain from throwing when the left side is `null`/`undefined`, short-circuiting to `undefined`. Only triggers on `null`/`undefined` specifically — `0`, `""`, `false` pass through untouched.
`??` provides a fallback value only when the left side is specifically `null`/`undefined` — unlike `||`, which incorrectly falls back on ANY falsy value. Common combined pattern: `user.profile?.address?.city ?? "Unknown"`.

### localStorage / sessionStorage vs Cookies
| | Lifespan | Sent to server automatically? | Readable by JS? | Use case |
|---|---|---|---|---|
| localStorage | Forever until cleared | No | Yes | Theme, non-sensitive cached UI state |
| sessionStorage | Until tab closes | No | Yes | Draft form data, one-time wizard state |
| Cookies | Explicit expiry you set | **Yes, every request** | Only if not `HttpOnly` | Auth tokens, session IDs |

Cookies attach automatically to every HTTP request (costing bandwidth on requests that don't need them) and are capped at ~4KB; `localStorage` never auto-sends and holds ~5-10MB. Rule: cookies for things the *server* needs to see (auth tokens — ties to Day 5 middleware's JWT cookie check); localStorage for purely client-side concerns. `HttpOnly` cookies are immune to theft via XSS (unlike a token in localStorage, directly readable by any injected script) — this is why auth tokens are `HttpOnly` cookies, not localStorage.

---

## Status: Step 1 (JS revision + micro-gaps) — COMPLETE

**Recurring patterns worth a final pre-interview double-check (flagged more than once across this session):**
- `var` (function-scoped) vs `let` (block-scoped) — repeated mix-up, said backwards twice across two different checkpoints
- TDZ direction (top-of-scope → declaration, not declaration → usage) — also repeated
- `this` in arrow functions — took several reframings to land (lexical scope, not "parent of object")
- Array method internals (`reduce`'s actual two arguments, `filter`/`some` syntax) — logic was generally sound, syntax slips were the recurring issue (missing parens)
- Event loop queues — initial mental model conflated call stack with the task queues; solid after the trace example

**Next up:** Step 2 — React revision + micro-gaps (including error boundaries).