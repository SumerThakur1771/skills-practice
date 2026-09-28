# Week 5 - Day 5: Middleware

## The problem middleware solves

Some checks need to happen BEFORE a request even reaches a page or API route - the most common one being "is this user logged in?" Without middleware, you'd have to manually repeat that check inside every single protected page/route. Middleware lets you write that check ONCE, and Next.js automatically runs it in front of every matching request before it reaches its destination.

## Where the file lives

`middleware.js` goes at the ROOT of the project - same level as the `app/` folder, NOT inside it.

```
my-project/
  app/
  middleware.js   <- here, next to app/, not inside it
  package.json
```

## The `request` parameter

```js
export function middleware(request) {
  // ...
}
```

Next.js automatically calls this function for you on every matching request, and automatically hands it a `request` object - similar to how a browser automatically passes an `event` object into an event listener callback. You don't create `request` yourself; Next.js builds it and gives it to you, containing details about the incoming request (its URL, its cookies, etc).

## What a cookie is

Think of it like a wristband at an event: you show ID once at the entrance, they snap a wristband on you, and after that you just show the wristband - no need to show ID again at every checkpoint inside.

A cookie works the same way: the server sets it once, the browser stores it, and the browser then automatically attaches that cookie to every future request to that same site - no extra work needed on each request.

## IronMind's real auth flow (the running example)

1. User logs in with email/password.
2. Server verifies the credentials, creates a JWT (a signed token proving who the user is).
3. Server sends that JWT back as an HttpOnly cookie (a cookie regular JS can't read, only the browser + server can).
4. From then on, the browser automatically attaches that cookie to every request to IronMind - including page visits.
5. Middleware can check for that cookie on every request, before the page loads, and decide whether to let the user through or redirect them to `/login`.

## The middleware code

```js
import { NextResponse } from "next/server";

export function middleware(request) {
  const token = request.cookies.get("token");

  if (!token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}
```

## Breaking down each piece

**`request.cookies.get("token")`** - `"token"` here is just the NAME/LABEL the cookie was stored under (a string) - it's how you look the cookie up. The `token` variable on the left is different - it's whatever `.get("token")` actually returns: the real cookie object (containing the actual JWT value), or `undefined` if no cookie with that name exists.

**`if (!token)`** - if no cookie was found (`token` is `undefined`), the user isn't logged in.

**`NextResponse`** - a helper object provided by Next.js itself, imported from `"next/server"`. It gives you ready-made methods for controlling what happens to the request:
- `NextResponse.redirect(url)` - sends the user to a different URL instead of the page they asked for.
- `NextResponse.next()` - means "everything's fine, let the request continue on to wherever it was originally headed."

**`new URL(path, base)`** - builds a complete, valid URL out of a relative path plus a base domain. `request.url` gives the full URL of the page the user was originally trying to visit (e.g. `https://ironmind-psi.vercel.app/dashboard`), which is used as the `base`. So:

```js
new URL("/login", request.url)
// request.url = "https://ironmind-psi.vercel.app/dashboard"
// result      = "https://ironmind-psi.vercel.app/login"
```

`request.url` is needed specifically to supply the correct domain - `"/login"` alone is just a path, not a full URL, so `new URL` needs something to attach it to.

## Doubts / Questions I had

**Q: Is `"token"` the same thing as the `token` variable?**

A: No - `"token"` (in quotes) is just the cookie's NAME, used to look it up. `token` (the variable, no quotes) is whatever `.get("token")` actually returns - the real cookie data, including the JWT value, or `undefined` if that cookie doesn't exist. Same word, two different things: one is a lookup key (string), the other is a variable holding whatever was found.

**Q: What is `NextResponse`, and where does it come from?**

A: It's an object provided by Next.js itself (not something you build), imported from `"next/server"`. It bundles together the common things middleware needs to do with a request - specifically `.redirect()` to send the user elsewhere, and `.next()` to let the request continue normally.

**Q: Why do we need `new URL(path, base)` instead of just redirecting to `"/login"` directly?**

A: A redirect needs a complete, absolute URL (protocol + domain + path), not just a bare path. `new URL("/login", request.url)` combines the path `"/login"` with the current domain (pulled from `request.url`, which holds the full URL of the page being visited) to build that complete URL. Traced example: if `request.url` is `https://ironmind-psi.vercel.app/dashboard`, then `new URL("/login", request.url)` produces `https://ironmind-psi.vercel.app/login`.

## Key takeaway (in my own words)

Middleware runs before a request reaches its destination, letting you check things like "is this user logged in?" in one central place instead of repeating that check in every page. It reads the incoming request's cookies to check for a valid auth token, and uses `NextResponse.redirect()` (built with `new URL(path, request.url)` to form a complete URL) to send unauthenticated users to `/login`, or `NextResponse.next()` to let authenticated users continue through normally.