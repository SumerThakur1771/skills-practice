# Week 4 - Day 3: useEffect - Purpose, Dependency Array, Cleanup

## The problem useEffect solves

useState lets data be stored and updated, with React re-rendering whenever that data changes. But sometimes something needs to happen OUTSIDE of just rendering - fetching data from an API, setting up an event listener, starting a timer, logging something. These are called SIDE EFFECTS - anything that reaches outside the component to interact with the world (network request, subscription, manually touching the DOM, a timer). A component's main job (the JSX it returns) should just describe the UI - useEffect is where "other stuff that needs to happen" goes.

**Precise distinction:** useEffect itself is not "a side effect" - it's the hook/mechanism for running side effects. The function passed into it is where the actual side-effect logic lives.

## Basic syntax

```jsx
import { useState, useEffect } from "react";

function Greeting() {
  const [name, setName] = useState("Sumer");

  useEffect(() => {
    console.log("Component rendered, name is:", name);
  });

  return <h1>Hello, {name}</h1>;
}
```

By default (no second argument), this function runs AFTER EVERY SINGLE RENDER - every time this component re-renders for any reason, this code fires again.

## The dependency array - controlling WHEN the effect runs

Running an effect after every render is often wasteful or buggy (e.g. an effect fetching data would cause unnecessary network requests on every unrelated re-render). The dependency array controls this.

```jsx
useEffect(() => { ... });         // no array at all -> runs after EVERY render
useEffect(() => { ... }, []);     // empty array -> runs ONLY ONCE, after the very first render
useEffect(() => { ... }, [name]); // array with a value -> runs after first render, AND whenever `name` changes
```

Example: `useEffect(() => { console.log("effect ran"); }, [count]);` only watches `count`. Calling `setName("Alex")` does NOT trigger it (name isn't in the dependency array); calling `setCount(5)` DOES trigger it.

## Cleanup functions

**The problem cleanup solves:** some effects set something up that needs to be torn down later - a timer, an event listener, a subscription. Without cleanup: memory leaks, duplicate listeners, or effects still running on components that no longer exist on screen.

**Analogy:** turning on a fan when entering a room (the effect "setting something up") vs turning it off before leaving (the cleanup "tearing it down"). Without cleanup, the fan just keeps running even after leaving the room.

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log("tick");
  }, 1000);

  return () => {
    clearInterval(timer); // cleanup function
  };
}, []);
```

`setInterval(callback, 1000)` is plain JS - runs callback repeatedly every 1000ms until explicitly stopped. `setInterval` returns an ID (stored in `timer`); `clearInterval(timer)` uses that ID to stop that specific timer.

**The React-specific rule:** whatever function is RETURNED from inside the effect function is treated by React as the cleanup logic, and React calls it automatically at the right time. This is a React API design convention, not something to derive logically - just a rule: return a function from inside useEffect, and React treats that specifically as cleanup. Never called manually - just defined (by returning it), and React calls it at the correct moment.

## When exactly cleanup runs - the general rule

Cleanup runs in exactly two situations, always:
1. Right before the effect is about to run again (only relevant if the effect actually re-runs, which depends on the dependency array)
2. When the component unmounts (always happens exactly once, regardless of dependency array)

**With `[]` (empty array):**
- Effect runs ONCE at first render (mount). Never re-runs, since nothing in the array can ever change.
- Cleanup runs ONCE, only at unmount.
- These are two separate single events, not the same thing happening twice.

```
1. Component first renders -> effect runs once -> timer starts
2. (re-renders for unrelated reasons -> effect does NOT run again, because of [])
3. Component removed from screen -> cleanup runs once -> timer stops
```

**With no array at all (runs after every render):**
- Effect runs after EVERY render.
- Since it keeps re-running, cleanup must run before EACH new run, to stop the previous setup before starting a new one (otherwise multiple overlapping timers/listeners would stack up).

```
1. First render -> effect runs -> timer #1 starts
2. Re-render -> cleanup runs (stops timer #1) -> effect runs again -> timer #2 starts
3. Re-render again -> cleanup runs (stops timer #2) -> effect runs again -> timer #3 starts
... continues every render ...
Eventually unmounts -> cleanup runs one final time
```

**With `[name]` (a real dependency):**
- Effect runs at mount, and again only when `name` specifically changes.
- Cleanup runs right before each of those re-runs, plus once at unmount.

```
1. First render -> effect runs -> timer #1 starts (using current name)
2. Re-render, but name did NOT change -> effect does NOT run again -> cleanup does NOT run -> timer #1 keeps running untouched
3. name DOES change -> cleanup runs (stops timer #1) -> effect runs again -> timer #2 starts, using new name value
4. Eventually unmounts -> cleanup runs one final time
```

## Summary table

- `[]` -> effect re-runs: never -> cleanup only ever fires at unmount
- `[name]` -> effect re-runs: whenever name changes -> cleanup fires before each of those re-runs, plus at unmount
- no array -> effect re-runs: every render -> cleanup fires before every single one of those re-runs, plus at unmount

## Interview relevance

"What is useEffect used for, and how does it differ from writing code directly in the component body?" - useEffect is specifically for side effects (things outside rendering), run after React updates the actual DOM, not during the render itself. "Why does my effect run twice / not clean up properly?" is a very common real bug rooted in misunderstanding the dependency array and cleanup timing exactly as covered here.

## Key takeaway (in my own words)

useEffect lets side-effect logic (things outside of rendering, like timers, API calls, event listeners) run in connection with a component's lifecycle. The dependency array controls how often the effect re-runs - no array means every render, an empty array means only once at mount, and an array with values means it re-runs whenever those specific values change. A function returned from inside the effect is automatically treated by React as cleanup logic, which runs right before the effect's next run (if it re-runs at all) and always once when the component is removed from the screen.