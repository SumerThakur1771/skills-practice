# Week 3 - Day 6: Enums, Type Assertions, How TS Compiles to JS

## Enums

**WHY they exist:** a fixed, known set of related values (directions, statuses, roles) needs a clean, readable name for each option instead of scattering raw strings/numbers throughout the code.

```ts
enum Role {
  Admin,
  Editor,
  Viewer,
}

let userRole: Role = Role.Admin;
console.log(userRole); // 0
```

By default, enums assign numbers starting at 0 - Admin = 0, Editor = 1, Viewer = 2. `Role.Admin` is much more readable than remembering "0 means admin" scattered across the codebase.

**String enums (often preferred):**
```ts
enum Role {
  Admin = "ADMIN",
  Editor = "EDITOR",
  Viewer = "VIEWER",
}

let userRole: Role = Role.Admin;
console.log(userRole); // "ADMIN"
```

Preferred over numeric enums because the actual value is meaningful on its own ("ADMIN" vs an unclear 0) - useful for debugging, logging, or if the value gets sent over an API.

**Enums vs union types:**
```ts
// Union type approach
type Role = "ADMIN" | "EDITOR" | "VIEWER";

// Enum approach
enum Role {
  Admin = "ADMIN",
  Editor = "EDITOR",
  Viewer = "VIEWER",
}
```

Both restrict a value to one of a few options. The genuine difference: enums are REAL VALUES that exist at runtime (compile into actual JS objects); union types are PURELY a compile-time construct that gets erased entirely - zero footprint in the final JS. Many modern TypeScript teams prefer union types over enums for this exact reason (simpler, no runtime overhead), but enums are still commonly asked about.

## Type Assertions (as)

**WHY they exist:** sometimes the developer knows more about a value's type than TypeScript can figure out on its own, and needs to override TypeScript's inference.

```ts
const someValue: unknown = "this is actually a string";
const strLength: number = (someValue as string).length;
```

`someValue as string` tells TypeScript: "treat this specific value as a string, even though you only know it as unknown." Once asserted, string-specific things (like .length) can be used on it.

**Common real scenario - DOM elements:**
```ts
const input = document.getElementById("username") as HTMLInputElement;
input.value = "Sumer"; // .value only exists on HTMLInputElement, not generic Element
```

`document.getElementById` normally returns `HTMLElement | null` - TypeScript doesn't know it's specifically an input element. Asserting `as HTMLInputElement` allows access to input-specific properties.

**Why type assertions are risky if overused:** `as` does NOT actually check or convert anything at runtime - pure compile-time instruction to trust the developer. If wrong, TypeScript won't catch it, and the code crashes at runtime exactly like plain JavaScript would.

```ts
const value: unknown = 42;
const wrongAssertion = value as string; // TypeScript allows this, no error
wrongAssertion.toUpperCase(); // Crashes at runtime - 42 has no toUpperCase method
```

Fundamentally similar risk to `any`, just applied to a single specific value instead of an entire variable.

**Assertion vs conversion - common misconception:** `as` does NOT convert a value from one type to another (unlike `Number("5")`, which genuinely converts a string to a number at runtime). It only tells TypeScript to TREAT the existing value as a different type for checking purposes - the actual value in memory never changes.

## How TypeScript compiles to JS

A tool called the TypeScript compiler (tsc) reads .ts files, checks all types for errors, and (assuming no errors) outputs plain .js files with type-related syntax stripped away - EXCEPT enums, which are the one exception since they produce real runtime objects.

```ts
// input.ts
interface User {
  name: string;
  age: number;
}

function greet(user: User): string {
  return `Hi ${user.name}`;
}

const sumer: User = { name: "Sumer", age: 25 };
console.log(greet(sumer));
```

```js
// output.js (after running tsc)
function greet(user) {
  return `Hi ${user.name}`;
}

const sumer = { name: "Sumer", age: 25 };
console.log(greet(sumer));
```

`interface User` is completely gone - it existed purely to check sumer's shape at compile time. Function signature lost its `: User` and `: string` annotations. Everything else (actual logic, template literal, function body) stays identical, since that's real JavaScript behavior, not type-checking syntax.

**Enum exception, shown concretely:**
```ts
enum Role {
  Admin = "ADMIN",
  Editor = "EDITOR",
}
```
```js
// this ACTUALLY exists in the compiled output, unlike a union type
var Role;
(function (Role) {
  Role["Admin"] = "ADMIN";
  Role["Editor"] = "EDITOR";
})(Role || (Role = {}));
```

## Interview-ready one-liner

"TypeScript adds zero runtime behavior differences - it's purely a compile-time type-checking layer that gets stripped away, and the actual executing code is identical plain JavaScript either way. The one exception is enums, which do produce real runtime objects since they're not just a type, they generate actual values."

## Practice check - type assertion risk

Given `const value: unknown = "hello";` - what does `(value as number).toFixed(2)` do?

**Answer:** Compiles fine (TypeScript trusts the assertion completely, no compile error), but crashes at RUNTIME - `value` is still genuinely the string "hello", strings don't have a `.toFixed()` method. `TypeError: value.toFixed is not a function`. This is exactly the risk of overusing `as` - it silences TypeScript's checking without making the underlying value actually safe.

## Doubts / Questions I had

**Q: To confirm - union types don't exist at all in the final stripped JS file, since they're purely compile-time?**

A: Correct. Union types (and every type annotation in general - string, number, interfaces, generics' <T>) exist ONLY during TypeScript's compile-time checking phase. Once compiled to plain JS, all of it gets completely stripped out - none of it makes it into the final .js file that actually runs. This is a concrete example of Day 1's core point: TypeScript is purely a development-time safety net.

**Q: But enums DO exist in the final stripped JS, since they're available at runtime, right?**

A: Correct - enums are the genuine exception to the "types get stripped away" rule, because they're not JUST a type, they also generate a real runtime value (an actual JS object). This was correctly reasoned through independently after seeing the union-vs-enum distinction explained.

## Key takeaway (in my own words)

TypeScript compiles down to plain JavaScript, stripping out all type-checking syntax (interfaces, type aliases, generics, most annotations) since they exist purely to catch mistakes before runtime. Enums are the one genuine exception - they generate real JS objects that exist and function at runtime. Type assertions (as) let a developer override TypeScript's own type inference, but this is a pure compile-time instruction with zero runtime safety - if the assertion is wrong, the code compiles fine but crashes at runtime, since the actual value never changes, only how TypeScript treats it for checking purposes.