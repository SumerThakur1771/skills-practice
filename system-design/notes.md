# System Design Vocabulary Log

Daily 15-minute vocabulary terms, building foundational system design language before full design sessions start (Week 5 onward) and the dedicated Week 9 deep-dive.

---

## Client-Server Model

**WHY it exists:** solves the problem of how many different devices (phones, laptops, browsers) all get access to the same shared data and logic, reliably, without each device having to store and compute everything itself.

**The core roles:**
- **Client** - the device/program that INITIATES a request. What the user directly interacts with (browser, mobile app, desktop app). Presents information, collects input, asks the server for what it needs.
- **Server** - a separate, typically remote, machine that LISTENS for requests and responds to them. Holds the actual data (via a database), runs business logic, enforces system rules. Doesn't act until a client asks it to.

**The relationship:** a request-response relationship. The client always INITIATES - the server never randomly pushes data without being asked first (exceptions like WebSockets/push notifications exist, but this is the base model).

```
Client (browser/app)  --- request --->  Server
Client (browser/app)  <--- response ---  Server
```

**Concrete example - Velzee Insights screen:**
1. Client (React Native app) sends a request: "give me this user's spending insights data."
2. Server (Velzee backend + Supabase) receives it, queries the database, runs logic (e.g. spending pace calculation).
3. Server sends a response back - actual data, usually as JSON.
4. Client receives the response and renders it on screen.

**Same pattern in IronMind:** Next.js frontend (client) sends a request to /api/chat (server-side route in the same Next.js app), server runs the RAG pipeline, sends back a streamed response the client displays.

**Why the separation actually matters:**
- Centralized data - many different clients can talk to the same server and get consistent, up-to-date data, rather than each device having a disconnected copy.
- Security/control - sensitive logic and data (e.g. Smart Categories' PL/pgSQL trigger logic, API keys) lives on the server, never exposed directly to the client where it could be inspected or tampered with.
- Scalability - the server can be upgraded, scaled, or replaced without needing to touch every client device.

**Interview relevance:** often the literal opening question of a system design interview, or implicitly assumed as the foundation before any specific system gets discussed. Base vocabulary everything else (load balancers, caching, CDNs) builds on top of.

---

## HTTP (HyperText Transfer Protocol)

**WHY it exists:** once a client and server exist, they need a shared, agreed-upon language to communicate - a set of rules for how requests/responses are formatted, what action is being requested, and how to signal success or failure. HTTP provides that.

**HTTP Methods - what kind of action the client wants:**
- GET - retrieve data (e.g. "give me this user's profile")
- POST - create new data (e.g. "create a new account")
- PUT/PATCH - update existing data
- DELETE - remove data

**Status codes - the server's way of telling the client what happened:**
- 200 - success
- 201 - created successfully
- 400 - bad request (client sent something wrong)
- 401 - unauthorized (not logged in / invalid credentials)
- 404 - not found
- 500 - server error (something broke server-side)

**Connecting to real code:** every API route touched (Velzee's Supabase queries, IronMind's /api/chat, /api/auth/login) is built on this - client sends an HTTP request with a specific method (POST for login, GET for fetching data), server responds with a status code plus a body (usually JSON).

---