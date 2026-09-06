# Week 4 - Day 5: useContext

## The problem useContext solves - "prop drilling"

Sometimes a piece of data needs to reach a component nested several levels deep inside other components.

```jsx
function App() {
  const user = { name: "Sumer" };
  return <Dashboard user={user} />;
}

function Dashboard({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserBadge user={user} />;
}

function UserBadge({ user }) {
  return <p>{user.name}</p>;
}
```

`Dashboard` and `Sidebar` receive `user` as a prop but never actually use it themselves - they just pass it straight through to the next component. This pattern - passing a prop down through components that don't need it themselves, purely to relay it further down - is called PROP DRILLING.

**Why this is a real problem, not just mildly annoying:** with 5-6 components stacked between where data originates and where it's actually used, every single middle component has to know about and pass along that prop, even though it doesn't care about it. Changing the shape of that data, or how it flows, means editing every component in the entire chain, not just the one that actually uses it.

## The core idea behind useContext

Context lets a value be put somewhere that ANY component underneath it in the tree can read directly - no matter how deeply nested - without it passing through every component in between.

**Analogy:** a radio broadcast. Instead of relaying a message person-to-person down a chain (everyone in between has to physically pass it along), the message is broadcast once, and anyone tuned in can pick it up directly, regardless of how far away they are.

## Setting it up - three steps

**Step 1: Create the context** - this is like creating "the radio channel" itself. Nothing is broadcast yet, nobody's listening.

```jsx
import { createContext } from "react";

const UserContext = createContext();
```

**Step 2: Provide the value** - wraps part of the component tree, broadcasting a value to everything inside it.

```jsx
function App() {
  const user = { name: "Sumer" };

  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}
```

`<Dashboard />` no longer receives `user` as a prop at all - it's wrapped inside the Provider. Anything nested inside the Provider now COULD access `user`, without it being explicitly passed down.

**Step 3: Consume the context** - "tuning in" to the channel, directly, from any descendant component, regardless of nesting depth.

```jsx
import { useContext } from "react";

function UserBadge() {
  const user = useContext(UserContext);
  return <p>{user.name}</p>;
}
```

## Critical detail: no more prop destructuring for the context value

Since `UserBadge` no longer receives `user` as a prop, it must NOT write `UserBadge({ user })` anymore - that would try to destructure a prop that no caller is passing, resulting in `undefined`. `user` now comes ENTIRELY from `useContext(UserContext)`, a completely different source than props.

```jsx
// WRONG - user is not a prop anymore, this would be undefined
function UserBadge({ user }) {
  return <p>{user.name}</p>;
}

// RIGHT - user comes from useContext, not props
function UserBadge() {
  const user = useContext(UserContext);
  return <p>{user.name}</p>;
}
```

## The full picture together

```jsx
function App() {
  const user = { name: "Sumer" };
  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}

function Dashboard() {
  return <Sidebar />; // no user prop needed anymore
}

function Sidebar() {
  return <UserBadge />; // no user prop needed anymore
}

function UserBadge() {
  const user = useContext(UserContext);
  return <p>{user.name}</p>;
}
```

`Dashboard` and `Sidebar` no longer need to accept or pass along the `user` prop at all - `UserBadge` gets it directly via `useContext`, regardless of how many components sit between it and the Provider.

## My practice exercise

Goal: create ThemeContext, provide "dark" from App wrapping Page, have Page read and display it via useContext.

**Final correct code (correct on first attempt, including correctly avoiding the prop-destructuring mistake):**

```jsx
import { useContext, createContext } from "react";
const ThemeContext = createContext();

function App() {
  const theme = { theme: "dark" };
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}

function Page() {
  const themeContext = useContext(ThemeContext);
  return <p>{themeContext.theme}</p>;
}
```

## Key takeaway (in my own words)

useContext solves prop drilling - instead of passing a value down through every component in a chain (even ones that don't need it), a value is provided once via a Provider wrapping part of the component tree, and any descendant component can read it directly via useContext, no matter how deeply nested. Components that only relay the value no longer need to accept or pass it as a prop at all, and the component that actually uses the value stops destructuring it from props entirely, pulling it from context instead.