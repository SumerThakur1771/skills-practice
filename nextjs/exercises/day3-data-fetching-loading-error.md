# Week 5 - Day 3: Data Fetching in Server Components, loading.js, error.js

## The core idea - fetching data directly in a Server Component

Server Components run only on the server, and can safely do backend-style things directly (Day 1). This means data can be fetched DIRECTLY inside the component itself, using async/await - something never possible in a regular Client Component before.

```jsx
export default async function UserPage({ params }) {
  const response = await fetch(`https://api.example.com/users/${params.id}`);
  const user = await response.json();

  return <p>{user.name}</p>;
}
```

The component function itself is `async` - only possible because Server Components run on the server, where waiting for a fetch to complete before sending the finished HTML is completely normal and expected.

**Connection to React's useFetch work:** in a Client Component, the component function itself could never be async, and useEffect's own function specifically couldn't be async either (must return nothing or a cleanup function, never a Promise) - which is why a separate async function had to be defined INSIDE the effect and called there. A Server Component skips this workaround entirely since the component function itself can just be async, directly.

## loading.js - automatic loading state, no manual state management

**The problem:** while `await fetch(...)` is happening, the page hasn't finished rendering yet. In a Client Component, a `loading` state would be tracked manually (like the UserProfile build). Server Components have no useState at all.

**The answer - a special file:**
```
app/
  users/
    [id]/
      page.js
      loading.js
```

```jsx
// app/users/[id]/loading.js
export default function Loading() {
  return <p>Loading...</p>;
}
```

Just like `page.js`, `loading.js` is a special, recognized filename - Next.js automatically shows whatever this file renders WHILE the corresponding page.js is still fetching its data, then automatically swaps to the real page once ready. No logic connects them manually - just creating this specially-named file is enough.

## error.js - automatic error handling

**The problem:** if `await fetch(...)` fails, or the response indicates an error, a Server Component would normally just crash, showing a broken, generic error page.

**The answer - another special file:**
```
app/
  users/
    [id]/
      page.js
      loading.js
      error.js
```

```jsx
"use client";

export default function Error({ error }) {
  return <p>Something went wrong: {error.message}</p>;
}
```

Next.js automatically shows whatever error.js renders if anything inside the matching page.js throws an error - no manual try/catch needed inside the page itself.

## Why error.js specifically MUST be a Client Component

React has a built-in safety mechanism: if something crashes while rendering, instead of the whole app breaking completely, React can catch that crash and show a fallback UI instead. This safety mechanism ("error boundaries") only works using tools that specifically require running in the BROWSER, not on the server - Server Components are only designed to build static HTML and hand it off, with no "catch a crash and recover gracefully" capability at all.

**The simple rule:** error.js's whole job is this "catch a crash and show a fallback" behavior, and since that mechanism only exists in the browser, error.js has no choice but to be a Client Component, every single time, no exceptions.

## Why error.js receives {error} while page.js receives {params} - different props for different jobs

`params` and `error` are two completely unrelated props, matched to what each specific file actually needs to do its job:
- `page.js` needs to know "what URL segment am I showing" -> receives `params` (the dynamic route values)
- `error.js` needs to know "what exactly broke" -> receives `error` (the actual error object thrown somewhere inside the matching page.js)
- `loading.js` doesn't need a prop at all, since showing a loading message requires no information

These aren't alternate names for the same concept - they're genuinely different pieces of information, each matched to that specific special file's purpose.

## The full picture - three files working together automatically

```
app/users/[id]/
  page.js      -> the actual data-fetching page (Server Component, can be async)
  loading.js   -> shown automatically while page.js is fetching
  error.js     -> shown automatically if page.js throws an error (must be Client Component)
```

## Key takeaway (in my own words)

Server Components can be async functions themselves, allowing data to be fetched directly inside the component before it renders, without the useEffect-wrapping-an-async-function workaround required in Client Components. Next.js provides two special, automatically-recognized files per route: loading.js, shown while the matching page.js is still fetching data, and error.js, shown if the matching page.js throws an error - both work with zero manual state management or try/catch needed in the page itself. error.js must always be a Client Component specifically because catching and recovering from a crash is a browser-only React capability that Server Components don't have. Each special file receives whatever prop matches its specific job - params for the URL's dynamic segments, error for what went wrong.