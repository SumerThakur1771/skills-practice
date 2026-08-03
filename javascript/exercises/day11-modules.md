# Week 2 - Day 11: Modules - import/export, named vs default

## WHY modules exist

As a codebase grows, keeping everything in one giant file becomes impossible to navigate, and risks naming collisions (same variable/function names overwriting each other). Modules let code be split into separate files, where each file explicitly controls what it shares (export) and what it pulls in from elsewhere (import). This is the exact structure already used in real projects - every file in Next.js/React is a module.

## Named exports

A file can export MULTIPLE named things - functions, variables, classes, anything.

```js
// mathUtils.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

Imported by EXACT name, wrapped in `{ }`:

```js
// app.js
import { add, subtract } from "./mathUtils.js";
console.log(add(2, 3)); // 5
```

The names inside `{ }` must match exactly what was exported - similar to the "names must match" rule from object destructuring (import syntax is intentionally modeled after destructuring).

## Default exports

A file can have exactly ONE default export - meant to represent "the main thing this file provides."

```js
// greet.js
export default function greet(name) {
  console.log("Hi " + name);
}
```

Importing a default export does NOT use `{ }`, and can be named ANYTHING on the importing side - doesn't need to match the original name:

```js
// app.js
import sayHello from "./greet.js";
sayHello("Sumer"); // "Hi Sumer"
```

`sayHello` is a made-up name here, different from `greet` in the original file - allowed specifically because it's a default export, not named.

## Mixing both in one file

```js
// utils.js
export const PI = 3.14;
export default function square(x) {
  return x * x;
}
```

**Import order rule:** default import comes FIRST, unmarked, then named imports follow inside `{ }`, separated by a comma.

```js
// app.js
import square, { PI } from "./utils.js";
```

## My practice exercise

Given:
```js
// shapes.js
export function circleArea(r) { return 3.14 * r * r; }
export default function square(x) { return x * x; }
```

**Mistake made:** initially wrote `import {circleArea}, squareOf from "./shape.js"` - had the order backwards (named import listed first, default listed second and unwrapped). Corrected to match the actual rule: default import first (renamed freely), then named import in `{ }` second.

**Correct answer:**
```js
import squareOf, { circleArea } from "./shapes.js";
```

`squareOf` is a freely chosen name for the default export (originally `square` in the file). `{ circleArea }` is the named export and must match its exported name exactly - unless using `as` to rename it (e.g. `{ circleArea as area }`), but the original name must still appear on the left side of `as`.

## Renaming with "as"

**Export-side renaming** - changes the name PERMANENTLY for anyone importing from that file:

```js
// mathUtils.js
function add(a, b) {
  return a + b;
}
export { add as sum };
```

```js
// app.js - must import using the renamed name "sum", not "add"
import { sum } from "./mathUtils.js";
console.log(sum(2, 3)); // 5
```

**Import-side renaming** - only affects the importing file; the original export name stays unchanged for everyone else:

```js
// mathUtils.js - exported normally, no renaming here
export function add(a, b) {
  return a + b;
}
```

```js
// app.js - file exports it as "add", renamed locally on import to "sum"
import { add as sum } from "./mathUtils.js";
console.log(sum(2, 3)); // 5
```

Both are relatively rare in real code compared to plain named/default imports, but good to recognize.

**Namespace import** - importing everything from a file as one object:
```js
import * as MathUtils from "./mathUtils.js";
MathUtils.add(2, 3);
```

## File paths - relative imports (./  vs  ../)

This is separate from export/import naming - it's about WHERE the file physically lives relative to the file doing the importing. Wrong path = "module not found" error, regardless of whether names are correct.

- `./` = look in the SAME folder as this file
- `../` = go UP one folder, then look from there
- `../../` = go up TWO folders

```js
// If app.js and mathUtils.js are in the SAME folder:
import { add } from "./mathUtils.js";

// If mathUtils.js is in a folder ONE LEVEL UP from app.js:
import { add } from "../mathUtils.js";

// If mathUtils.js is in a "utils" subfolder, inside the same folder as app.js:
import { add } from "./utils/mathUtils.js";
```

## Key takeaway (in my own words)

Named exports let a file export multiple things, and importing them requires matching the exact name inside `{ }`. Default exports allow exactly one per file, imported without `{ }` and can be freely renamed on the importing side. When combining both in one import line, the default import always comes first, followed by a comma and the named imports in `{ }`.