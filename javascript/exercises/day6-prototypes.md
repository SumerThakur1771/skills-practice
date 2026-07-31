# Week 1 - Day 6: Prototypes and the Prototype Chain

## WHY this matters

Every object in JS is linked to another object called its "prototype," which it can borrow methods and properties from. This is how things like .map(), .filter(), .toString() exist on every array/object without writing them individually - they live on a prototype, not on each individual object.

## The core idea

When accessing a property on an object, JS first checks: does this object have it directly? If not, it checks the object's prototype. If not there, it checks the prototype's prototype, and so on up a chain - until it finds the property, or hits the end (null) and gives undefined.

```js
const arr = [1, 2, 3];
arr.map((x) => x * 2);
```

`arr` doesn't have its own personal copy of `.map()`. It's linked to `Array.prototype`, which has `.map()`, `.filter()`, `.forEach()`, etc. defined once, shared across every array. When calling `arr.map(...)`, JS checks `arr` first (not there), then checks `Array.prototype` (found it), and uses that.

## Why this exists / the benefit

Without prototypes, every single array would need its own private copy of `.map()`, `.filter()`, etc. - massively wasteful. Instead, all arrays share ONE copy of those methods via `Array.prototype`, and each array just has a link pointing to it.

## Example - Object.create

```js
const animal = {
  makeSound() {
    console.log("Some generic sound");
  },
};

const dog = Object.create(animal);
dog.bark = function () {
  console.log("Woof!");
};

dog.bark();       // "Woof!" - dog has this directly
dog.makeSound();  // "Some generic sound" - not on dog, found on its prototype (animal)
```

`Object.create(animal)` creates a new object (`dog`) whose prototype is `animal`. `dog` itself only has `bark`, but because it's linked to `animal` as its prototype, it can also use `makeSound` as if it were its own.

## My practice exercise - written from scratch

```js
const vehicle = {
  startEngine() {
    console.log("Engine started");
  },
};

const car = Object.create(vehicle);
car.honk = function () {
  console.log("Beep beep!");
};

car.startEngine(); // "Engine started" - found on car's prototype (vehicle)
car.honk();         // "Beep beep!" - car's own method, directly on car
```

**What's happening step by step:**
- `car.startEngine()` - JS checks car first, doesn't find startEngine there, walks up to car's prototype (vehicle), finds it there, runs it.
- `car.honk()` - JS checks car first, finds honk right there directly (added with `car.honk = function(){...}`), runs it immediately without needing to walk the chain at all.

**Mistake made along the way:** Initially tried to add `honk` using shorthand method syntax (`honk(){...}`) as a standalone statement outside any object - this isn't valid, since that shorthand only works inside an object literal `{ }`. Fixed by using `car.honk = function(){...}` to add a method onto an already-existing object using dot notation (same pattern as adding any property after object creation).

## Key takeaway (in my own words)

Prototypes let objects share and inherit methods/properties from another linked object instead of each object needing its own private copy. When accessing a property, JS checks the object itself first, then walks up the prototype chain until it finds a match or reaches the end.