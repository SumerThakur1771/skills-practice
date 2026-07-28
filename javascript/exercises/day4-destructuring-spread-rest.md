# Week 1 - Day 4: Object Methods, Destructuring, Spread, Rest

## Object Methods

```js
const person = { name: "Sumer", age: 25 };

Object.keys(person);    // ["name", "age"]
Object.values(person);  // ["Sumer", 25]
Object.entries(person); // [["name", "Sumer"], ["age", 25]]
```

`Object.keys` gives property names, `Object.values` gives values, `Object.entries` gives both as [key, value] pairs — useful for looping over an object using array methods like forEach/map, which normally only work on arrays.

## Object Destructuring

A shortcut for pulling values out of an object into their own variables, instead of writing `object.property` repeatedly.

```js
// Manual way
const name = person.name;
const age = person.age;

// Destructuring way - same result
const { name, age } = person;
```

**Critical rule:** the variable names on the left must exactly match the property names in the object. It's not freely naming new variables — it's telling JS "find a property with this exact name and put its value here."

**Renaming while destructuring** (optional):
```js
const { name: fullName } = person;
console.log(fullName); // "Sumer"
```

**Skipping properties** - just don't name them:
```js
const car = { brand: "Toyota", year: 2022, color: "blue" };
const { brand, color } = car;
// year is never touched
```

## Array Destructuring

Same idea, but matches by **position/index**, not by name (arrays don't have named properties).

```js
const colors = ["red", "green", "blue"];
const [first, second, third] = colors;
// first = "red", second = "green", third = "blue"
```

Variable names can be anything you want, since matching is purely by order.

**Skipping items** with a blank comma:
```js
const scores = [95, 82, 77];
const [highest, , lowest] = scores;
// highest = 95, lowest = 77 (82 skipped)
```

## Spread ( ... ) - Expanding Things Out

Used to copy or combine arrays/objects. Spread is used on the RIGHT side of an assignment, or inside a NEW `{ }` / `[ ]` you're building.

**Copying an array** (creates a real separate copy, not a reference to the same array):
```js
const nums = [1, 2, 3];
const copy = [...nums]; // [1, 2, 3], separate from nums
```

**Combining arrays:**
```js
const a = [1, 2];
const b = [3, 4];
const combined = [...a, ...b]; // [1, 2, 3, 4]
```

**Copying/combining objects:**
```js
const person = { name: "Sumer", age: 25 };
const personCopy = { ...person };
```

**Combining objects with override** - later spreads override earlier ones for matching keys:
```js
const base = { a: 1, b: 2 };
const overrides = { b: 99, c: 3 };
const result = { ...base, ...overrides };
// { a: 1, b: 99, c: 3 } - b gets overwritten since overrides comes second
```

## Rest ( ... ) - Collecting Things Together

Same symbol as spread, opposite job. Rest is used on the LEFT side of a destructuring assignment, or in function parameters - it gathers leftover/extra values into one array or object.

**Rest in function parameters** - collects any number of arguments into an array:
```js
function greet(first, ...others) {
  console.log(first);
  console.log(others);
}
greet("Sumer", "Dylan", "Kellin", "Meg");
// logs: "Sumer"
// logs: ["Dylan", "Kellin", "Meg"]  <- collected as a real array
```

**Rest in object destructuring** - collects whatever's left after pulling out named items:
```js
const { name, ...rest } = { name: "Sumer", age: 25, city: "Boston" };
// name = "Sumer"
// rest = { age: 25, city: "Boston" }
```

**Rest in array destructuring:**
```js
const [x, ...y] = [1, 2, 3, 4];
// x = 1
// y = [2, 3, 4]
```

## Doubts / Questions I had

**Q: In object destructuring, how do I tell spread and rest apart when the syntax looks identical?**

A: It depends on which side of `=` the `...` is on, and whether you're building something new or pulling values out.

- REST: left side of a destructuring assignment, pulling values OUT of something and creating new variables.
  ```js
  const { a, ...rest } = { a: 1, b: 2, c: 3 };
  // rest = { b: 2, c: 3 }
  ```
- SPREAD: right side / inside a fresh `{ }` or `[ ]` you're constructing, expanding an existing object/array's contents into it.
  ```js
  const combined = { a: 1, ...someOtherObject };
  ```

Simple test: am I on the left side pulling values out and naming new variables (rest)? Or am I on the right side / inside `{ }` `[ ]` building something new (spread)?

## Key takeaway (in my own words)

Destructuring pulls values out of objects (by matching name) or arrays (by matching position) into their own variables in one line instead of doing it one at a time. Spread expands an existing array/object's contents outward, used when building something new (copying, combining). Rest does the opposite - it collects multiple leftover values together into a single array or object, used in function parameters or on the left side of destructuring.