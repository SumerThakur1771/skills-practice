# Week 3 - Day 5: Utility Types (Partial, Required, Pick, Omit, Record)

## The motivating problem

```ts
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}
```

Writing a brand new interface by hand for every variation (e.g. an "update user" type where only some fields are needed) would be repetitive and easy to get out of sync if User ever changes. Utility types solve this: deriving a new type from an existing one automatically, based on a specific transformation.

## Partial<T>

Makes EVERY property optional.

```ts
type PartialUser = Partial<User>;
// equivalent to: { id?: number; name?: string; email?: string; age?: number; }

function updateUser(id: number, updates: Partial<User>) {
  // updates can have any subset of User's fields, or none at all
}

updateUser(1, { name: "New Name" }); // valid
updateUser(1, {}); // also valid
```

## Required<T>

The opposite - makes EVERY property mandatory, even ones originally optional.

```ts
interface Draft {
  title?: string;
  content?: string;
}

type PublishedPost = Required<Draft>;
// equivalent to: { title: string; content: string; }
```

Useful for a type with optional fields during a "draft" phase, needing to guarantee everything is filled in by a later stage (e.g. "publish").

## Pick<T, Keys>

Creates a new type using ONLY specific properties selected from an existing type.

```ts
type UserPreview = Pick<User, "id" | "name">;
// equivalent to: { id: number; name: string; }

const preview: UserPreview = { id: 1, name: "Sumer" }; // valid
const preview2: UserPreview = { id: 1, name: "Sumer", email: "x@x.com" }; // Error - email isn't part of UserPreview
```

Give it the original type, and a UNION of property names (as string literals) to keep - everything else gets dropped.

## Omit<T, Keys>

The opposite of Pick - creates a new type with EVERYTHING EXCEPT the specific properties named.

```ts
type UserWithoutEmail = Omit<User, "email">;
// equivalent to: { id: number; name: string; age: number; }
```

Same syntax pattern as Pick, but excludes instead of includes.

**When to use which:** Pick when only needing a small slice of a larger type (e.g. a preview card showing just id and name). Omit when wanting "everything except this one sensitive/irrelevant field" (e.g. sending a user object to the frontend but stripping password or an internal-only field).

## Record<Keys, ValueType>

Builds an object type specifying what the KEYS can be and what type every VALUE must be - useful for dictionary/map-like objects.

```ts
type Scores = Record<string, number>;

const scores: Scores = {
  math: 95,
  science: 88,
  history: 76,
};
```

Unlike Pick/Omit (which work off an existing type's specific properties), Record builds a brand new object type from scratch based on a key pattern and value type.

**Record with a union of specific keys (not just any string):**
```ts
type Status = "loading" | "success" | "error";

type StatusMessages = Record<Status, string>;
// equivalent to: { loading: string; success: string; error: string; }

const messages: StatusMessages = {
  loading: "Please wait...",
  success: "Done!",
  error: "Something went wrong",
};
```

Combines a union type (Day 3) with Record - locking down exactly which keys are allowed while requiring every one of them to be present.

## My practice exercises

**Pick exercise:**
```ts
type UserContactInfo = Pick<User, "email" | "age">;
```

**Record exercise:**
```ts
type StatusColors = Record<Status, string>;

const obj: StatusColors = {
  loading: "blue",
  success: "green",
  error: "red",
};
```

## Doubts / Questions I had

**Q: Do utility types only work with `type`, or also with `interface`?**

A: Utility types work with BOTH - they operate on the SHAPE itself, not on which keyword was used to define it.
```ts
interface User { id: number; name: string; }
type PartialUser = Partial<User>; // works fine

type Product = { id: number; title: string; };
type PartialProduct = Partial<Product>; // also works fine
```
Makes sense given Day 2's finding that interface and type behave nearly identically for plain object shapes - utility types just care about the resulting shape. One detail: the OUTPUT of a utility type is always a `type`, not an `interface`, even if the original was an interface - doesn't functionally matter for usage though.

**Q: Is `interface PartialUser : Partial<User> {...}` valid syntax?**

A: No, for two reasons. First, interfaces don't use `:` to extend/build on something - they use `extends` (from Day 2): `interface PartialUser extends Partial<User> { ... }`. Second, even with correct syntax, this is unusual to write because `Partial<User>` already produces a complete type - wrapping it in another interface with nothing added is unnecessary. If just naming the result of a utility type with nothing added, the natural way is a type alias: `type PartialUser = Partial<User>;`. `interface ... extends` makes more sense when genuinely adding NEW properties on top:
```ts
interface PartialUserWithNote extends Partial<User> {
  note: string; // genuinely new property
}
```
Rule of thumb: plain renaming/aliasing a utility type's output -> `type =`. Adding new properties on top -> `interface extends`.

## Key takeaway (in my own words)

Utility types derive new types from existing ones automatically, without rewriting shapes by hand, and work the same whether the original was declared with `type` or `interface`. Partial/Required toggle whether every property is optional or mandatory. Pick/Omit select or exclude specific properties from an existing type. Record builds a brand new object type from a key pattern and value type, optionally combined with a union type to lock down exactly which keys are allowed.