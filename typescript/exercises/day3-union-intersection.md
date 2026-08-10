# Week 3 - Day 3: Union Types, Intersection Types, Optional & Readonly Properties

## Union types (|)

**WHY they exist:** sometimes a value can legitimately be one of SEVERAL types, not always exactly one fixed type.

```ts
let id: string | number;
id = "abc123"; // valid
id = 42;        // also valid
id = true;      // Error - boolean is not part of the union
```

`string | number` means "this can be a string OR a number, nothing else." Read `|` as "or."

**Common real use case - literal type unions:**
```ts
type Status = "loading" | "success" | "error";
let currentStatus: Status = "loading";
currentStatus = "success"; // valid
currentStatus = "done";    // Error - "done" isn't one of the allowed values
```

Instead of allowing any string, only these exact specific string values are allowed. Very common practical pattern for status flags, button variants, API response states.

## Intersection types (&)

**WHY they exist:** combining multiple shapes into one, requiring everything from all of them at once.

```ts
type Name = { name: string };
type Age = { age: number };
type Person = Name & Age;

const sumer: Person = { name: "Sumer", age: 25 };
```

`Name & Age` means "must satisfy Name's requirements AND Age's requirements, both at the same time." Missing either name or age throws an error.

## Union vs intersection - the core distinction

`|` = "could be this, OR that" (fewer requirements, more flexibility in what's allowed - only needs to satisfy ONE shape)
`&` = "must be this, AND that, combined" (more requirements, stricter - needs everything from EVERY piece)

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

const cat: Cat | Dog = { meow: () => console.log("meow") }; // valid, satisfies Cat
const dog: Cat | Dog = { bark: () => console.log("woof") }; // valid, satisfies Dog

const catDog: Cat & Dog = {
  meow: () => console.log("meow"),
  bark: () => console.log("woof"),
}; // valid - has BOTH, satisfies the intersection
```

## Optional properties (?)

Sometimes an object property might or might not be present.

```ts
interface User {
  name: string;
  age: number;
  nickname?: string; // optional - ? marks it as not required
}

const user1: User = { name: "Sumer", age: 25 }; // valid - nickname omitted entirely
const user2: User = { name: "Sumer", age: 25, nickname: "Sam" }; // also valid
```

Without `?`, TypeScript requires that property on every object of that type. `?` says "allowed, but not mandatory."

## Readonly properties

A property that can be set once, but never changed afterward.

```ts
interface Config {
  readonly apiKey: string;
  timeout: number;
}

const config: Config = { apiKey: "abc123", timeout: 5000 };

config.timeout = 10000;   // allowed - not readonly
config.apiKey = "xyz789"; // Error - cannot assign to 'apiKey' because it is a read-only property
```

Locks that specific property after its initial assignment - useful for things that genuinely shouldn't change after creation, like an ID or a config value set once at startup.

## My practice exercise

Goal: interface Product with id (readonly number), name (required string), discount (optional number).

**Final correct code:**

```ts
interface Product {
  readonly id: number;
  name: string;
  discount?: number;
}

const product1: Product = { id: 1, name: "Laptop" }; // valid - discount omitted
const product2: Product = { id: 2, name: "Phone", discount: 10 }; // valid - discount included

product1.id = 5; // Error - readonly, cannot reassign
```

## Doubts / Questions I had

**Q: What's the actual difference in what Cat | Dog allows versus what Cat & Dog requires?**

A: `Cat | Dog` only requires satisfying ONE of the two shapes (either has meow, or has bark) - flexible, fewer requirements. `Cat & Dog` requires satisfying BOTH shapes at once (must have meow AND bark) - strict, combined requirements. Reasoned through this correctly on the first attempt: union = fewer requirements/more flexibility, intersection = stricter/combined requirements.

**Q: Typo caught - wrote `Interface` with capital I instead of lowercase `interface`.**

A: TypeScript keywords are case-sensitive, same as the rest of JS - must be lowercase `interface`.

## Key takeaway (in my own words)

Union types (|) allow a value to be one of several possible types, satisfying just one of them - useful for things like status flags or IDs that could be a string or number. Intersection types (&) combine multiple shapes into one, requiring everything from all pieces at once. Optional properties (?) mark a field as not required on every instance of that shape. Readonly properties can be set once at creation but never reassigned afterward, useful for values like IDs or config that shouldn't change after being set.