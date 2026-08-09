# Week 2 - Day 10: The Event Loop

## WHY this matters

JavaScript is single-threaded - it can only do one thing at a time. But it handles timers, network requests, and Promises without freezing the whole program while waiting. The Event Loop is the mechanism that makes this possible. One of the most commonly asked "explain what's happening under the hood" interview questions, often paired with a "predict the output" code snippet.

## The kitchen analogy

JS is like a single chef who can only cook one dish at a time, standing at one station (the **Call Stack** - where actual code execution happens, one thing after another, top to bottom).

If an order requires something slow (like "marinate this meat for 2 hours"), the chef doesn't stand there waiting. They hand it off to a slow cooker in the back (**Web APIs** - the browser's or Node's own built-in capabilities that handle waiting/background work, NOT an external "backend" server) and immediately goes back to cooking other orders.

Once the slow task finishes, it doesn't jump straight back into the chef's hands - it gets placed on a **pickup counter** (**Callback Queue** - a waiting line of finished background work, ready whenever the chef is free).

The chef only checks the pickup counter once their station is COMPLETELY empty (**Call Stack is empty**) - never interrupts an active task to go grab something from the counter.

**The Event Loop** = the name for this repeated action: "Is my station empty? If yes, check the counter for anything ready, and if so, bring it to the station and work on it."

## The VIP counter - Microtask Queue

There are actually TWO counters, not one - and one is higher priority.

- Promises (.then, .catch, async/await continuations) go into a separate, higher-priority counter: the **Microtask Queue**.
- Things like setTimeout go into the regular **Callback Queue**.

**The rule:** every single time the call stack becomes empty, the event loop checks the Microtask Queue (VIP counter) FIRST and completely empties it - before ever touching the regular Callback Queue, even if something's been sitting there longer.

This is why Promises tend to run before setTimeout, even if the setTimeout was scheduled with 0ms delay.

## The two categories of code - what actually goes in a queue

**Category 1: Normal, synchronous code** - console.log, math, variable assignments, normal function calls. Runs DIRECTLY on the Call Stack, immediately, top to bottom. Never touches any queue at all.

**Category 2: Async operations** - setTimeout, Promises, network requests. Only THESE get handed to the background, and only their callback functions (the function to run after they finish) end up in a queue (Callback Queue or Microtask Queue depending on type).

Plain synchronous code never goes into any queue - it just executes immediately as JS reads through the code. The queues are exclusively for callback functions of things that had to wait somewhere else first.

## Traced example - solved correctly myself

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
```

Step by step:
- `console.log("A")` runs immediately on the call stack -> logs A
- `setTimeout(..., 0)` handed to background, callback goes to the regular Callback Queue once ready
- `Promise.resolve().then(...)` - callback goes to the Microtask Queue (VIP)
- `console.log("D")` runs immediately -> logs D
- Call stack now empty. Event loop checks Microtask Queue FIRST -> runs it -> logs C
- Microtask Queue now empty. Event loop checks regular Callback Queue -> runs it -> logs B

**Output: A, D, C, B**

## Doubts / Questions I had

**Q: What exactly IS "the event loop" - I don't see any code syntax for it, where is it?**

A: There's no line of code written called `eventLoop()` - never directly interacted with. The event loop is just the name for the constant background process the JS engine runs, repeatedly asking: "Is the call stack empty? If yes, check the microtask queue first - run everything there. Then check the callback queue - run the next item." It's a loop (literally repeats forever) that decides WHEN queued-up async callbacks actually get executed. Invisible machinery, always running underneath the code:

```js
console.log("Start");
setTimeout(() => { console.log("Timeout callback"); }, 0);
Promise.resolve().then(() => { console.log("Promise callback"); });
console.log("End");
```
None of this code explicitly mentions "event loop" anywhere - it's the process that decides WHEN each queued callback actually gets pulled and run, not something written directly.

**Q: Why do Promise/async-await go to the Microtask Queue while setTimeout goes to the regular Callback Queue - why the different treatment?**

A: The real distinction is "microtask" vs "macrotask," based on how urgent/lightweight the result is expected to be.

Microtasks (Promises, async/await) = high-priority, should-run-ASAP work, because a resolved Promise represents "I already have the answer, I just need to hand it off" - the spec designers wanted that handoff to happen immediately, before the browser does anything else (like rendering or running a timer).

Macrotasks (setTimeout, setInterval, DOM events) = lower-priority, can-wait-its-turn work, since setTimeout is inherently about SCHEDULING something for later, not competing for immediate priority.

Practical consequence: if Promises shared the same queue as setTimeout callbacks, pending timers could delay already-finished async data from being processed, even though that data was ready right now. Giving Promises their own faster lane avoids that unnecessary delay.

Interview-ready one-liner: "Promises are treated as microtasks because they represent work that just finished and should be handled immediately, while setTimeout is a macrotask because it's explicitly about waiting/scheduling for later - so the spec gives Promises a faster, priority queue that runs before the regular one."

**Q: Does JS's own fast, synchronous code get added to the callback queue too?**

A: No. Regular synchronous code (console.log, math, variable assignments, normal function calls) runs directly on the Call Stack immediately, top to bottom - it never touches any queue at all. Only async operations (setTimeout, Promises, network requests) get handed to the background, and only THEIR callback functions end up in a queue once finished, waiting for their turn to run.

## Key takeaway (in my own words)

JS is single-threaded and can only do one task at a time on the call stack. Tasks that require time to produce output get handed over to the Web APIs (browser/Node background) to process in parallel, while JS keeps executing other code on the call stack. Once the background task completes, its callback gets added to a queue - Promises go to the higher-priority Microtask Queue, things like setTimeout go to the regular Callback Queue. The event loop only pulls from these queues once the call stack is completely empty, always draining the Microtask Queue first before touching the Callback Queue.