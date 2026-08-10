# Week 3 - Day 2: Interfaces vs Type Aliases

## The problem both solve

Describing the SHAPE of an object - what properties it has, and what type each one is - without rewriting that shape every single time. Give it a name once, reuse everywhere.

## Interfaces

```ts
interface User {
  name: string;
  age: number;
  isActive: boolean;
}

const sumer: User = {
  name: "Sumer",
  age: 25,
  isActive: true,
};
```

Defines the shape once. `const sumer: User = {...}` says "this object must match the User shape exactly" - every property listed, with matching types. Missing a property or wrong type is caught immediately.

## Type aliases

```ts
type User = {
  name: string;
  age: number;
  isActive: boolean;
};
```

Looks nearly identical to the interface version - same shape, same usage. Difference: the keyword (`type` vs `interface`) and using `=` before the `{ }`. Type aliases require a trailing semicolon after the closing `}`; interfaces don't.

For simple object shapes like this, interface and type behave identically - the real differences show up in more specific scenarios.

## Difference 1: Extending / combining shapes

**Interfaces** use `extends`:
```ts
interface Animal {
  name: string;
}
interface Dog extends Animal {
  breed: string;
}
const myDog: Dog = { name: "Rex", breed: "Labrador" };
```

**Type aliases** use `&` (intersection) instead:
```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };
const myDog: Dog = { name: "Rex", breed: "Labrador" };
```

Both achieve the same result - Dog requires both name and breed - just different syntax for "build on top of an existing shape."

## Difference 2: Declaration merging (interfaces only - genuine capability gap)

Declaring an `interface` with the same name TWICE automatically MERGES them into one combined shape:

```ts
interface User {
  name: string;
}
interface User {
  age: number;
}
// User is now automatically: { name: string; age: number; }
const sumer: User = { name: "Sumer", age: 25 };
```

Type aliases CANNOT do this - declaring `type User` twice is a straight-up naming collision error:
```ts
type User = { name: string };
type User = { age: number }; // Error: Duplicate identifier 'User'
```

**Real-world use case for declaration merging:** extending third-party library types. If a library defines an interface (e.g. Express's Request object) and a custom property needs to be added in a project without modifying the library's source, declaration merging allows this:
```ts
interface Request {
  user?: { id: string }; // adding a custom property to an existing library interface
}
```

Also useful for a shape that's expected to grow/scale over time with new properties added incrementally.

## Difference 3: Type aliases can represent non-object types

```ts
type Status = "loading" | "success" | "error"; // interfaces CANNOT do this at all
```

Interfaces are specifically for describing object shapes - they can't represent "one of several possible values" (union types, covered in depth on Day 3).

## Practical guidance

Interfaces make sense when a shape might need to be extended/added to later (scaling scenario, or merging with library types). Type aliases make more sense when something should be more fixed/sealed and not silently extendable elsewhere in the codebase, or when representing something that isn't a plain object shape at all (unions, primitives).

**Interview-ready answer:** "Interfaces and type aliases are almost interchangeable for simple object shapes. The real differences: interfaces support declaration merging and are extended with extends; type aliases use & for combining and can represent non-object types like unions, which interfaces cannot. Many teams default to interface for object shapes and type for everything else, but it's largely a style/convention choice for simple cases."

## Doubts / Questions I had

**Q: Both interface and type seem useful in different scenarios - when would each actually be chosen in real code?**

A: If an object's shape might need properties added later (scaling over time), interface makes more sense since it stays "open" for extension - and declaration merging is genuinely used in real codebases for extending third-party library types (e.g. adding a custom property to Express's Request interface without touching the library's source). If something should be more fixed/sealed and not silently extendable elsewhere in a large codebase, or if it's not a plain object shape at all (unions, primitives), type makes more sense - since type aliases can't accidentally merge with another declaration of the same name.

**Q: Can an interface and a type alias share the same name, since declaration merging works for interfaces?**

A: No - declaration merging only works between MULTIPLE INTERFACES with the same name, not between an interface and a type alias. Initially tried declaring both `interface Book` and `type Book` using the same name `Book` for the practice exercise - this is a naming collision, not merging. They collide because both are trying to claim the same identifier. Fixed by using two different names (Book1, Book2) so both could exist side by side for comparison.

**Q: Do interface properties use commas or semicolons?**

A: Semicolons are the standard convention (commas technically work in some cases, but semicolons are what's expected in real code and interviews) - same separator style as a type alias's object shape.

## My practice exercise

Goal: write both an interface and type alias called Book (title: string, author: string, pages: number), and create one object using each.

**Final correct code:**

```ts
interface Book1 {
  title: string;
  author: string;
  pages: number;
}

type Book2 = {
  title: string;
  author: string;
  pages: number;
};

const book1: Book1 = {
  title: "How to make friends",
  author: "Daniel",
  pages: 350,
};

const book2: Book2 = {
  title: "How to make friends",
  author: "Daniel",
  pages: 350,
};
```

Both book1 and book2 are valid, type-checked objects, proving interface and type behave identically for a plain object shape.

## Key takeaway (in my own words)

Interfaces and type aliases behave the same for basic object shapes. Interfaces support declaration merging (multiple declarations with the same name automatically combine) and are extended with `extends` - useful for shapes expected to grow over time or for extending library types. Type aliases use `&` to combine shapes instead, cannot merge (a naming collision if declared twice), and can represent things beyond object shapes entirely, like union types, which interfaces cannot do at all.