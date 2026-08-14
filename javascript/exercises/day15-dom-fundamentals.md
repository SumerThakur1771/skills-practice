# DOM Fundamentals (Added Before Week 4 React)

## What the DOM actually is

DOM = Document Object Model. When a browser loads an HTML page, it builds an actual LIVE, in-memory tree of objects representing every element on the page. This tree is what JavaScript can read from and write to in real time, while the page is running.

```html
<body>
  <h1>Hello</h1>
  <p>World</p>
</body>
```

Becomes roughly:
```
document
  └── body
        ├── h1 ("Hello")
        └── p ("World")
```

This isn't just a description of the page - it's the actual, live thing being rendered on screen. Changing the DOM immediately reflects visually, no reload/re-parse needed.

**Why the distinction matters:** HTML is the initial blueprint - text describing what to build. The DOM is the actual built structure, sitting in memory, that JavaScript interacts with after the page loads.

## Selecting elements

```js
document.querySelector("h1");        // first matching element
document.querySelector(".card");     // first element with class "card"
document.querySelector("#title");    // element with id "title"
document.querySelectorAll(".card");  // ALL matching elements, as a list (NodeList)
```

## Reading and changing content

```js
const heading = document.querySelector("h1");
console.log(heading.textContent); // reads current text
heading.textContent = "Goodbye";  // changes what's displayed, immediately
```

## Changing styles directly

```js
heading.style.color = "blue";
heading.style.fontSize = "24px";
```

CSS properties with hyphens (font-size) become camelCase in JS (fontSize) - same naming convention shift seen elsewhere in JS.

## Event listeners

```js
const button = document.querySelector("button");
button.addEventListener("click", () => {
  console.log("Button was clicked!");
});
```

`addEventListener(eventName, callback)` - same callback pattern as array methods (Day 3). Event name ("click", "submit", "input", etc.) plus a callback the browser calls whenever that event happens.

## WHY manual DOM manipulation gets painful - the real motivation for React

**Realistic scenario - a counter:**
```html
<p id="count">0</p>
<button id="increment">+1</button>
```
```js
let count = 0;
const countDisplay = document.querySelector("#count");
const button = document.querySelector("#increment");

button.addEventListener("click", () => {
  count = count + 1;
  countDisplay.textContent = count; // manually keeping DOM in sync with data
});
```

Works fine for one small piece of UI, but the core pattern: the DEVELOPER is personally responsible for remembering to update the DOM every time the data changes. `count` and what's displayed are two SEPARATE things manually kept in sync.

**Why this doesn't scale:** a real app has dozens of data points, each potentially affecting multiple UI locations (e.g. a username in a header, sidebar, and welcome message all at once). Every place that data appears needs its own manual selector + update logic. Miss one, and the UI silently shows stale data - a common, hard-to-track bug class in apps built this way.

**This is exactly the problem React solves:** React lets the UI be described declaratively based on current data, and React itself figures out which parts of the actual DOM need to change and updates them - no more manually writing querySelector + .textContent update logic scattered everywhere.

## Practice scenario - same value in multiple places

If the same `count` value is displayed in 3 different places (header, sidebar, footer):

**Selecting** can be done in one line if elements share a class:
```js
const allCountDisplays = document.querySelectorAll(".count-display");
```

**But updating still requires touching each one individually** - querySelectorAll returns a LIST (NodeList), not a single element, so `.textContent =` only works on one at a time:
```js
button.addEventListener("click", () => {
  count = count + 1;
  allCountDisplays.forEach((el) => {
    el.textContent = count; // still conceptually happening 3 times, just via a loop
  });
});
```

**The deeper problem this reveals:** even with cleaner selecting, the developer is still manually remembering to write update logic everywhere data could appear. With many unrelated data points across dozens of components, this breaks down - not a line-count problem, but a cognitive burden of manually tracking every sync point, which is what React actually eliminates.

## Doubts / Questions I had

**Q: Could .map be used instead of .forEach for updating multiple DOM elements?**

A: Technically runs, but is the WRONG tool. `.map` builds and returns a NEW ARRAY based on each item's return value (Day 3). `.forEach` just runs a callback on each item and returns nothing. Updating the DOM is a side effect, not building a new collection of transformed data - there's no meaningful "returned array" to use afterward.
```js
// Works, but wrong-tool usage - creates and discards a throwaway array of undefined
allCountDisplays.map((el) => {
  el.textContent = count;
});
```
`forEach` is semantically correct because it's a side effect (DOM update) on each item, not a data transformation. Using `.map` when meaning `.forEach` is a code smell interviewers may flag - signals not fully understanding why each method exists, even if it technically runs.

**Q: What's the difference between .textContent, .innerText, and .innerHTML?**

A: Three genuinely different behaviors:
- `.textContent` - gets/sets raw text content, includes hidden elements, does NOT parse HTML (a string like "<b>bold</b>" displays literally, tags and all).
- `.innerText` - similar to textContent, but "aware" of CSS styling - excludes text hidden via CSS (display: none), can trigger a layout recalculation (slightly more performance-costly) since it checks what's actually visible.
- `.innerHTML` - gets/sets content and ACTUALLY PARSES it as real HTML - a string like "<b>bold</b>" renders as genuinely bold text, creating real new DOM elements from that string.

```js
element.textContent = "<b>Hi</b>"; // displays literal text: <b>Hi</b>
element.innerHTML = "<b>Hi</b>";   // displays actual bold: Hi (rendered as HTML)
```

**Why .innerHTML is often avoided:** setting it with user-provided input is a genuine security risk - classic XSS (cross-site scripting) vulnerability, since malicious HTML/JS could be injected and actually executed. `.textContent` is the safer default unless real HTML markup genuinely needs to be rendered.

## Key takeaway (in my own words)

The DOM is the live, in-memory tree the browser builds from HTML, which JavaScript can read and modify in real time via methods like querySelector, textContent, style, and addEventListener. Manually keeping this tree in sync with changing data is repetitive and error-prone as an app grows, since every place data appears needs its own manual update logic - this exact pain point is why React exists, letting the UI be described declaratively while React handles the actual DOM updates.