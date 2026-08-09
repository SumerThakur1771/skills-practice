# Week 3 - Day 1: Why TypeScript Exists, Type Inference, Basic Types

## The problem TypeScript solves

JavaScript is dynamically typed - it doesn't check what type of data something is until the code actually runs. A mistake like passing a string where a number was expected often doesn't get caught until the program is already running, sometimes in production, sometimes in a rarely-used code path.

```js
// plain JavaScript - no error until this actually runs
function double(num) {
  return num * 2;
}
double("five"); // NaN - a silent, confusing bug, not caught until runtime
```

TypeScript adds a type system on top of JavaScript. Types are declared (or inferred), and TypeScript checks the code BEFORE it ever runs - usually right in the editor, well before trying to run it.

```ts
function double(num: number) {
  return num * 2;
}
double("five"); // Error caught immediately: Argument of type 'string' is not assignable to parameter of type 'number'
```

## Critical fact: TypeScript is not a separate runtime

Browsers and Node.js cannot run TypeScript directly. TypeScript files get COMPILED DOWN to plain JavaScript before they ever execute, and during compilation all type annotations get stripped away completely. The final .js file that runs has zero trace of `: number` or `: string`.

```ts
// TypeScript (what you write)
function double(num: number): number {
  return num * 2;
}
```
```js
// Plain JavaScript (what actually runs, after compiling)
function double(num) {
  return num * 2;
}
```

Types exist purely as a development-time safety net - they catch mistakes while writing code, but have zero effect on how the code actually behaves once running. The .js file with types stripped is what genuinely executes; TypeScript itself is never directly run by anything.

## Typing function parameters AND return values separately

```ts
function double(num: number): number {
  //            ^^^^^^^^^^^^  parameter type
  //                          ^^^^^^^^  return type
  return num * 2;
}
```

`num: number` types the parameter. The `: number` right after the closing `)` types the return value. TypeScript checks both independently:

```ts
function getLength(text: string): number {
  return text.length; // parameter is string, but returns a number - both checked separately
}
```

If the actual return value doesn't match the declared return type, TypeScript catches it immediately in the editor:
```ts
function double(num: number): number {
  return "not a number"; // Error: Type 'string' is not assignable to type 'number'
}
```

## Type inference vs explicit typing

**Explicit typing** - writing the type yourself:
```ts
let username: string = "Sumer";
```

**Type inference** - no type written, TypeScript figures it out from the assigned value:
```ts
let username = "Sumer"; // TS infers: string, automatically
let age = 25;            // TS infers: number
age = "twenty five";     // Error: Type 'string' is not assignable to type 'number'
```

Both approaches result in the same enforced type - the difference is only whether the type was written explicitly or inferred from the value. Once inferred, TypeScript still holds the variable to that type going forward.

## Primitive types

```ts
let username: string = "Sumer";
let score: number = 95;
let isActive: boolean = true;
```

## Arrays

```ts
let tags: string[] = ["frontend", "backend"]; // array where every item must be a string
let scores: number[] = [95, 82, 77];
```

## Tuples

A fixed-length array where each position has its own specific type - unlike a regular array (any length, all items same type).

```ts
let coordinates: [number, number] = [10, 20];
coordinates = [10, 20, 30]; // Error - too many items, expected exactly 2
coordinates = ["10", 20];   // Error - first position must be number
```

**Important: tuples need EXPLICIT typing to get the fixed-length guarantee.** Without an explicit type annotation, TypeScript infers a plain array (`number[]`), NOT a tuple, even if it happens to have exactly 2 items at that moment:

```ts
let coordinates = [10, 20]; // inferred as number[] (flexible array), NOT a tuple
coordinates.push(30);        // ALLOWED - number[] can be any length
console.log(coordinates);    // [10, 20, 30]

let point: [number, number] = [10, 20]; // explicitly typed tuple
point = [10, 20, 30]; // Error - blocked, expected exactly 2 elements
```

Just because an array happens to have a certain number of items when written doesn't make TypeScript treat it as a tuple - the fixed-length enforcement only applies when explicitly declared as a tuple type.

## any vs unknown

Both represent "could be any type," but they differ in ENFORCEMENT, not possibility.

**any** - completely opts OUT of type checking. TypeScript doesn't require any verification at all, even if usage is wrong:
```ts
let anything: any = fetchSomeData();
anything.toUpperCase(); // TypeScript allows this with NO warning, even if it's actually a number - would crash at runtime
```
Defeats the purpose of TypeScript if overused - relies entirely on the developer remembering to be careful, with zero compiler safety net.

**unknown** - the safer alternative. Also means "could be anything," but TypeScript FORCES verification of the actual type before it can be used:
```ts
let notSure: unknown = fetchSomeData();
notSure.toUpperCase(); // Error - TypeScript REFUSES to compile this until the type is proven

if (typeof notSure === "string") {
  notSure.toUpperCase(); // fine now - TS knows it's a string within this block
}
```

**The key distinction:** with `any`, a safety check could be added manually, but TypeScript never requires it - if forgotten, broken code slips through silently until it crashes at runtime. With `unknown`, TypeScript makes the check MANDATORY - it's not optional developer discipline, the compiler physically won't allow risky usage until the type is narrowed down first.

## void vs never

**void** - function doesn't return anything meaningful, but completes normally:
```ts
function logMessage(message: string): void {
  console.log(message);
  // no return statement
}
```

**never** - function NEVER successfully completes at all - not "returns nothing," but "never even reaches a point of returning," because it always throws or runs forever:
```ts
function throwError(message: string): never {
  throw new Error(message); // the ONLY path - always throws, never completes
}
```

**The distinguishing test:** does the function have ANY path that completes normally? If yes (even alongside a throw path), it's `void`. If EVERY path always throws or never terminates, it's `never`.

```ts
// void - has a path that completes normally (valid input case)
function validateInput(input: string): void {
  if (!input) {
    throw new Error("Input cannot be empty"); // one possible path
  }
  console.log("Valid input"); // the OTHER path - completes normally, returns nothing
}
```

## Key takeaway (in my own words)

TypeScript compiles down to plain JavaScript with all type annotations stripped out - it's purely a development-time safety net that catches type mistakes before code ever runs, with zero effect on actual runtime behavior. Type inference lets TypeScript figure out types automatically from assigned values, but some cases (like tuples) need explicit typing to get the stricter guarantees. `any` disables type checking entirely and trusts the developer; `unknown` forces verification before use, with the compiler enforcing it rather than relying on developer discipline. `void` means a function completes normally without a meaningful return value; `never` means a function has no path that ever completes normally at all.