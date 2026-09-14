# React Integration Practice - 3 Varied Builds (Post Week 4)

Purpose: build multi-hook fluency across varied problem shapes (not repeating one pattern), addressing a fluency/reps gap identified after the Day 7 checkpoint - all individual concepts were solid, but combining multiple hooks fluidly without heavy scaffolding needed more practice.

---

## Build 1: Stopwatch (useRef + useEffect + state-driven control)

**Task:** count up every second, Start/Stop buttons, pause without resetting, proper cleanup, interval ID stored in useRef.

**Key reasoning worked through independently:**

**Why a NEW piece of state (isRunning) is needed, not something pre-existing:** initially didn't think to introduce a new boolean specifically to control the effect. Key realization: `isRunning` isn't data being displayed - it's a CONTROL SIGNAL purely there to drive when an effect should run. This generalizes: any "start/stop," "open/closed," "active/inactive" type behavior likely needs a boolean introduced specifically to control an effect via the dependency array.

**Why count persists correctly across pause/resume without special logic:** reasoned through that since `count` lives in its own useState, separate from the interval mechanism, it just sits at its last value while paused (nothing is calling setCount during that time) - resuming naturally continues from wherever it is, no "remember where I left off" logic needed.

**Why handlers only flip isRunning, not contain the timer logic themselves:** handlers express INTENT ("user wants it running/stopped") by calling setIsRunning; useEffect (watching [isRunning]) is where the actual interval creation and cleanup lives, since useEffect's cleanup function guarantees proper teardown tied to the component's lifecycle - handlers calling setInterval directly would have no clean, guaranteed way to stop a previous interval before a new one starts.

**Mistakes made and fixed:**
1. Wrote `setInterval((count) => { setCount(count + 1); })` - incorrectly assumed setInterval's callback receives an argument. setInterval calls its callback with ZERO arguments, always. Confused this with setCount's functional-update pattern, which DOES receive the real current value - but that behavior belongs to setCount, not setInterval. Fixed to `setInterval(() => { setCount((prevCount) => prevCount + 1); })` - functional update applied to setCount, not setInterval's callback.
2. Referenced a variable `timer` in `clearInterval(timer)` that was never declared - the ref was named `intervalID`. Fixed to `clearInterval(intervalID.current)`.
3. Had a redundant `else` branch calling `clearInterval` when `isRunning` is false - realized this is unnecessary since the effect's own cleanup function (returned from the `if` branch) already runs automatically right before the effect re-runs, including the transition to `isRunning = false`.
4. Left a typo `prevCountcount` (merged variable names) instead of `prevCount`.

**Final correct code:**
```jsx
import { useEffect, useState, useRef } from "react";

function Stopwatch() {
  const [count, setCount] = useState(0);
  const [isRunning, setRunning] = useState(false);
  const intervalID = useRef(null);

  function handleStartClick() {
    setRunning(true);
  }

  function handleStopClick() {
    setRunning(false);
  }

  useEffect(() => {
    if (isRunning) {
      intervalID.current = setInterval(() => {
        setCount((prevCount) => prevCount + 1);
      }, 1000);

      return () => {
        clearInterval(intervalID.current);
      };
    }
  }, [isRunning]);

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleStartClick}>Start</button>
      <button onClick={handleStopClick}>Stop</button>
    </div>
  );
}
```

---

## Build 2: SignupForm (multi-field validation with touched-state UX)

**Task:** email/password fields, error messages shown only after a field is "touched" AND invalid, Submit disabled until both valid.

**Key reasoning worked through independently:**

**Why "touched" needs useState, not useRef:** correctly reasoned that since touched status directly controls whether an error message shows/hides on screen, it needs to trigger a re-render - the exact useState vs useRef distinction from the gap-closing session, applied correctly to a new scenario.

**Operator precedence bugs caught and fixed through tracing:**
1. `!password.length >= 8` - `!` binds tighter than `>=`, so this evaluates as `(!password.length) >= 8`, not the intended `!(password.length >= 8)`. Fixed using `password.length < 8` directly (clearer than negating a comparison).
2. Disabled condition initially mixed "is empty" checks (`!email`, `!password`) together with "is invalid" checks in a tangled `&&`/`||` combination that didn't produce correct results - traced with real values (`node`) to prove the bug, then simplified to just the two pure validation checks combined with `||`: `disabled={!email.includes("@") || password.length < 8}`.

**Mistakes made and fixed:**
1. Both inputs used `value={input}` - referenced a variable that didn't exist; fixed to `value={email}` and `value={password}` respectively.
2. Used `<button onSubmit={handleSubmit}>` - buttons don't have onSubmit (that's for `<form>` elements); fixed to `onClick={handleSubmit}`.
3. Component initially named `signUpForm` (lowercase) - React requires component names to start with a capital letter to be recognized as components; fixed to `SignupForm`.

**Final correct code:**
```jsx
function SignupForm() {
  const [email, setEmail] = useState("");
  const [emailTouched, setEmailTouched] = useState(false);
  const [password, setPassword] = useState("");
  const [passwordTouched, setPasswordTouched] = useState(false);

  function handleEmail(e) {
    setEmail(e.target.value);
    setEmailTouched(true);
  }

  function handlePassword(e) {
    setPassword(e.target.value);
    setPasswordTouched(true);
  }

  function handleSubmit() {
    console.log("Form submitted:", email, password);
  }

  return (
    <div>
      <input value={email} onChange={handleEmail}></input>
      {!email.includes("@") && emailTouched && <p>Email must contain @</p>}
      <input value={password} onChange={handlePassword}></input>
      {!(password.length >= 8) && passwordTouched && (
        <p>Password must contain atleast 8 characters</p>
      )}
      <button
        onClick={handleSubmit}
        disabled={!email.includes("@") || password.length < 8}
      >
        Submit
      </button>
    </div>
  );
}
```

---

## Build 3: UserProfile (async fetch with loading/error states)

**Task:** fetch a user via a provided `fetchUser(id)` function, show loading/error/data states appropriately, fetch on mount.

**Key clarification worked through:** `fetchUser(id)` is a DIFFERENT, higher-level function than the raw browser `fetch()` used in the WHOOP assessment - it resolves directly to the final data object, with no `.ok` to check and no `.json()` to call (unlike raw `fetch()`, which returns a response object that still needs unwrapping). Recognizing which "shape" of async function is being used (raw fetch vs. an already-processed data-returning function) determines what code is actually needed.

**Mistakes made and fixed:**
1. Initially placed the actual `fetchData()` call OUTSIDE the try/catch/finally block (only the function DEFINITION was inside try) - meant errors thrown when fetchData actually ran would never be caught. Fixed by moving the try/catch/finally to wrap the function's internal logic properly, with the call `fetchData();` happening after the function is fully defined (a plain function call, unaffected by hoisting/ordering the way const/let variables are).
2. Initially still tried to use `.ok` and `.json()` on the result of `fetchUser(id)`, treating it like raw `fetch()` - corrected once the distinction was clarified, to just `const result = await fetchUser(id); setData(result);`.
3. Component named `userProfile` (lowercase) - fixed to `UserProfile`.
4. Rendered `data.name`/`data.email` unconditionally, which would crash while `data` is still `null` before the fetch resolves - fixed with `{data && <p>{data.name}</p>}` guards.
5. Took `id` as a direct function parameter (`function UserProfile(id)`) instead of destructuring it from props (`function UserProfile({ id })`) - corrected to match how every other component receives data (via a single props object).

**Final correct code:**
```jsx
function UserProfile({ id }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchData() {
      try {
        const response = await fetchUser(id);
        setData(response);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    fetchData();
  }, [id]);

  return (
    <div>
      {loading && <p>Loading...</p>}
      {error && <p>Error: {error}</p>}
      {data && <p>{data.name}</p>}
      {data && <p>{data.email}</p>}
    </div>
  );
}
```

Usage: `<UserProfile id={1} />` - id passed as a prop.

---

## Overall progress noted across all three builds

Compared to the Day 7 checkpoint, these three builds needed noticeably less scaffolding - real bugs were still made (as expected, this is how the learning method is designed to work), but self-correction happened faster, and several genuinely subtle points were reasoned through independently before being confirmed correct: recognizing the need for a new control-signal state variable, distinguishing useState vs useRef based on whether re-rendering is needed, tracing operator precedence bugs with real values rather than guessing, and correctly identifying the props-vs-parameter pattern without being told directly. This reflects real progress on the specific fluency gap identified after Week 4's checkpoint - not new conceptual gaps, but increasing speed and independence in combining known concepts.