# Week 4 - Day 1: React Fundamentals - JSX, Components, Props

## WHY React exists

Manually keeping the DOM in sync with changing data (covered in the DOM fundamentals session) is repetitive and error-prone as an app grows - every place data appears needs its own manual update logic, and missing one causes stale UI bugs. React solves this by letting the UI be described declaratively based on current data, and React itself figures out what actually needs to change in the real DOM and updates it.

## JSX - what it is and its purpose

JSX is syntax that looks like HTML, written directly inside JavaScript code. It is NOT a separate file type or language - a React component file is a normal JavaScript (.js or .jsx) file that uses JSX syntax specifically for the "what should render on screen" parts. Everything else in the file (variables, functions, hooks, logic) is completely ordinary JavaScript.

```jsx
const element = <h1>Hello</h1>;
```

**Why JSX exists - the actual purpose:** to describe UI structure in a way that's easy to read, instead of writing it the "hard way" with nested function calls.

```js
// Without JSX - the hard way
React.createElement("div", null,
  React.createElement("h1", null, "Hello"),
  React.createElement("p", null, "Welcome to the page")
);
```
```jsx
// With JSX - same result, easy to read
<div>
  <h1>Hello</h1>
  <p>Welcome to the page</p>
</div>
```

Both produce the exact same result on screen. JSX just lets the actual page layout be visualized immediately from the code, rather than parsing nested function calls.

JSX gets COMPILED into plain JavaScript before it ever runs in the browser - browsers cannot run JSX directly. This connects to TypeScript's Day 1 lesson: JSX, like TS types, is a layer that gets stripped/converted away into something the browser actually understands, before execution.

## Components

A component is just a REGULAR JAVASCRIPT FUNCTION that returns JSX, describing a piece of UI.

```jsx
function Welcome() {
  return <p>Welcome to WHOOP</p>;
}
```

That is the entire definition - a function, like any other function, that happens to return JSX instead of a number or string. (LoginForm from the WHOOP assessment was already a component, without it being explicitly named as such at the time.)

**Using/rendering a component** - written like an HTML tag, using its function name:

```jsx
function App() {
  return (
    <div>
      <Welcome />
    </div>
  );
}
```

`<Welcome />` means "run the Welcome function, and put whatever JSX it returns right here."

## Props - passing data into a component

**The problem props solve:** a component like Welcome always shows the exact same content - props let a single component be reused with different data instead of writing a new component for every variation.

```jsx
function Welcome(props) {
  return <p>Welcome, {props.name}</p>;
}

<Welcome name="Sumer" />
```

`props` is a regular JavaScript object, automatically built by React from whatever attributes are written on the tag. Writing `<Welcome name="Sumer" />` makes React call `Welcome({ name: "Sumer" })` - same as calling any regular function with an object argument.

`{props.name}` inside JSX - the curly braces `{ }` let a plain JavaScript value be dropped directly into JSX content.

**Destructuring props directly** (the pattern already used in LoginForm):

```jsx
function Welcome({ name }) {
  return <p>Welcome, {name}</p>;
}
```

Same thing, using object destructuring (Day 4 JS material) directly in the function parameter instead of writing `props.name` repeatedly.

## My practice exercises

**Exercise 1 - Welcome + App components:**
```jsx
function Welcome() {
  return <p>Welcome to WHOOP</p>;
}

function App() {
  return (
    <div>
      <Welcome />
    </div>
  );
}
```
Correct on first attempt.

**Exercise 2 - rendering UserCard with props:**
```jsx
function UserCard({ username, age }) {
  return <p>{username} is {age} years old</p>;
}
```

**Mistakes made and fixed:**
1. First attempt used `userName` (capital N) when calling the component, but UserCard destructures `username` (lowercase, one word) - since these don't match exactly, `username` inside UserCard would be undefined. React does not do fuzzy prop-name matching; the prop name on the tag must exactly match what the component destructures.
2. Initially passed `age="30"` (a string) instead of `age={30}` (an actual number) - fine for plain display text, but would matter if age were ever used in math (e.g. age + 1). Fixed to `age={30}`.

**Final correct code:**
```jsx
function App() {
  return (
    <div>
      <UserCard username="Alex" age={30} />
    </div>
  );
}
```

## Key takeaway (in my own words)

JSX is HTML-like syntax used inside regular JavaScript to describe what should render on screen, and it compiles down to plain JS before running. A component is just a function that returns JSX. Props let data be passed into a component the same way arguments are passed into any function, letting one component be reused with different data - the prop name on the tag must exactly match what the component destructures, or it comes through as undefined.