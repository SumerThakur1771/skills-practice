# React Gap-Closing Session (Post Week 4) - Rules of Hooks, React.memo, Virtual DOM, useFetch

Session to close 4 specific gaps identified via research into what's most commonly tested in React interviews, before moving to Week 5 (Next.js).

## Gap 1: Rules of Hooks

**The rule:** hooks (useState, useEffect, useRef, etc.) must ALWAYS be called at the top level of a component - never inside if statements, loops, or nested functions.

```jsx
// WRONG - hook inside a conditional
function MyComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    const [name, setName] = useState(""); // NEVER do this
  }
  return <div>...</div>;
}
```

**Why this rule exists - the actual mechanism:** React doesn't track hooks by name - it tracks them by the ORDER they're called in, on every render. Internally React keeps something like a numbered list ("1st hook call is this state, 2nd is that effect"). If a hook is sometimes called and sometimes skipped (inside an if), that order shifts between renders, and React connects the wrong stored value to the wrong hook call, causing broken, confusing bugs.

```
Render 1 (isLoggedIn = true): hook order is [useState(name)]
Render 2 (isLoggedIn = false): hook order is [] - completely different!
```

**The fix:** hooks always run, unconditionally, in the same order, every render. Conditional LOGIC happens around them, not wrapping the hook call itself.

```jsx
function MyComponent({ isLoggedIn }) {
  const [name, setName] = useState(""); // always called, same position every render
  if (isLoggedIn) {
    // conditional logic is fine here, just not the hook call
  }
  return <div>...</div>;
}
```

## Gap 2: React.memo and its relationship to useCallback

### Step 1: Default behavior, no React.memo at all

By default, when a parent re-renders, EVERY child component inside it re-renders too, automatically, regardless of whether the child's own props changed.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child />
    </div>
  );
}

function Child() {
  console.log("Child rendered");
  return <p>Hi</p>;
}
```
Every click re-renders Parent, and Child re-renders too every time, even with zero props and nothing that could need to change - wasteful if Child does anything expensive.

### Step 2: Add React.memo - now it correctly skips

```jsx
const Child = React.memo(function Child() {
  console.log("Child rendered");
  return <p>Hi</p>;
});
```
React.memo checks "did props change?" before re-rendering. With no props at all, it always concludes "nothing changed" and skips re-rendering Child. Works perfectly here, no useCallback needed.

### Step 3: Add a function prop - React.memo alone breaks again

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const handleButtonClick = () => {
    console.log("clicked");
  };
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child handleButtonClick={handleButtonClick} />
    </div>
  );
}

const Child = React.memo(function Child({ handleButtonClick }) {
  console.log("Child rendered");
  return <button onClick={handleButtonClick}>Click me</button>;
});
```
Even with React.memo applied, Child re-renders on EVERY click. Why: every time Parent's function body runs again, `const handleButtonClick = () => {...}` creates a BRAND NEW function object each time - a genuinely different value in memory, even though it behaves identically. React.memo correctly detects "yes, this prop changed" (it's technically a new function) and re-renders Child accordingly. React.memo isn't broken - it's accurately detecting a real difference in the function reference.

### Step 4: useCallback fixes this specific problem

```jsx
const handleButtonClick = useCallback(() => {
  console.log("clicked");
}, []);
```
Now handleButtonClick is the exact same function object every render (since `[]` means never recreate it). React.memo's check now correctly sees "same reference as before" and skips re-rendering Child.

### The comparison table

| | React.memo alone, no function props | React.memo + function prop, no useCallback | React.memo + function prop + useCallback |
|---|---|---|---|
| Does Child re-render on unrelated Parent changes? | No (skipped) | YES (function prop looks "new" every time) | No (skipped) |

**The one-sentence relationship:** React.memo only works correctly for function props if that function's identity stays stable - useCallback is what keeps a function's identity stable, so React.memo's check doesn't get "fooled" by a harmless new function object that behaves identically to the old one.

**Important naming lesson:** a custom prop name (e.g. what a component chooses to call an incoming function prop) is completely separate from a real, built-in DOM event like a `<button>`'s own `onClick`. Naming a custom prop the same as a real DOM event (e.g. also calling it `onClick`) can cause genuine confusion about which one is being referred to at any point - using a distinct name (like `handleButtonClick`) for custom props makes the distinction clear.

## Gap 3: Virtual DOM and Reconciliation

**What the Virtual DOM is:** React keeps a lightweight, in-memory COPY of what the UI should look like - a plain JavaScript object representation, not the real DOM elements sitting in the browser.

**Why this exists:** manually updating the real DOM constantly is relatively slow and can cause real performance problems (covered in the DOM fundamentals session). React's approach: when something changes (like setState), React doesn't immediately touch the real DOM - it first builds a NEW Virtual DOM snapshot representing what the UI should look like now.

**What reconciliation means:** React then COMPARES the new Virtual DOM snapshot against the previous one - this comparison process is called reconciliation. It figures out exactly what changed between the two (a "diff"), then updates the REAL DOM only in those specific spots, not rebuilding the entire page from scratch.

**Why this is faster:** comparing plain JS objects (Virtual DOM snapshots) is fast. Touching the real browser DOM is comparatively slow. By figuring out the minimal necessary changes first (fast, in-memory), React touches the slow real DOM as little as possible.

**Interview-ready one-liner:** "React keeps a Virtual DOM - an in-memory representation of the UI. When state changes, React builds a new Virtual DOM snapshot, compares it against the previous one (reconciliation), and applies only the minimal necessary changes to the real DOM, rather than re-rendering everything, which would be slow."

## Gap 4: Building useFetch (common live-coding custom hook)

**Step 1 - the three pieces of state:**
```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
}
```
data holds the fetched result (starts null), loading tracks fetch-in-progress (starts true), error holds any error message (starts null).

**Step 2 - the effect that does the actual fetching:**
```jsx
useEffect(() => {
  async function fetchData() {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch");
      }
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  fetchData();
}, [url]);
```
useEffect's own function CANNOT be async directly (a real rule - its function must return nothing or a cleanup function, never a Promise), so a separate async function is defined INSIDE the effect and called immediately. Uses the exact response.ok pattern from the WHOOP assessment prep work.

**Step 3 - return what the caller needs:**
```jsx
return { data, loading, error };
```

**Using it in a component:**
```jsx
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(`https://api.example.com/users/${userId}`);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  return <p>{data.name}</p>;
}
```

## Key takeaway (in my own words)

Hooks must always be called in the same order every render, since React tracks them positionally, not by name - conditional logic must wrap around hook calls, never the hook call itself. React.memo skips re-rendering a component when its props haven't changed, but function props are tricky because a newly-created function (even with identical behavior) always counts as "different" - useCallback keeps a function's reference stable so React.memo's check isn't fooled by this. The Virtual DOM is React's in-memory UI representation, and reconciliation is the diffing process that finds the minimal real DOM changes needed, avoiding slow, unnecessary direct DOM manipulation. useFetch combines useState (for data/loading/error) and useEffect (wrapping an inner async function, since the effect itself can't be async) into a reusable hook for fetching data.