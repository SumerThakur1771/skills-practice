# Week 5 - Day 1: Next.js - Why It Exists, Server vs Client Components

## What Next.js actually is, and why it exists on top of React

React alone handles building UI with components, but doesn't handle everything a real website needs: which URL shows which page (routing), how the very first HTML gets sent to the browser, or where to put backend/API code. Next.js is a framework built on top of React that adds all of this - routing, server-side rendering, and a place for backend code - in one project.

## What server-side rendering (SSR) actually is

There are two different places "building the page" work can happen: on the SERVER (a remote computer, before anything reaches the user) or on the CLIENT (the browser, after downloading files).

**Server-side rendering** means the actual HTML content of a page gets built ON THE SERVER first, and the browser receives a complete, ready-to-display page immediately.

**Without SSR (typical plain React app):** browser first receives a nearly-empty HTML file (just a `<div id="root"></div>`), downloads a JS bundle, then RUNS that JS to build and insert all the content - a brief period where the page looks blank/incomplete until JS finishes running.

**With SSR (Next.js style):** the server does the "build the content" work before sending anything - browser receives a fully-formed page immediately, with real content visible right away, even before JS has run.

## What "backend" means in this context

Not specifically "linking to a database" - that's just one thing a backend often does. More broadly, backend means any code that runs on the SERVER, handling things the browser itself shouldn't/can't do directly - talking to a database, checking authentication, calling external APIs with secret keys, or calculations that shouldn't be exposed to users.

**Why the browser can't/shouldn't do these directly:** if database credentials or a secret API key were in frontend JS code, anyone could open dev tools and see them - a real security problem. Backend code runs somewhere the user can't inspect.

**What Next.js specifically adds:** traditionally, this required TWO separate projects - a React frontend and a completely separate backend (like Node/Express). Next.js lets both live in ONE project - React components (frontend) and server-side logic (backend, via API routes, covered in a later day) together, in one codebase.

**Real example - IronMind's /api/chat:** backend code living inside the Next.js project that receives a message, does the RAG lookup (touching a database), calls an external AI API (using a secret key), and sends back a response. None of that logic - database queries, secret key - ever gets sent to the browser; only the final response does.

## Server Components vs Client Components - the App Router's core distinction

**The problem this split solves:** the App Router makes it an explicit, deliberate choice, per component, whether something runs only on the server or needs browser interactivity - clearer than the older Pages Router system.

**Server Components (the default):** run ONLY on the server, as part of SSR. Since they never run in the browser, they can safely do backend-style things directly (like querying a database) right inside the component, with zero risk of exposing secrets, since none of that code ships to the browser.

**Client Components:** actually get sent to the browser as real, running JavaScript - specifically needed for useState, useEffect, click handlers, or any interactivity that only makes sense once a real user is actively using the page.

**The decision rule:** ask "does this specific piece of UI need to react to user interaction, or manage changing state?" If yes -> Client Component (`"use client"`). If no (just displaying content, maybe pulling from a database) -> leave it as a Server Component (default, no directive needed).

## Building up the syntax, piece by piece

**Piece 1 - import (same as TypeScript Day 11 modules, just importing a component):**
```jsx
import Counter from "./Counter";
```
Another file called Counter (same folder, since it starts with ./) has a default export - bring it into this file, referred to as Counter.

**Piece 2 - why components live in separate files:** so far, multiple components were often written in one file for practice. In real projects, each component usually gets its own file, imported wherever needed - purely for organization, nothing conceptually new about components themselves.

**Piece 3 - the special `page.js` filename:** in the App Router, a file literally named `page.js` (or .jsx/.tsx) inside the `app/` folder has special meaning - Next.js automatically treats it as the actual page shown at that folder's URL. No routing code is written manually - the file's name and location IS the routing.

```
app/
  page.js  -> this becomes the homepage, at "/"
```

**Piece 4 - the smallest working example (no imports yet):**
```jsx
// app/page.js
export default function HomePage() {
  return <h1>Welcome to my page</h1>;
}
```
Just a normal component, already known, placed in a specially-named file.

**Piece 5 - adding a second, separate component file:**
```jsx
// app/Greeting.js
export default function Greeting() {
  return <p>Hello there!</p>;
}
```
```jsx
// app/page.js
import Greeting from "./Greeting";

export default function HomePage() {
  return (
    <div>
      <h1>Welcome to my page</h1>
      <Greeting />
    </div>
  );
}
```
Same "component using another component" pattern from React Day 1 (App rendering Welcome), just physically split into two files this time, connected via import/export default.

**Piece 6 - adding "use client" for interactivity:**
```jsx
// app/Greeting.js
"use client";

import { useState } from "react";

export default function Greeting() {
  const [liked, setLiked] = useState(false);

  return (
    <div>
      <p>Hello there!</p>
      <button onClick={() => setLiked(true)}>
        {liked ? "Liked!" : "Like"}
      </button>
    </div>
  );
}
```
page.js does NOT need to change at all - still just imports and uses Greeting, completely unaware of whether Greeting is a Server or Client Component underneath.

## What "use client" actually does, mechanically

Server Components run only on the server, and their whole output is just plain HTML sent to the browser - no JavaScript logic from that component ships along with it. But useState, onClick, and interactivity genuinely require real JavaScript running LIVE in the browser - a button can't "remember" it was clicked using only static HTML.

Without "use client", Next.js assumes a file is a Server Component by default and tries to run it only on the server, producing static HTML. If that file used useState under these assumptions, it would fail, since useState fundamentally needs to exist and run in a live browser environment.

"use client" tells Next.js: don't just render this once on the server and throw away the JavaScript - actually bundle this component's JavaScript and send it to the browser, so it can run there, live, and respond to events like clicks.

**The simplest mental model:** "use client" is the switch that decides whether a component's code gets thrown away after producing static HTML on the server (Server Component, default), or gets shipped to the browser to keep running there (Client Component, with "use client").

## Key takeaway (in my own words)

Next.js adds routing, server-side rendering, and backend capability on top of plain React, all in one project. Server-side rendering means the server builds the actual page content before sending it, rather than the browser having to build it after receiving mostly-empty HTML. Backend code is server-only code that handles things too sensitive or complex to expose to the browser, like database queries or secret API keys. The App Router's Server/Client Component split makes it explicit per-component whether something runs only on the server (default, safe for backend-style work) or needs to ship real JavaScript to the browser via "use client" (needed for hooks and interactivity) - other files that just import and render a Client Component don't need the directive themselves.