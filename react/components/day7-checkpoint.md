# Week 4 - Day 7: React Checkpoint - SearchableList (Multiple Hooks Combined)

## The task

Build a component called SearchableList that:
1. Has an array of items
2. Has a text input tracked with useState
3. Displays only items containing the search term (case-insensitive), filtered live
4. Uses useRef to auto-focus the search input on mount (via useEffect with an empty dependency array)
5. Has a custom hook called useFilteredList(items, searchTerm) that extracts the filtering logic

This combined useState, controlled inputs, array filtering, useRef, useEffect, and custom hooks - everything from Week 4 in one integrated build. Needed real guided debugging to complete - a genuine synthesis challenge after only 5 days of React, expected at this stage.

## Building it piece by piece

### Piece 1 - basic structure: items array + controlled search input

**Mistakes made and fixed:**
1. Called `useState()` with no starting value - needed `useState("")` for an empty string default.
2. Left `onChange={}` empty - needed a handler function to actually update state as the user types.
3. Missing `value={input}` on the input entirely.

**The key concept clarified - why value={input} is needed (controlled vs uncontrolled inputs):**

Without `value={input}`, the input becomes UNCONTROLLED - the browser manages the displayed text on its own. onChange still fires and updates state correctly, but state and the visible box can DRIFT APART, since nothing forces the box to reflect state changes that come from anywhere other than typing.

**Concrete traced scenario proving this:** imagine a "Clear" button that calls `setInput("")`.
- Without `value={input}`: typing "apple" shows "apple" in the box (browser manages it). Clicking Clear sets state to "" but the box STILL VISUALLY SHOWS "apple", because nothing tells the actual DOM input to update - only typing (onChange) touches the box's display in this uncontrolled setup.
- With `value={input}`: when setInput("") runs, React re-renders and FORCES the box to display whatever `input` currently is - it correctly clears to empty, because React now owns what's displayed.

**Final correct code for this piece:**
```jsx
function trackInput(e) {
  setInput(e.target.value);
}

<input onChange={trackInput} value={input} />
```

### Piece 2 - filtering logic

**Key new concept: .includes() for substring matching (not exact match).**

```js
"Banana".includes("an"); // true - "an" is found somewhere inside "Banana"
"Banana" === "an";        // false - not equal, "an" isn't the WHOLE string
```

Combined with `.toLowerCase()` on both sides for case-insensitive matching:
```jsx
const filteredItems = items.filter((item) => {
  return item.toLowerCase().includes(input.toLowerCase());
});
```

Clarified that "contains" search (typing "an" shows "Banana") is the correct, intended, realistic search-box behavior - not a misunderstanding of the requirement, which specifically said "contain," not "exact match."

**Ordering bug caught and fixed:** initially declared `filteredItems` (which references `input`) BEFORE `const [input, setInput] = useState("")` - using a variable before its declaration (TDZ, Day 1 concept). Fixed by moving the useState declaration above the filtering logic that depends on it.

### Piece 3 - rendering the filtered list with .map() returning JSX

**Key concept clarified: React has a special rule for arrays of JSX inside curly braces.**

```jsx
<ul>
  {filteredItems.map((item) => (
    <li key={item}>{item}</li>
  ))}
</ul>
```

Initially confused about how an ARRAY (what .map() returns) ends up displaying as separate, individual list items rather than as a raw array. Clarified: this is a DELIBERATE, SPECIFIC React behavior, not something JavaScript does naturally - React specifically renders each JSX element in an array individually, in order, when it encounters an array of JSX inside `{ }`. Confirmed .map() still returns a genuine new array as always (Day 3 concept unchanged) - the array just happens to contain JSX elements, which React knows how to render item-by-item.

**Also avoided document.createElement/appendChild** (the vanilla JS DOM approach from the DOM fundamentals session) in favor of the React way (.map() returning JSX) - clarified these do NOT mix; React has its own declarative rendering system that should be used instead of manual DOM manipulation inside a component.

### Piece 4 - auto-focus with useRef + useEffect

**Key reasoning exercise: why `[]` and not `[input]` for the dependency array.**

Initial instinct was `[input]`, reasoning "when the user types a new search, it needs focus too." Self-corrected upon realizing this would re-run the effect (and re-call .focus()) on EVERY keystroke - redundant, since the input never actually loses focus while the user is actively typing into it.

**Clarified further:** focus is a genuinely separate browser concept from "what text is in the box." Once an input has focus, it STAYS focused through any number of keystrokes, deletions, and retypes, entirely on its own - no JavaScript needed to maintain it. The effect's only job is to START focus once, on mount, since nothing is focused by default when the page first loads. `[]` is correct specifically because "focus on load" is a one-time action with no changing value that should ever cause it to repeat.

**Final correct code:**
```jsx
const inputRef = useRef(null);

useEffect(() => {
  inputRef.current.focus();
}, []);

<input onChange={trackInput} value={input} ref={inputRef} />
```

### Piece 5 - extracting into a custom hook: useFilteredList

**Mistake made and fixed:** called `useFilteredList()` with no arguments - the hook expects `(items, searchTerm)` as parameters. Fixed to `useFilteredList(items, input)`, correctly passing both required arguments.

**Final correct code:**
```jsx
function useFilteredList(items, searchTerm) {
  const filteredItems = items.filter((item) => {
    return item.toLowerCase().includes(searchTerm.toLowerCase());
  });
  return { filteredItems };
}
```

## Full final correct solution

```jsx
function useFilteredList(items, searchTerm) {
  const filteredItems = items.filter((item) => {
    return item.toLowerCase().includes(searchTerm.toLowerCase());
  });
  return { filteredItems };
}

function SearchableList() {
  const items = ["Apple", "Banana", "Cherry", "Date", "Elderberry"];
  const [input, setInput] = useState("");
  const inputRef = useRef(null);
  const { filteredItems } = useFilteredList(items, input);

  function trackInput(e) {
    setInput(e.target.value);
  }

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return (
    <div>
      <input onChange={trackInput} value={input} ref={inputRef}></input>
      <ul>
        {filteredItems.map((item) => {
          return <li key={item}>{item}</li>;
        })}
      </ul>
    </div>
  );
}
```

## Additional clarification worked through - function declaration hoisting order

Question arose about whether `trackInput` needed to be declared before `useFilteredList(items, input)` is called. Clarified: `trackInput` is a FUNCTION DECLARATION, which is fully hoisted (Day 1 concept) - its position in the file doesn't matter, since the whole function is available throughout the component regardless of where it's written. This is different from `const`/`useState` variables (like `input`, `filteredItems`), which genuinely must be declared before use due to the TDZ. The rule: check whether something is a hoisted function declaration (position-independent) or a const/let variable (position matters, must exist before use).

## Result

All 5 requirements built successfully after guided debugging on each piece. Real conceptual gaps surfaced and resolved: controlled vs uncontrolled inputs (traced through a concrete Clear-button bug scenario), arrays-of-JSX rendering as a deliberate React-specific behavior, dependency array reasoning for one-time mount effects, and function hoisting order. Every individual concept was understood correctly once isolated and explained - the challenge was specifically in combining multiple hooks fluidly without scaffolding, which is a fluency/reps gap rather than a knowledge gap, expected after only 5 days of React practice.

## Key takeaway (in my own words)

Combining multiple hooks into one integrated component is harder than using any single hook in isolation - genuine synthesis work, not just recall. Controlled inputs need `value={}` specifically so React (not the browser) owns what's displayed, preventing drift between state and the visible UI. Arrays of JSX rendered inside `{ }` is a deliberate React rendering rule, not generic JavaScript array behavior. Dependency arrays should be reasoned about by asking "what changing value should cause this effect to run again" - for a one-time mount action like auto-focus, the honest answer is "nothing," which is exactly when `[]` is correct.