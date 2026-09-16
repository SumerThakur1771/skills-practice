# Week 5 - Day 4: API Routes

## What an API actually is

API = Application Programming Interface, but the simplest way to think about it: a defined way for one piece of software to ask another piece of software for something, and get a response back.

**Connecting to system design vocab:** an API is a specific URL, on a server, designed to receive a request and send back data - not a page for humans to look at, but a structured response meant to be used by code (frontend JavaScript, another program, etc.) - built directly on the Client-Server model and HTTP already covered.

**A concrete example already used:**
```js
const response = await fetch("https://api.example.com/users/5");
const data = await response.json();
```
Every fetch call in the WHOOP prep, useFetch hook, or UserProfile component was calling an API - the URL fetched from was an API endpoint, and the JSON returned was the API's response.

**Distinction from a regular page:** a page (page.js) is meant to be VIEWED by a human in a browser, showing formatted content. An API endpoint is meant to be USED BY CODE - returns raw data (usually JSON), no visual styling, no HTML page.

## The traditional setup WITHOUT Next.js's all-in-one approach

Two entirely separate projects, each with their own folder, package.json, dependencies:
```
my-frontend/     (a plain React app, e.g. Vite)
  package.json
  src/App.js

my-backend/      (a separate Node/Express project)
  package.json
  server.js
```

Genuinely two separate servers, in two separate terminal windows:
```bash
# Terminal 1 - my-backend folder
npm run start   # starts Express server, e.g. localhost:5000

# Terminal 2 - my-frontend folder
npm run dev     # starts React app, e.g. localhost:3000
```
The frontend would fetch("http://localhost:5000/api/users") - a request ACROSS to a completely different running program on a different port, started and kept running separately.

**Why this is more complicated:** two codebases to maintain, two package.json's, two things to start up when developing, two separate deployments in production, data shapes manually kept in sync across two completely separate codebases.

## What Next.js API routes change concretely

```
my-nextjs-app/          (ONE project)
  package.json
  app/
    page.js              -> frontend page
    api/
      users/
        route.js          -> backend logic, SAME project
```

```bash
# ONE terminal, ONE command
npm run dev    # starts BOTH pages AND API routes
```

Frontend code calls `fetch("/api/users")` - a RELATIVE URL, since it's part of the exact same running application, not a separately-run server.

## route.js vs page.js

`route.js` is a special filename (like page.js) that turns a folder into an API ENDPOINT instead of a visitable page.

```
app/
  about/
    page.js       -> becomes a PAGE at /about
  api/
    hello/
      route.js    -> becomes an API ENDPOINT at /api/hello
```

**Key difference in what each returns:** page.js returns JSX, meant to be rendered visually as HTML for a human. route.js returns raw data (usually JSON), meant to be read/used by code, not displayed as a webpage.

`api/` as a top-level folder name is purely a NAMING CONVENTION to keep API routes organized and visually distinct from pages - not a strict technical requirement.

## Basic API route structure

```jsx
// app/api/hello/route.js
export async function GET() {
  return Response.json({ message: "Hello from the API" });
}
```

**export async function GET()** - instead of a default export (like page.js), API routes export functions NAMED AFTER HTTP METHODS (GET, POST, PUT, DELETE - from the system design vocab HTTP entry). Whichever HTTP method a request uses, Next.js calls the matching named function.

**Response.json({...})** - built-in function that formats data correctly as JSON and sets the right headers for the response.

Visiting `/api/hello` in a browser (or `fetch("/api/hello")`) makes a GET request by default, so Next.js runs the GET function and returns `{ "message": "Hello from the API" }`.

## The folder-to-URL rule applies identically to API routes (Day 2 connection)

Just like page routing, the folder structure directly determines the API's URL:

```
app/
  api/
    hello/
      route.js    -> creates the URL: /api/hello
```

If it were just `app/api/route.js` (no `hello` folder), the URL would be `/api`, not `/api/hello`. The subfolder name is entirely a CHOICE, based on what URL segment is wanted for that specific endpoint - exactly the same rule as naming page folders, no special separate rule for API routes.

**Real example - IronMind's /api/chat:**
```
app/
  api/
    chat/
      route.js    -> /api/chat
```
IronMind's frontend chat interface sends a request to /api/chat; the route.js inside that folder is the backend code that receives it, runs the RAG pipeline (embedding, vector search, AI model call), and sends back the response the frontend displays - the client-server model playing out concretely within one Next.js project.

## Key takeaway (in my own words)

An API is a defined URL that returns raw data for code to use, rather than a page for humans to view. Next.js API routes let backend logic live inside the same project as the frontend, using the special route.js filename (page.js's counterpart) which exports functions named after HTTP methods (GET, POST, etc.) instead of a default JSX-returning function. This eliminates the need for a separate backend project with its own codebase, terminal process, and deployment. The folder-to-URL mapping rule from Day 2 applies identically to API routes - the folder structure inside api/ determines the endpoint's URL, with folder names chosen to match the desired endpoint path.