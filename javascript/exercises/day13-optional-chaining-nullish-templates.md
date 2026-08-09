# Week 2 - Day 13: Optional Chaining, Nullish Coalescing, Template Literals

## Optional Chaining (?.)

**WHY it exists:** normally, accessing a property on something that's undefined or null throws an error and crashes the program.

```js
const user = { name: "Sumer" };
console.log(user.address.city); // TypeError: Cannot read properties of undefined
```

`user` has no `address` property, so `user.address` is `undefined` - and reading `.city` off of `undefined` crashes.

**What ?. does:** checks "is the thing on the left null or undefined? If so, stop right here and return undefined - don't crash, don't keep going."

```js
console.log(user.address?.city); // undefined, no crash
```

Also works with function calls:
```js
user.greet?.(); // does nothing, no crash - greet doesn't exist, skips calling it entirely
```

## My practice exercise

```js
const item = "coffee";
const price = 4.5;
console.log(`You bought ${item} for ${price}`);
// "You bought coffee for 4.5"
```

Note: if a literal $ sign is wanted before the price (e.g. "$4.5"), it must be typed as an actual character in the string - it's not automatically added: `` `You bought ${item} for $${price}` ``.

## Nullish Coalescing (??)

**WHY it exists:** providing a fallback/default value if something is null or undefined. The old approach using `||` has a bug - it treats ALL falsy values (0, "", false) as "missing," not just null/undefined.

```js
const userScore = 0;
const displayScore = userScore || "No score yet";
console.log(displayScore); // "No score yet" - WRONG, 0 is a valid real value
```

**What ?? does differently:** only falls back if the value is SPECIFICALLY null or undefined - not for other falsy values like 0, "", or false.

```js
const userScore = 0;
const displayScore = userScore ?? "No score yet";
console.log(displayScore); // 0 - correctly keeps the real value
```

**Practice example:**
```js
const settings = { volume: 0, theme: null };

settings.volume ?? 50;    // 0 - volume is a real value (0), not null/undefined, so it's kept
settings.theme ?? "dark"; // "dark" - theme IS null, so it falls back
```

## Template Literals

**WHY they exist:** building strings with + concatenation gets messy with lots of variables mixed in.

```js
const name = "Sumer";
const age = 25;

// Old way
const message = "Hi " + name + ", you are " + age + " years old.";

// Template literal way
const message2 = `Hi ${name}, you are ${age} years old.`;
```

**Syntax:**
- Use backticks (`) instead of quotes
- Drop any variable or expression directly inside `${ }`, right in the middle of the string - no breaking out with +
- Can put any valid expression inside `${ }`, not just a plain variable:

```js
const a = 5;
const b = 10;
console.log(`The sum of ${a} and ${b} is ${a + b}`);
// "The sum of 5 and 10 is 15"
```

**Bonus - multi-line strings:** template literals allow actual line breaks directly in the string, no \n needed:
```js
const multiLine = `Line one
Line two`;
```

## Doubts / Questions I had

**Q: In `user.address?.city`, shouldn't there be two dots - one for the `?.` and one before `city`, like `user.address?..city`?**

A: `?.` is ONE single operator, not a separate `?` plus a separate `.` after it. It REPLACES the normal `.` connector at that specific point - doesn't need an additional dot after it before the property name.

```js
user.address.city   // normal: user . address . city (two connectors)
user.address?.city  // safe:   user . address ?. city (still two connectors, one swapped for ?.)
```

**Q: Since `address` is the property that doesn't exist, shouldn't the `?.` go right after `user` instead - like `user?.address.city`?**

A: This was the trickiest part to get right. The crash doesn't happen ON the missing property itself - accessing a missing property (`user.address`) just evaluates to `undefined`, no crash happens at that step. The crash happens on the NEXT step - trying to read a property OFF OF that undefined value (`undefined.city`). So `?.` needs to go on the connector immediately BEFORE the step that's actually risky - i.e., right before going one level deeper into something that might be undefined.

```js
user.address?.city
```

Read as: "Get user.address (safe, might just be undefined). NOW before going deeper into .city - check: is what I just got null/undefined? If yes, stop safely."

`user` itself doesn't need `?.` here because `user` is a real, solid object - accessing `.address` on it is always safe. If `user` itself might ALSO be null/undefined (e.g. from a failed API call), THEN that step would need protecting too: `user?.address?.city`.

The general rule: place `?.` at each specific junction that's actually at risk of being null/undefined, based on what's genuinely uncertain - not automatically at the very start of the whole chain.

## Key takeaway (in my own words)

Optional chaining (?.) safely stops a property access chain right before it would try to reach into something that might be null/undefined, avoiding a crash - placed at the specific risky junction, not automatically at the start. Nullish coalescing (??) provides a fallback value only for null/undefined, correctly preserving other falsy-but-valid values like 0 or empty string, unlike ||. Template literals use backticks and ${ } to embed variables/expressions directly into a string without manual concatenation.