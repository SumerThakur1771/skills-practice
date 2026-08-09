# Week 2 - Day 14: Final JavaScript Checkpoint (Cold, No Notes)

## Result: All 3 functions written correctly by the end, after real debugging on each. JavaScript (Weeks 1-2): COMPLETE.

---

## Question 1: Write myFilter from scratch

**Goal:** function that takes an array and callback, returns a new array containing only items where the callback returns true.

**Mistakes made along the way, in order:**

1. Forgot to name the function - wrote `function(array, callback){...}` instead of `function myFilter(array, callback){...}`.
2. Used `int i` instead of `let i` in the for loop - old habit from Java/C++, JS doesn't use type declarations.
3. Wrote `callback(arr.push[i])` - a confused mix of three separate ideas jammed into one broken line:
   - Tried to use `.push` to READ an item, but `.push` is for ADDING to an array, not reading from it. Reading an item is just `arr[i]`.
   - Used `[i]` after `.push` like an index, which doesn't apply to a function/method reference.
   - Passed the (broken) result directly into `callback`, without separating "get the item" from "test the item" from "conditionally keep the item."
4. After fixing the read (`arr[i]`), initially wrote `result.push(callback(arr[i]))` - this pushes the CALLBACK'S RETURN VALUE, which is what myMap does, not myFilter. Filter needs to push the ORIGINAL ITEM, not the callback's result.
5. After fixing that to `result.push(arr[i])`, forgot the `if` check entirely - this pushed EVERY item unconditionally, ignoring the callback altogether. Needed an `if (callback(arr[i]))` wrapping the push.
6. Missing closing parenthesis: wrote `if(callback(arr[i]) result.push(arr[i]);` - needed `if(callback(arr[i])) result.push(arr[i]);`.

**Final correct code:**

```js
function myFilter(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i])) result.push(arr[i]);
  }
  return result;
}
```

**Key distinction learned:** map pushes the callback's RETURN VALUE. Filter pushes the ORIGINAL ITEM, but only conditionally, when the callback returns true. Two genuinely different jobs that look similar in skeleton (loop + push) but differ in exactly what gets pushed and when.

---

## Question 2: Write an async function getUserData with try/catch

**Goal:** async function taking userId, calling fetchUser(userId) (returns a Promise), using try/catch to log the result or an error.

**Mistakes made along the way, in order:**

1. First attempt: put `try/catch` OUTSIDE the async function, wrapping the plain call `getUserData(10)` instead of putting it INSIDE the function around the `await` line. This doesn't work because getUserData is async and returns a Promise - a rejection happening inside an async function's execution can't be caught by a try/catch sitting outside around the plain function call.
2. Second attempt: tried to chain `.try{ }` directly onto the function call like `getUserData(10).try{...}` - not real syntax. `try` is a keyword that starts its own block (`try { } catch { }`), it cannot be chained with a dot like `.then()`.
3. Also incorrectly used `resolve(result)` inside that broken block - `resolve` only exists inside the executor function when manually creating `new Promise((resolve, reject) => {...})`. getUserData doesn't manually create a Promise - it's an async function, which automatically returns a Promise on its own. There's no resolve to call here at all.
4. Fixed by going back to the correct shape from Day 9's runAgeCheck example: try/catch goes INSIDE the async function, wrapping the await line directly.

**Final correct code:**

```js
async function getUserData(userId) {
  try {
    const result = await fetchUser(userId);
    return result;
  } catch (error) {
    console.log(error.message);
  }
}

getUserData(10);
```

**Key distinction learned:** try/catch must be inside the async function itself, wrapping the await line - not outside around the call to the async function. resolve/reject only apply when manually building a `new Promise(...)`, not when using async/await, which handles the Promise creation automatically.

---

## Question 3: Write a custom AuthError class and login function

**Goal:** AuthError class extending Error, and login(password) that throws AuthError("Invalid password") if password !== "secret123", otherwise returns "Logged in".

**Mistakes made along the way, in order:**

1. Wrote `extends error` (lowercase) instead of `extends Error` (capital E) - JS is case-sensitive, the built-in class must match exactly.
2. Same mistake as Day 12: wrote `constructor(message);` with a semicolon and no `{ }` body, with `super(message)` and `this.name = ...` sitting outside the constructor instead of inside its curly braces.

**Final correct code:**

```js
class AuthError extends Error {
  constructor(message) {
    super(message);
    this.name = "AuthError";
  }
}

function login(password) {
  if (password != "secret123") throw new AuthError("Invalid password");
  return "Logged in";
}

login("hello"); // throws AuthError("Invalid password") since "hello" !== "secret123"
```

**Key distinction learned:** recognized and fixed the constructor body placement mistake faster this time than on Day 12 - same category of error caught more quickly, which is the actual goal of the "write wrong first, then correct" method.

---

## Overall Key Takeaway for Weeks 1-2 JavaScript

The recurring mistake categories across the whole two weeks were: leftover Java/C++ syntax habits (int instead of let), constructor body placement in custom classes (missing curly braces around super() and this.name), and confusing which value gets pushed/returned in array method implementations (callback's return value vs the original item). Recognizing these same categories faster in later exercises (Day 14 catching the constructor issue quicker than Day 12) is a real sign of the fundamentals sticking, not just pattern-matching to a memorized answer.