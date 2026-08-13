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