# Week 2 - Day 12: Error Handling Patterns and Custom Errors

## finally

A third, optional piece alongside try/catch that runs EVERY SINGLE TIME, regardless of whether the try block succeeded or the catch block ran due to an error.

```js
try {
  console.log("trying something");
} catch (error) {
  console.log("something broke");
} finally {
  console.log("this runs no matter what");
}
```

**Core use case:** any cleanup work that must happen regardless of success or failure. The unifying shape across all use cases: "start some state or resource -> do risky work -> no matter what happens, undo/reset/release that state or resource."

Real examples:
1. Resetting a loading/UI state (hide a spinner either way)
2. Releasing a resource - database connection, file handle, lock (must close it either way or it leaks)
3. Resetting a "processing" flag so future calls aren't blocked

```js
async function fetchData() {
  showSpinner();
  try {
    const data = await getData();
    console.log(data);
  } catch (error) {
    console.log("Failed to fetch");
  } finally {
    hideSpinner(); // hide it either way
  }
}
```

## Throwing your own errors

Not just catching errors JS creates naturally - you can create and throw your own errors intentionally when your own code detects something is wrong.

```js
function withdraw(balance, amount) {
  if (amount > balance) {
    throw new Error("Insufficient funds");
  }
  return balance - amount;
}

try {
  withdraw(100, 500);
} catch (error) {
  console.log(error.message); // "Insufficient funds"
}
```

**Breaking down throw new Error(...):**
- `new Error("message")` creates an actual Error OBJECT (not just a string) - automatically comes with `.message` and `.stack` (trace of where the error happened).
- `throw` stops normal execution immediately and hands the error object off to be caught somewhere.

**Why throw instead of return an error message string:** returning a string gives the caller no forced way to know something went wrong - it's on them to remember to check the return value, which can silently hide bugs. Throwing interrupts execution and demands to be handled via try/catch - if not wrapped in try/catch, the program crashes loudly and visibly instead of quietly continuing with a bad value nobody checked for.

## Custom error types

For distinguishing WHAT KIND of error happened, not just "an error happened," so different failures can be handled differently.

```js
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

function checkAge(age) {
  if (age < 0) {
    throw new ValidationError("Age cannot be negative");
  }
  return age;
}

try {
  checkAge(-5);
} catch (error) {
  console.log(error.name);    // "ValidationError"
  console.log(error.message); // "Age cannot be negative"
}
```

**Breaking down class ValidationError extends Error:**
- `class ValidationError extends Error` - inherits everything a normal Error has (.message, .stack)
- `constructor(message)` - runs when `new ValidationError("text")` is created, receiving the text as message
- `super(message)` - calls the built-in Error's own constructor, passing the message along so Error's normal setup (like .stack) still happens properly. Without this line, the custom error would be missing standard Error behavior.
- `this.name = "ValidationError"` - overrides the default "Error" name so this specific error type can be told apart from a generic one

**Using instanceof to handle different error types differently:**
```js
try {
  checkAge(-5);
} catch (error) {
  if (error instanceof ValidationError) {
    console.log("Fix your input:", error.message);
  } else {
    console.log("Unexpected error:", error.message);
  }
}
```

## Doubts / Questions I had

**Q: Is class/extends/super just Java-style inheritance?**

A: It LOOKS like Java syntax, but underneath it's still prototype-based inheritance, same mechanism as Object.create from Day 6 - not true classical inheritance. `class ValidationError extends Error` is really just JS setting up `ValidationError.prototype`'s prototype to be `Error.prototype` - the same object-to-object linking, just with syntax designed to look familiar to people coming from class-based languages like Java.

Interview-ready one-liner: "JS's class/extends/super syntax was intentionally designed to look familiar to people coming from Java or other class-based languages, but underneath it's still doing prototype-based inheritance - the same object-to-object linking mechanism as Object.create, not true classical inheritance."

The exact same custom error behavior CAN be built with plain prototypes and Object.create instead of class syntax - class doesn't add a new capability, it's just a cleaner, less error-prone way to write the same underlying prototype mechanism.

## My practice exercise - written and debugged myself

```js
class NegativeNumberError extends Error {
  constructor(message) {
    super(message);
    this.name = "NegativeNumberError";
  }
}

function checkPositive(num) {
  if (num < 0) {
    throw new NegativeNumberError("Number cannot be negative");
  }
  return num;
}

try {
  checkPositive(-1);
} catch (error) {
  console.log(error.message); // "Number cannot be negative"
}
```

**Mistakes made and fixed:**
1. Initially wrote `constructor(message);` with a semicolon and no `{ }` body, with `super(message)` and `this.name = ...` sitting outside/after it instead of inside the constructor's curly braces. Fixed by wrapping both lines inside `constructor(message) { ... }`.
2. Typo'd the function call as `checkPostive(-1)` instead of `checkPositive(-1)` - would have thrown "checkPostive is not defined" instead of the intended NegativeNumberError. Fixed by matching the exact function name.

## Key takeaway (in my own words)

finally runs regardless of success or failure, useful for cleanup that must always happen. Throwing an error (rather than returning an error value) forces the caller to explicitly handle failure via try/catch instead of silently allowing a bug to slip through. Custom error classes via extends Error allow different failure types to be distinguished and handled differently using instanceof, and the class/extends/super syntax is really prototype-based inheritance underneath, just with syntax designed to look like classical languages such as Java.