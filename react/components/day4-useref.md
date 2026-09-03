# Week 4 - Day 4: useRef - Value Tracking and Direct DOM Access

## What useRef actually does - the basic mechanic

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

## Why would useRef ever be useful, if it doesn't update the screen?

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

**Example 3 - direct DOM element access** (see below) - needs a genuine reference to the actual element in the DOM, a fundamentally different job than tracking data.

## useRef's second major use case - accessing real DOM elements

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

Setting `ref={inputRef}` on a JSX element gives a DIRECT HANDLE to the real DOM node - the moment React renders that element, it automatically fills `inputRef.current` with the actual DOM element (same kind of object as `document.querySelector` would return from the DOM fundamentals session). This connection happens automatically because of the `ref={...}` attribute - not something done manually.

Once `inputRef.current` holds the real element, genuine built-in DOM methods can be called on it - like `.focus()`, which is plain browser behavior (not React-specific), same category as `.textContent` or `.style` from the DOM session.

**What .focus() actually does:** moves the user's keyboard focus/cursor to that element, exactly as if the user had clicked directly into it themselves. Common real use: auto-focusing a search bar on page load, or focusing the first invalid field after a failed form validation (accessibility).

## My practice exercises - built through real debugging

**Exercise 1 - ClickTracker (non-visual counter):**

Mistakes made and fixed along the way:
1. Missing declaration keyword: wrote `refCount = useRef(0);` instead of `const refCount = useRef(0);` - needs const/let/var before a new variable name.
2. Closing tag typo: wrote `<button><button/>` (two opening-style tags) instead of `<button>...</button>` (proper opening and closing pair).

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

Mistakes made and fixed along the way:
1. Casing mismatch: declared `const InputRef` (capital I) but referenced `inputRef` (lowercase i) elsewhere - JS is case-sensitive, names must match exactly everywhere.
2. JSX floating outside return: had `<input ref={inputRef}` as a standalone statement outside the return(...) block - JSX must be inside a return statement (or assigned to a variable) to actually render.
3. Mixed function syntax: wrote `function handleClick = () => {...}` - incorrectly combining a function declaration with arrow function assignment syntax. Must be either `function handleClick() {...}` OR `const handleClick = () => {...}`, not both together.
4. Missing `.current`: wrote `InputRef.focus()` instead of `InputRef.current.focus()` - useRef's actual value always lives behind `.current`, must go through it to reach the real DOM element.
5. Self-closing tag missing: wrote `<input ref={inputRef} >` instead of `<input ref={inputRef} />` - input has no children/separate closing tag, needs the `/` before `>` to be properly self-closed.
6. Multiple root elements: had `<input>` and `<button>` as two sibling elements directly in return, with no wrapper - a component can only return ONE single root element, needed to wrap both in a `<div>`.

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

## useRef vs useState - quick comparison

| | useState | useRef |
|---|---|---|
| Triggers re-render on change? | Yes | No |
| Value persists across renders? | Yes | Yes |
| Typical use | Data that affects what's shown on screen | Data to track internally, or direct DOM access, without affecting rendering |

## Key takeaway (in my own words)

useRef creates a box (`{ current: value }`) that persists across renders but never triggers a re-render when changed - the value genuinely updates internally, the screen just doesn't reflect it until some unrelated re-render happens to occur. Useful for tracking values that shouldn't cause re-renders (render counts, timer IDs) and for getting a direct reference to a real DOM element via `ref={...}`, enabling genuine DOM methods like `.focus()` to be called on it.