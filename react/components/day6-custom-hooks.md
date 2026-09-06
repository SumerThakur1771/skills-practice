# Week 4 - Day 6: Custom Hooks

## The problem custom hooks solve

Sometimes the same useState/useEffect logic needs to be reused across multiple different components, without copy-pasting it into each one.

**Motivating example - duplicated counter logic:**

```jsx
function CounterDisplay() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return <button onClick={increment}>{count}</button>;
}

function ClickCounter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return <p>Clicks: {count} <button onClick={increment}>Click</button></p>;
}
```

The exact same three lines (useState, the increment function, the logic inside it) are duplicated in both components. Any future change to how counting works would need to be made in every single place it was copy-pasted.

## What a custom hook actually is

A custom hook is just a REGULAR JAVASCRIPT FUNCTION that:
1. Its name starts with `use` (by convention, so React and other developers recognize it as a hook)
2. Internally uses other hooks (like useState), bundling that shared logic into one reusable place

## Extracting the shared logic into a custom hook

```jsx
function useCounter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return { count, increment };
}
```

Nearly identical to the duplicated logic - just wrapped in its own function, named useCounter, returning both `count` and `increment` so any component calling this hook can access them.

## Using the custom hook in multiple components

```jsx
function CounterDisplay() {
  const { count, increment } = useCounter();
  return <button onClick={increment}>{count}</button>;
}

function ClickCounter() {
  const { count, increment } = useCounter();
  return <p>Clicks: {count} <button onClick={increment}>Click</button></p>;
}
```

Each component's call to `useCounter()` gets its OWN INDEPENDENT `count` and `increment` - completely separate from each other (same idea as calling `makeCounter()` twice in the closures work - each call creates its own separate state, not a shared one).

`const { count, increment } = useCounter();` is object destructuring (Day 4 JS material), pulling both values directly out of the object useCounter returned.

**The actual payoff:** neither component writes useState or the increment logic itself anymore - it all lives in ONE place (useCounter), and both components just USE it. Any future bug fix or feature only needs to change inside useCounter, once, and both components automatically benefit.

## Clarifying JSX attribute vs content confusion

```jsx
<button onClick={increment}>{count}</button>
//      ^^^^^^^^^^^^^^^^^^^  ^^^^^^^
//      attribute: what          content: what's
//      happens on click         displayed as the button's text
```

Two separate, unrelated pieces sitting in the same tag:
- `onClick={increment}` - the ATTRIBUTE, defines what happens when clicked
- `{count}` - sitting BETWEEN the opening and closing tags, this is the button's visible text/content, unrelated to the attribute

```jsx
<p>Clicks: {count} <button onClick={increment}>Click</button></p>
```

Here the button's text is literally the word "Click" (not tied to count at all) - the count is shown separately, in the paragraph text before it. Two different UI designs, same underlying mechanism.

## My practice exercise - useToggle custom hook

Goal: custom hook tracking a boolean (starting false), returning the value and a toggle function; used in a LightSwitch component showing "On"/"Off".

**Mistakes made and fixed:**

1. Wrote `useState[false]` (square brackets) instead of `useState(false)` (parentheses) - useState is a function call.
2. Inside the toggle function, wrote `const value = toggle ? setToggle(false) : setToggle(true);` - created an unused variable, and referenced the current `toggle` directly to decide the next value, carrying the same stale-value risk covered in Day 2's useState deep dive.
3. First attempt at the functional update fix wrote `(t) => setToggle(!t);` as a standalone statement - this just creates and immediately throws away an arrow function, since it's never actually passed anywhere or called. Needed to be `setToggle((t) => !t);` - actually CALLING setToggle, with the arrow function passed in as its argument (same pattern as `setCount((prevCount) => prevCount + 1)` from Day 2).
4. Display logic: wrote `{toggle}?"On":"Off"` - the ternary's `? "On" : "Off"` was sitting OUTSIDE the curly braces, so JSX would try to literally render those characters as text. A ternary is a single JavaScript expression and needs the ENTIRE thing inside one set of curly braces: `{toggle ? "On" : "Off"}`.

**Final correct code:**

```jsx
function useToggle() {
  const [toggle, setToggle] = useState(false);

  function toggleTracker() {
    setToggle((t) => !t);
  }

  return { toggle, toggleTracker };
}

function LightSwitch() {
  const { toggle, toggleTracker } = useToggle();
  return (
    <p>
      <button onClick={toggleTracker}>Click</button>
      {toggle ? "On" : "Off"}
    </p>
  );
}
```

`useToggle` bundles boolean state and a safe toggle function using the functional update pattern (avoiding stale-value issues). `LightSwitch` destructures both, wires the button to flip the value, and uses a properly-wrapped ternary to display "On" or "Off".

## Key takeaway (in my own words)

A custom hook is just a regular function, named starting with `use`, that bundles reusable useState/useEffect logic into one place so multiple components can share it without duplicating code - each component calling the hook gets its own independent copy of that state, same as calling any function that creates closures. Attributes like onClick and the content between JSX tags are two separate, unrelated things that just happen to sit in the same tag. A full expression like a ternary must be entirely inside one set of curly braces in JSX, not split with part of it outside.