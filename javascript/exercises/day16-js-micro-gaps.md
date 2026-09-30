# Week 5 (WHOOP Prep) - Session 1: JavaScript Revision + Micro-Gap Closing

## == vs === and Type Coercion

`===` (strict equality) checks both value AND type — no conversion. If types differ, immediately `false`.

`==` (loose equality) allows type coercion — converts one or both sides to a common type before comparing.

```js
5 === "5"   // false - types differ, no conversion
5 == "5"    // true - "5" converted to 5 first, then compared
```

**Classic surprises:**

```js
null == undefined   // true - special-cased loose equality rule
null === undefined  // false - different types

0 == "0"    // true - "0" -> Number("0") -> 0
0 == ""     // true - "" -> Number("") -> 0
0 == false  // true - false -> Number(false) -> 0
```

All three (`0`, `"0"`, `""`, `false`) end up compared as `0 == 0` under the hood, since `==` converts both sides using JS's ToNumber rules before comparing. This unpredictability is exactly why `===` is the production default.

## Debounce (built from scratch)

**The problem:** an event (keystroke, scroll) can fire far more often than you actually want to react to it - e.g. firing a search API call on every single keystroke instead of once the user stops typing.

**The fix:** delay execution until the event has stopped firing for a set period, cancelling and restarting the delay every time it fires again.

**Why the closure pattern (function returning a function) is required**, not just stylistic:
- `timerId` needs to persist across multiple separate calls to the debounced function - a variable declared *inside* the inner function would reset every call, useless.
- `timerId` also needs to be private/isolated per debounced function - a single global `timerId` would cause two unrelated debounced functions (e.g. one for search, one for scroll tracking) to accidentally cancel each other's timers.
- The closure solves both: each call to `debounce(fn, delay)` creates a brand new, private `timerId` that only that specific returned function can see and touch.

```js
function debounce(fn, delay) {
  let timerId;                        // private, shared across calls to the SAME debounced function

  return function () {
    clearTimeout(timerId);            // cancel whatever is still pending from the LAST call
    timerId = setTimeout(fn, delay);  // schedule a fresh timer, save its ID
  };
}
```

**How it actually works, traced:** every call cancels the previous pending timer (which is then gone forever, not paused) and starts a new one. Only the very last call's timer never gets cancelled by anything after it, so it's the only one that survives long enough to actually fire. This converts "fire after every call" into "fire once, 300ms after the LAST call."

**Note on `return` vs assignment (a real point of confusion worked through):** `debounce`'s own `return` statement returns the *inner function* - that's the one thing separate from `timerId`. `timerId` is not "returned" by anything; it's a plain variable that gets its value through assignment (`timerId = setTimeout(...)`), the same as any other variable reassignment. `setTimeout(...)` is the thing that returns a value; that value just gets stored into `timerId`.

## Event Delegation

**The problem:** attaching a separate `addEventListener` to every item in a list (e.g. 100 items) wastes memory and doesn't automatically cover items added later.

**The fix:** attach ONE listener to the shared parent, relying on **event bubbling** - when you click a child, the browser fires the event on that child first, then automatically propagates ("bubbles") it upward through every ancestor. A listener on the parent still gets notified.

**Identifying which child was actually clicked:** every event object has `.target` - the exact element the click physically originated on (determined at the moment of the click, before bubbling even starts), regardless of where the listener happens to be attached. Bubbling only changes where the notification travels to, not what was originally clicked.

```js
const list = document.querySelector("ul");

list.addEventListener("click", function (event) {
  if (event.target.tagName === "LI") {
    console.log("You clicked:", event.target.textContent);
  }
});
```

**Note:** `tagName` always returns uppercase (`"LI"`, `"BUTTON"`), regardless of how the tag is written in HTML - a common gotcha if you compare against a lowercase string.

**My own exercise - button delegation, written correctly on first try:**

```js
const div = document.querySelector("div");
div.addEventListener("click", (event) => {
  if (event.target.tagName === "BUTTON") {
    console.log(event.target.textContent);
  }
});
```

## Shallow vs Deep Copy

Spread (`{...obj}`) only copies **top-level** properties. For primitive values (strings, numbers), this means a real, independent copy. For nested objects/arrays, spread only copies the **reference** (pointer) to that same nested object - it does not duplicate the nested object itself.

```js
const original = { name: "Sumer", address: { city: "Boston" } };
const copy = { ...original };

copy.address.city = "Changed";
console.log(original.address.city); // "Changed" - NOT "Boston"!
```

Both `original.address` and `copy.address` point at the exact same object in memory - there's only one `address` object, shared by reference. Changing it through either variable changes the only copy that exists. This is exactly why it's called a "shallow" copy - only the first level is truly duplicated.

**To make a real, fully independent deep copy:**

```js
const deepCopy = structuredClone(original);

deepCopy.address.city = "Changed";
console.log(original.address.city); // "Boston" - untouched
```

`structuredClone` recursively copies every level of nesting, so no references are shared anywhere. An older technique, `JSON.parse(JSON.stringify(original))`, achieves the same result but has quirks (silently drops functions and `undefined` values) - `structuredClone` is the modern, safer choice.

## Doubts / Questions I had

**Q: Which function "returns" timerId - debounce or the inner function?**

A: Neither "returns" it in the return-statement sense. `debounce`'s `return` statement returns the *inner function itself* - that's the only thing being returned there. `timerId` is just a regular variable that gets its value through plain assignment (`timerId = setTimeout(...)`), not through a `return`. `return` means "this function call is done, here's the one value handed back to the caller" - it's tied to one specific function finishing. Assignment just means "store this value in this variable" - nothing is handed back to anyone.

**Q: In the counter example, why doesn't `counterA()` always log 1 - why does it become 2, then 3?**

A: `let count = 0` only runs ONCE - the single time `makeCounter()` itself is called. `counterA` is not `makeCounter` - it's the *inner function* that got returned. Calling `counterA()` only re-runs the inner function's code (`count++; return count`); it never goes back and re-runs `makeCounter`'s body, so `count = 0` never resets. Each call increments the same persistent `count` that's been kept alive since that one original `makeCounter()` run.

**Q: For debounce, why are we deliberately cancelling (killing) timers instead of letting them run after their 300ms?**

A: Without cancelling, every single call (e.g. every keystroke while typing "cat") would independently schedule its own 300ms timer, and ALL of them would eventually fire - resulting in multiple firings, which defeats the purpose. Cancelling the previous timer on every new call ensures only the LAST call's timer survives to actually fire - converting "fire after every call" into "fire once, 300ms after the most recent call," which is the actual definition of debounce.

**Q: If a timer is "set" for 300ms, doesn't it always run for the full 300ms regardless?**

A: No - `setTimeout(fn, 300)` means "IF nothing cancels this first, run `fn` after 300ms" - not a guarantee. It's like an alarm clock: setting an alarm for 5 minutes doesn't force it to ring; you can switch it off before then, and if you do, it simply never rings. A cancelled timer isn't paused or still counting somewhere - it's gone entirely. Only the timer that nothing ever cancels gets to actually finish counting and fire.

**Q: Why use the closure ("function returning a function") pattern for debounce instead of just a plain local or global variable?**

A: A local variable declared inside the inner function would reset to empty on every call, so `clearTimeout` would never actually have anything to cancel. A single global `timerId` would work for exactly one debounced function, but break the moment two different debounced functions exist in the same app (e.g. one for search, one for scroll tracking) - they'd share the same `timerId` and incorrectly cancel each other's unrelated timers. The closure gives each call to `debounce()` its own private, persistent `timerId`, solving both problems at once.

**Q: Since the click listener is on the `<ul>`, how does `event.target` know which specific `<li>` was clicked instead of just reporting the `<ul>`?**

A: The browser determines `target` based on the physical click location - the exact element under the mouse cursor at the moment of the click (a specific `<li>`, not the invisible `<ul>` container) - before bubbling even starts. Bubbling then carries that event (and its `target`, unchanged) upward to the parent's listener. The listener's location (`<ul>`) and the event's origin (`<li>`) are two separate facts - bubbling only changes where the notification travels to, not what was originally clicked.

**Q: Why is it `event.target.tagName === "LI"` and not `"li"`?**

A: `tagName` always returns the tag name in uppercase, regardless of how it's written in the actual HTML (`<li>` still reports as `"LI"`) - this is just a DOM quirk, not an error in the HTML. Comparing against a lowercase string would always fail since `===` is exact and case-sensitive.

## Key takeaway (in my own words)

`==` converts types before comparing (leading to surprising results like `0 == false`), while `===` never converts, which is why it's preferred. Debounce uses the exact same closure pattern as makeCounter - a private, persistent variable (`timerId`) that survives across calls - to repeatedly cancel a pending timer and reschedule it, so only the last call in a rapid sequence actually executes. Event delegation puts one listener on a parent and relies on event bubbling plus `event.target` to identify which specific child triggered the event, avoiding the need for a listener on every child. Spread only creates independent copies one level deep - nested objects/arrays are shared by reference unless a true deep copy (`structuredClone`) is used instead.