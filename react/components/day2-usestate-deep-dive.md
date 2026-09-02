# Week 4 - Day 2: useState In Depth - Stale State and Functional Updates

## Quick recap of basic useState

```jsx
const [username, setUsername] = useState("");
```

useState(startingValue) returns an array with two things: the current value, and a function to update it.

## Why you can't just mutate state directly

```jsx
const [count, setCount] = useState(0);

count = count + 1; // WRONG - does NOT trigger a re-render, React has no idea this happened
setCount(count + 1); // RIGHT - this is the function React gives specifically to trigger updates + re-renders
```

Directly reassigning a state variable does nothing meaningful - React only knows to re-render when you call the setter function it gave you.

## The mental model - state is a snapshot, frozen for the whole render

Every time a component re-renders, its function body runs again from scratch - meaning a state variable like `count` is technically a brand new variable each render, holding whatever value React currently has stored for it at that moment. This connects to closures (Day 5 JS material) - React keeps track of the current value behind the scenes and hands a fresh snapshot of it on every render.

Critically: `count` NEVER changes during a single function execution (like one click handler running), no matter how many times setCount is called within that same execution. It stays frozen at its snapshot value for the entire duration of that function call.

## The classic gotcha - calling setState multiple times in a row

```jsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
```

**If count starts at 0 and this runs once, the result is 1, NOT 3.**

**Why:** `count + 1` gets CALCULATED IMMEDIATELY, using the frozen snapshot value, before setCount ever sees it:

```jsx
setCount(count + 1); // count is 0 (frozen). Literally means: setCount(1). "next render, count should be 1"
setCount(count + 1); // count is STILL 0 (same execution, still frozen). Still means: setCount(1).
setCount(count + 1); // count is STILL 0. Still means: setCount(1).
```

All three calls evaluate `count + 1` using the same stale `0` - they're all just redundantly requesting "set count to 1." The last one overwrites the earlier ones (they're identical anyway). Final result: 1.

## The fix - functional updates (passing a function instead of a value)

```jsx
function handleClick() {
  setCount((prevCount) => prevCount + 1);
  setCount((prevCount) => prevCount + 1);
  setCount((prevCount) => prevCount + 1);
}
```

This correctly results in count becoming 3.

**Why this works - the key mechanism:** instead of calculating `count + 1` immediately (using the stale snapshot), this hands setCount a FUNCTION - a set of instructions for how to compute the next value, to be run LATER by React, using whatever the real, most up-to-date value is at that moment. Nothing is calculated right now; the calculation is delayed until React actually processes the update.

React internally queues state updates. When a function is passed, React guarantees it runs using the most recent value so far, including earlier updates from the same batch:

```
starts at 0
runs instruction 1: prevCount is 0 -> 0 + 1 = 1
runs instruction 2: prevCount is now 1 (from previous instruction) -> 1 + 1 = 2
runs instruction 3: prevCount is now 2 -> 2 + 1 = 3
final result: 3
```

## The comparison that made this click - like .map's callback

```jsx
arr.map((item) => item * 2);
//       ^^^^^^^^^^^^^^^^^ the callback - item is the parameter, .map calls this and feeds it the real current array item

setCount((prevCount) => prevCount + 1);
//        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the callback - prevCount is the parameter, setCount calls this and feeds it the real current state value
```

`setCount` is like `.map` itself - the thing that CALLS your function and feeds it a value. `prevCount` is like `item` - just a parameter name receiving whatever real, current value gets handed in when React actually runs it. `prevCount` is not a pre-existing variable, it's a name chosen to represent "whatever React passes in."

`setCount(count + 1)` (the broken version) skips this mechanism entirely - it hands setCount an already-computed number, calculated too early from the stale count, rather than a function for React to run later with the real value.

## Interview relevance

"Why doesn't calling setState multiple times in a row work the way you'd expect?" is a very commonly asked React question - it directly tests understanding that state updates aren't instant/synchronous within the same function call, and that closures capture a snapshot of state at render time.

## Doubts / Questions I had

**Q: Why did calling setCount(count+1) not save the incremented value in state so the next call would build on it?**

A: Because `count` is a frozen snapshot for the ENTIRE execution of handleClick - it doesn't get live-updated in between setCount calls within the same function run. Each `count + 1` calculation happens immediately, using that same frozen 0, so all three calls just redundantly compute and request the same value (1). setCount does not reach back and mutate the original count variable in place - it only schedules a future update for the next render.

## Key takeaway (in my own words)

State variables are frozen snapshots for the duration of a single render/function execution - calling the setter with a plain calculated value (count + 1) uses that stale snapshot every time, so multiple calls in the same execution collapse into one redundant update. Passing a function instead (prevCount => prevCount + 1) delays the calculation until React actually processes the update, at which point React feeds in the real, most current value - similar to how .map's callback receives the real current array item, not some value computed in advance.