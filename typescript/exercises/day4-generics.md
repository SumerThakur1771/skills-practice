# Week 3 - Day 4: Generics

## The problem generics solve - motivating example

```ts
function identity(value: number): number {
  return value;
}
```

Works for numbers only. What if it also needs to work for strings, booleans, objects?

**Bad option 1 - separate function per type:**
```ts
function identityNumber(value: number): number { return value; }
function identityString(value: string): string { return value; }
```
Repetitive, doesn't scale.

**Bad option 2 - use any:**
```ts
function identity(value: any): any {
  return value;
}
```
"Works" for every type but loses ALL type safety - TypeScript can no longer tell anything about what comes out.
```ts
const result = identity(5);
result.toUpperCase(); // No error from TypeScript, but WILL crash at runtime
```

## What generics actually do

Write ONE function that works with ANY type, while still PRESERVING the connection between what type went in and what type comes out - no loss of safety like `any`.

```ts
function identity<T>(value: T): T {
  return value;
}
```

`<T>` goes right after the function name, before the parentheses.

## What T represents

A placeholder for a TYPE - same idea as a function parameter, but standing in for a type instead of a value, decided at the moment the function is actually CALLED, not when it's written.

```ts
const result1 = identity(5);
// TypeScript sees a number passed in, fills in T = number
// result1 is inferred as type: number

const result2 = identity("hello");
// T = string
// result2 is inferred as type: string
```

Unlike `any`, this PRESERVES the real type. `result1.toUpperCase()` would correctly throw an error since TypeScript knows result1 is a number, not a string - it tracked exactly what T was for that specific call.

## Explicit type specification (optional)

```ts
const result3 = identity<string>("hello");
// explicitly telling TypeScript T = string, rather than letting it infer
```

Rarely necessary for simple cases since TypeScript usually infers correctly on its own - useful when the type can't be cleanly inferred from arguments alone.

## The core mental model

A generic function is like a template - not committing to one specific type when writing the function, but saying "whatever type comes in, that same type should come out (or be used consistently)." T gets filled in fresh, individually, every single call, based on that call's actual arguments.

## Practice check - firstElement

```ts
function firstElement<T>(arr: T[]): T {
  return arr[0];
}
```

- `firstElement([1, 2, 3])` -> T inferred as number, returns 1, typed as number
- `firstElement(["a", "b", "c"])` -> T inferred as string, returns "a", typed as string

## Generics with multiple type parameters

```ts
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair("Sumer", 25);
// T = string, U = number
// result typed as: [string, number]
```

Can have as many placeholder type names as needed (T, U, V, etc. - conventionally single uppercase letters, though descriptive names work too). Each is independently inferred from its corresponding argument.

## My practice exercise

Goal: generic function wrapInArray that takes a single value of any type, returns an array containing just that value.

**Final correct code:**

```ts
function wrapInArray<T>(val: T): T[] {
  return [val];
}

wrapInArray(5);    // [5], typed as number[]
wrapInArray("hi"); // ["hi"], typed as string[]
```

## Doubts / Questions I had

**Q: Why is `"string"` written with quotes in `typeof notSure === "string"`, when types are normally written without quotes like `let x: string`?**

A: These are two completely different things that happen to both involve the word "string." A TYPE ANNOTATION (`let username: string`) refers to the type itself, written directly with no quotes - it's checked by TypeScript at compile-time only, then stripped away. `typeof` is plain JavaScript (not TypeScript-specific) - an operator that runs AT RUNTIME and evaluates to an actual STRING VALUE describing what type something currently is:
```ts
console.log(typeof "hello"); // logs the actual string "string"
console.log(typeof 5);       // logs the actual string "number"
```
Since `typeof notSure` evaluates to a real string value at runtime, comparing it against specific text requires quotes, exactly like comparing any two strings: `if (typeof notSure === "string")` reads as "if the result of typeof notSure equals the literal text 'string'." One is a compile-time-only type; the other is real, executing JavaScript producing an actual string value.

**Q: My first wrapInArray attempt used `[T]` as the return type - what's wrong with that?**

A: `[T]` with square brackets containing a single type is TUPLE syntax (from Day 1/3) - meaning "an array with exactly one element of type T," a fixed-length structure. The exercise wanted a regular, flexible array of T instead, written as `T[]` (no brackets around T, just T followed by []). Recalling the Day 1 distinction:
```ts
let coordinates: [number, number]; // tuple - fixed length, exactly 2 numbers
let scores: number[];              // regular array - any length, all numbers
```
Same distinction applies with a generic placeholder: `[T]` = tuple with one T, `T[]` = flexible array of T.

## Key takeaway (in my own words)

Generics let one function work correctly across multiple types without losing type safety, unlike `any` which works for everything but tracks nothing. T is a placeholder for a type, filled in automatically by TypeScript based on what's actually passed in at each call, preserving an accurate connection between input type and output type. The same array-vs-tuple distinction from basic types applies to generic type placeholders too - T[] for a flexible array of T, [T] for a fixed one-element tuple containing a T.