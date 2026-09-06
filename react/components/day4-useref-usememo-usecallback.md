# Week 4 - Day 4: useRef, useMemo, useCallback

## Part 1: useRef

### What useRef actually does - the basic mechanic

useRef gives a "box" that holds a value, and that box SURVIVES across re-renders (same as useState in that one way). But unlike useState, changing what's in the box does NOT make React re-render anything.

```jsx
const myRef = useRef(0);
```

This returns an object shaped like `{ current: 0 }`. Reading or changing the value always goes through `.current`:

```jsx
console.log(myRef.current); // 0
myRef.current = 5; // genuinely changed - but nothing on screen updates because of this
console.log(myRef.current); // 5
```

**Critical distinction confirmed through testing:** the value DOES genuinely update - `myRef.current` really becomes the new value, no trickery. It's just that changing it never notifies React to re-render. If that value happens to be displayed in JSX, the screen won't reflect the new value until SOMETHING ELSE causes a re-render for an unrelated reason - at which point it will correctly show whatever `.current` truly holds by then.

**Traced example:** clicking a button 3 times that does `countRef.current = countRef.current + 1` each time - the screen (`<p>{countRef.current}</p>`) stays frozen at 0 (no re-render ever triggered), but `countRef.current` genuinely IS 3 internally. If some unrelated re-render occurs afterward (e.g. from a different useState changing), the JSX re-evaluates and now correctly shows 3, since it just reads whatever the ref's real current value is at that moment.

**Deeper test - putting the ref directly in JSX:**
```jsx
const myRef = useRef(0);

function handleClick() {
  myRef.current = myRef.current + 1;
  console.log(myRef.current); // WILL show the updated number correctly every time
}

return <button onClick={handleClick}>{myRef.current}</button>; // will NOT visually update on click
```
The console.log always prints the correct, updated number, because that reads the real value directly when handleClick runs. But the displayed button text was locked in at whatever myRef.current was during the LAST ACTUAL RENDER - since refs never trigger renders, that locked-in value never gets a chance to update. Putting a ref directly in JSX does NOT make it "live" or reactive - JSX only reflects what it showed at the moment of the last render.

**How to actually make this reactive - two options:**

Option 1 (correct fix): if a value needs to be displayed and updated on screen, that's the textbook case for useState, not useRef - useRef was the wrong tool.
```jsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1);
}

return <button onClick={handleClick}>{count}</button>;
```

Option 2 (exists, but wrong tool for this case): keep the ref but separately force a re-render using an unrelated state value purely as a trigger.
```jsx
const myRef = useRef(0);
const [, forceRerender] = useState(0);

function handleClick() {
  myRef.current = myRef.current + 1;
  forceRerender((prev) => prev + 1); // this triggers the re-render
}
```
Option 1 is the right answer when a value needs to be shown and updated on screen - recognizing "this should just be useState" is a stronger answer than patching useRef to fake reactivity.

### Why would useRef ever be useful, if it doesn't update the screen?

The rule: ask "does the screen need to visually change because of this value?" If yes -> useState. If no, but something still needs to be remembered across renders -> useRef.

**Example 1 - counting renders for debugging, without needing it on screen:**
```jsx
const renderCount = useRef(0);
renderCount.current = renderCount.current + 1;
console.log("This component has rendered:", renderCount.current, "times");
```
Using useState here would cause INFINITE re-renders (updating state triggers a re-render, which increments again, which triggers another re-render...). useRef avoids this entirely since updating it never causes a re-render.

**Example 2 - storing a timer ID to cancel later (connects to Day 3's useEffect cleanup):**
```jsx
const timerRef = useRef(null);

function startTimer() {
  timerRef.current = setInterval(() => console.log("tick"), 1000);
}
function stopTimer() {
  clearInterval(timerRef.current);
}
```
The timer ID needs to be remembered between renders to stop it later, but has nothing to do with what's displayed on screen.

**Example 3 - direct DOM element access** (see below).

### useRef's second major use case - accessing real DOM elements

```jsx
function TextInput() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus();
  }

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus the input</button>
    </div>
  );
}
```

Setting `ref={inputRef}` on a JSX element gives a DIRECT HANDLE to the real DOM node - the moment React renders that element, it automatically fills `inputRef.current` with the actual DOM element (same kind of object as `document.querySelector` would return, from the DOM fundamentals session). This connection happens automatically because of the `ref={...}` attribute.

Once `inputRef.current` holds the real element, genuine built-in DOM methods can be called on it - like `.focus()`, plain browser behavior (not React-specific), same category as `.textContent` or `.style` from the DOM session.

**What .focus() actually does:** moves the user's keyboard focus/cursor to that element, exactly as if the user had clicked directly into it themselves. Common real use: auto-focusing a search bar on page load, or focusing the first invalid field after a failed form validation (accessibility).

### My practice exercises - built through real debugging

**Exercise 1 - ClickTracker (non-visual counter):**

Mistakes made and fixed:
1. Missing declaration keyword: wrote `refCount = useRef(0);` instead of `const refCount = useRef(0);`.
2. Closing tag typo: wrote `<button><button/>` instead of `<button>...</button>`.

**Final correct code:**
```jsx
function ClickTracker() {
  const refCount = useRef(0);

  function handleClick() {
    refCount.current = refCount.current + 1;
    console.log(refCount.current);
  }

  return (
    <button onClick={handleClick}>Click me</button>
  );
}
```

**Exercise 2 - FocusInput (DOM access):**

Mistakes made and fixed:
1. Casing mismatch: declared `const InputRef` (capital I) but referenced `inputRef` (lowercase i) elsewhere - names must match exactly everywhere.
2. JSX floating outside return: had `<input ref={inputRef}` as a standalone statement outside the return(...) block.
3. Mixed function syntax: wrote `function handleClick = () => {...}` - incorrectly combining a function declaration with arrow function assignment syntax.
4. Missing `.current`: wrote `InputRef.focus()` instead of `InputRef.current.focus()`.
5. Self-closing tag missing: wrote `<input ref={inputRef} >` instead of `<input ref={inputRef} />`.
6. Multiple root elements: had `<input>` and `<button>` as sibling elements with no wrapper - a component can only return ONE root element, needed a `<div>` wrapper.

**Final correct code:**
```jsx
function FocusInput() {
  const InputRef = useRef(null);

  function handleClick() {
    InputRef.current.focus();
  }

  return (
    <div>
      <input ref={InputRef} />
      <button onClick={handleClick}>click me</button>
    </div>
  );
}
```

### useRef vs useState - quick comparison

| | useState | useRef |
|---|---|---|
| Triggers re-render on change? | Yes | No |
| Value persists across renders? | Yes | Yes |
| Typical use | Data that affects what's shown on screen | Data to track internally, or direct DOM access, without affecting rendering |

---

## Part 2: useMemo

### The problem useMemo solves

Every time a component re-renders, EVERYTHING inside its function body runs again from scratch - including expensive calculations, even if the inputs to that calculation haven't actually changed.

```jsx
function ProductList({ products }) {
  const expensiveTotal = products.reduce((sum, p) => sum + p.price, 0);
  // this reduce() runs on EVERY render, even if products never changed
  return <p>Total: {expensiveTotal}</p>;
}
```

### What useMemo does

```jsx
import { useMemo } from "react";

function ProductList({ products }) {
  const expensiveTotal = useMemo(() => {
    return products.reduce((sum, p) => sum + p.price, 0);
  }, [products]);

  return <p>Total: {expensiveTotal}</p>;
}
```

useMemo takes a function and a dependency array - same shape as useEffect. It runs the function ONCE, caches ("memoizes") the result, and only re-runs the calculation if something in the dependency array actually changes. If products hasn't changed since last render, React hands back the CACHED result instantly, skipping the expensive recalculation.

### Key difference from useEffect

useEffect's function runs AFTER rendering, for side effects, and doesn't return a value used in rendering. useMemo's function runs DURING rendering, and its RETURN VALUE is directly used in JSX/logic.

**Verified with a trace:**
```jsx
const result = useMemo(() => {
  console.log("calculating...");
  return a + b;
}, [a, b]);
```
If the component re-renders due to unrelated state changing (not a or b), "calculating..." does NOT print again - React just returns the cached value without re-running the function.

### My practice exercise

```jsx
function ScoreBoard({ scores }) {
  const total = useMemo(() => {
    return scores.reduce((sum, s) => sum + s, 0);
  }, [scores]);

  return <p>Total: {total}</p>;
}
```
Correct on first attempt.

---

## Part 3: useCallback

### The problem useCallback solves

Functions are values in JS - every time a component re-renders, any function defined inside it is technically a BRAND NEW function object, even if it does the exact same thing as before.

```jsx
function Parent() {
  const handleClick = () => {
    console.log("clicked");
  };
  return <Child onClick={handleClick} />;
}
```

Every render of Parent recreates handleClick as a new function object. This can cause a real performance problem if Child is optimized to skip re-rendering when its props haven't "changed" (e.g. via React.memo) - React sees handleClick as a "different" prop every time, since it's a new function reference.

### What useCallback does

```jsx
import { useCallback } from "react";

function Parent() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);

  return <Child onClick={handleClick} />;
}
```

useCallback caches the FUNCTION ITSELF (not a calculated value like useMemo) - handleClick stays the exact same function reference across re-renders, as long as the dependency array items haven't changed.

### The core distinction between useMemo and useCallback

useMemo caches a VALUE (the result of running a function). useCallback caches the FUNCTION ITSELF. `useCallback(fn, deps)` is really shorthand for `useMemo(() => fn, deps)` - caching a function is just a specific case of caching a value, where the "value" happens to be a function.

**When to use which (correctly reasoned through independently):**
- A function that calculates and returns a number -> useMemo
- A function meant to be passed down to a child component as an event handler -> useCallback

---

## Key takeaway (in my own words)

useRef creates a box (`{ current: value }`) that persists across renders but never triggers a re-render when changed - useful for tracking values that shouldn't cause re-renders (render counts, timer IDs) and for getting a direct reference to a real DOM element to call genuine DOM methods on it. useMemo caches the RESULT of an expensive calculation, only recomputing when its dependencies change. useCallback caches a FUNCTION REFERENCE itself, keeping it stable across re-renders - useful when passing callbacks down to child components. All three help avoid unnecessary work or unwanted re-renders, but solve genuinely different problems: remembering non-visual data (useRef), caching computed values (useMemo), and caching function identities (useCallback).