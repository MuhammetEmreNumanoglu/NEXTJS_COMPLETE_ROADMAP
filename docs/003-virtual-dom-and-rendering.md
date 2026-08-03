# 003 — Virtual DOM and Rendering

---

## Purpose of This Topic

This topic explains one of React's most important internal mechanisms.  
Understanding the Virtual DOM is key to understanding why React is fast,  
why re-renders happen, and how to optimize your components later.

This knowledge will directly help you when you debug performance issues  
in real applications — and performance topics come up in almost every  
senior-level interview.

---

## Why Does This Topic Exist?

When React was created, the biggest performance problem in web development  
was manipulating the real DOM.

The DOM (Document Object Model) is the browser's representation of an HTML page.  
Every time you change something — add a class, update text, hide an element —  
the browser does a lot of expensive work behind the scenes.

React's inventors asked: **What if we didn't touch the real DOM unless we had to?**  
The answer was the Virtual DOM.

---

## Where Is This Used in Real Projects?

You don't directly interact with the Virtual DOM in your code.  
React manages it for you automatically.

But understanding it matters because:

- When you write components, you need to know **when** and **why** they re-render
- When your app is slow, you need to know **what triggers unnecessary re-renders**
- When you use `React.memo`, `useMemo`, and `useCallback`, these are all tools to  
  **control re-rendering** — and you need to understand rendering to use them correctly

---

## Real-Life Analogy

Imagine you're an editor at a newspaper.

```
Without Virtual DOM (Old approach):
┌────────────────────────────────────────────┐
│ Every time one word changes in an article, │
│ you reprint the entire newspaper.          │
│ Even if only the headline changed,         │
│ every single page gets reprinted.          │
│ Expensive. Slow. Wasteful.                 │
└────────────────────────────────────────────┘

With Virtual DOM (React approach):
┌────────────────────────────────────────────┐
│ You first make your edits on a draft copy. │
│ Then you compare the draft with the        │
│ original and circle only the differences.  │
│ Only the circled parts go to the printer.  │
│ Fast. Efficient. Smart.                    │
└────────────────────────────────────────────┘
```

The "draft copy" is the Virtual DOM.  
The "comparing and circling differences" is Diffing.  
The "sending only the circled parts to the printer" is Reconciliation.

---

## Simple Explanation

React maintains two trees:

1. **The current Virtual DOM** — what's on screen right now
2. **The new Virtual DOM** — what should be on screen after state changes

When something changes, React:
1. Creates the new Virtual DOM
2. Compares it with the current one (this is called **Diffing**)
3. Finds only the differences
4. Updates only those parts in the real DOM (this is called **Patching** or **Committing**)

This approach is significantly faster than rebuilding the entire real DOM from scratch.

---

## Technical Explanation

### What Is the DOM?

The DOM is a tree structure the browser builds from your HTML:

```
Document
└── html
    ├── head
    │   ├── title
    │   └── meta
    └── body
        ├── header
        │   └── nav
        │       ├── a (Home)
        │       ├── a (About)
        │       └── a (Contact)
        └── main
            ├── h1 (Welcome)
            └── p  (Lorem ipsum...)
```

Each element in this tree is a **DOM node**.  
Manipulating DOM nodes (reading, updating, deleting) is **slow** — because  
the browser has to recalculate layouts, repaint pixels, and more.

### What Is the Virtual DOM?

The Virtual DOM is a JavaScript object that mirrors the DOM tree.  
It lives in memory — not in the browser.

```
Virtual DOM (JavaScript object):
{
  type: 'div',
  props: { className: 'app' },
  children: [
    {
      type: 'h1',
      props: {},
      children: ['Welcome']
    },
    {
      type: 'p',
      props: { className: 'description' },
      children: ['This is a React app.']
    }
  ]
}
```

Operations on JavaScript objects are **extremely fast** compared to real DOM operations.  
That's the key insight behind the Virtual DOM.

### The Render Cycle

```
Step 1: Initial Render
─────────────────────────────────────────────────
Your component returns JSX
       │
       ▼
React creates Virtual DOM from JSX
       │
       ▼
React converts Virtual DOM → real DOM
       │
       ▼
Browser paints the page
       │
       ▼
User sees the page ✓


Step 2: State Change
─────────────────────────────────────────────────
User triggers an event (click, input, etc.)
       │
       ▼
State updates (via setState / useState setter)
       │
       ▼
Component function runs again (re-render)
       │
       ▼
New Virtual DOM is created
       │
       ▼
New VDOM is diffed against old VDOM
       │
       ▼
Only changed nodes are updated in real DOM
       │
       ▼
Browser repaints only those parts
       │
       ▼
User sees the update ✓
```

---

## Diffing Algorithm

React's diffing algorithm makes two key assumptions to be fast:

### Assumption 1: Different Types = Rebuild

If the root element type changes, React tears down the old tree and builds a new one.

```tsx
// Before:
<div>
  <Counter />
</div>

// After:
<section>
  <Counter />
</section>
```

Because `div` changed to `section`, React destroys everything including `Counter`  
and creates a fresh tree. The Counter's state is lost.

### Assumption 2: Same Type = Update in Place

If the root element type stays the same, React updates its props.

```tsx
// Before:
<div className="card" style={{ color: "blue" }}>
  Hello
</div>

// After:
<div className="card" style={{ color: "red" }}>
  Hello
</div>
```

React keeps the `div`, just updates `style.color` from `blue` to `red`.  
Fast. Minimal DOM mutation.

### Keys in Lists

When React renders a list, it needs a way to identify each item across renders.  
This is what the `key` prop is for.

```tsx
// Without keys — React can't tell which item changed:
<ul>
  <li>Apple</li>   // index 0
  <li>Banana</li>  // index 1
  <li>Cherry</li>  // index 2
</ul>

// If "Apple" is removed:
<ul>
  <li>Banana</li>  // now index 0 — React thinks this was updated, not moved
  <li>Cherry</li>  // now index 1 — same problem
</ul>
// React unnecessarily updates every remaining item.
```

```tsx
// With stable keys — React knows exactly what happened:
<ul>
  <li key="fruit-apple">Apple</li>
  <li key="fruit-banana">Banana</li>
  <li key="fruit-cherry">Cherry</li>
</ul>

// If "Apple" is removed:
<ul>
  <li key="fruit-banana">Banana</li>  // React: "banana is still here, unchanged"
  <li key="fruit-cherry">Cherry</li>  // React: "cherry is still here, unchanged"
</ul>
// React removes only the apple node. Zero unnecessary updates.
```

---

## What Is Rendering?

**Rendering** means React calling your component function to get JSX output.

```tsx
function ProductCard({ name, price }: { name: string; price: number }) {
  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p>{price} USD</p>
    </div>
  );
}
```

Every time React calls `ProductCard()`, that's a **render**.  
The function runs. It returns JSX. React processes the output.

### Initial Render vs Re-render

```
Initial Render:
  - Happens once when a component first appears on screen
  - Creates the real DOM nodes
  - User sees the component for the first time

Re-render:
  - Happens when state or props change
  - Runs the component function again
  - React diffs and patches only the changed parts
```

### What Triggers a Re-render?

```
1. Component's own state changes     → useState, useReducer
2. Parent component re-renders       → props may have changed
3. Context value changes             → useContext
4. Custom hook's internal state changes
```

---

## The Full Rendering Pipeline

```
┌─────────────────────────────────────────────────────┐
│                   REACT RENDERING                    │
└─────────────────────────────────────────────────────┘

Phase 1: RENDER PHASE (Pure, no side effects)
─────────────────────────────────────────────
  Component functions run
         │
         ▼
  New Virtual DOM is built
         │
         ▼
  Diffing happens
         │
         ▼
  List of changes is calculated

Phase 2: COMMIT PHASE (Side effects allowed)
─────────────────────────────────────────────
  React applies changes to real DOM
         │
         ▼
  useLayoutEffect runs (synchronously)
         │
         ▼
  Browser paints the screen
         │
         ▼
  useEffect runs (asynchronously)
```

The Render Phase is **pure** — React may run it multiple times or skip it.  
The Commit Phase happens **once** — this is where the DOM actually changes.

---

## React 18: Concurrent Rendering

React 18 introduced **Concurrent Mode** — a major upgrade to how rendering works.

### The Problem Before React 18

```
Before React 18:
  Rendering was synchronous and blocking.
  
  If you had a huge list with 10,000 items:
  
  React starts rendering → 500ms of work
         │
         │  ← User clicks a button
         │  ← User types in an input
         │  ← Browser can't respond!
         ▼
  React finishes → UI finally updates
  
  The user experiences a frozen interface for 500ms.
```

### The Solution: Concurrent Mode

```
React 18 Concurrent Mode:
  Rendering is interruptible.
  
  React starts rendering the huge list
         │
         │  ← User clicks a button
         │
  React PAUSES the list rendering
  React handles the urgent button click first
  React RESUMES the list rendering
         │
         ▼
  User never notices a freeze
```

You get this behavior with the `startTransition` API and hooks like `useTransition`:

```tsx
import { useState, useTransition } from "react";

type SearchResult = {
  id: string;
  title: string;
};

function SearchPage() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<SearchResult[]>([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (inputValue: string) => {
    setQuery(inputValue);

    startTransition(() => {
      const filteredResults = performExpensiveSearch(inputValue);
      setResults(filteredResults);
    });
  };

  return (
    <div>
      <input
        value={query}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      {isPending && <p>Loading results...</p>}
      <ResultsList results={results} />
    </div>
  );
}

function performExpensiveSearch(query: string): SearchResult[] {
  return [];
}

function ResultsList({ results }: { results: SearchResult[] }) {
  return (
    <ul>
      {results.map((result) => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
}
```

**Line-by-line explanation:**

`useTransition()` → Returns `isPending` (is the transition in progress?) and `startTransition` (marks an update as non-urgent).  
`setQuery(inputValue)` → This is urgent — update the input's display immediately.  
`startTransition(() => {...})` → This is non-urgent — do the expensive filtering work, but let React pause it if needed.  
`isPending` → Shows a loading indicator while the transition is in progress.  
The input stays responsive even during heavy computation.

---

## Hydration

In Next.js, you'll frequently encounter the concept of **Hydration**.  
Let's understand it now so it doesn't confuse you later.

```
Server-Side Rendering + Hydration:
────────────────────────────────────────────────────

Step 1: Request arrives at server
  Server runs React components
  Server produces HTML string
  Server sends HTML to browser
         │
         ▼
Step 2: Browser receives HTML
  User can see the page immediately (fast!)
  But the page is "dead" — no JavaScript yet
         │
         ▼
Step 3: Browser downloads JavaScript bundle
  React "hydrates" the page:
  React attaches event listeners to existing HTML
  React makes the page interactive
         │
         ▼
Step 4: Page is fully interactive ✓
```

**Hydration** is the process of React taking over server-rendered HTML  
and making it interactive by attaching event handlers.

A common bug is a **Hydration Mismatch** — when the HTML from the server  
doesn't match what React renders on the client:

```tsx
function BadComponent() {
  return <p>Time: {new Date().toLocaleTimeString()}</p>;
}
```

**Problem:** The server renders the time at 14:30:00. By the time  
the client hydrates (a few hundred milliseconds later), the time is 14:30:01.  
React sees a mismatch and throws a hydration error.

```tsx
"use client";

import { useState, useEffect } from "react";

function GoodComponent() {
  const [time, setTime] = useState<string | null>(null);

  useEffect(() => {
    setTime(new Date().toLocaleTimeString());
  }, []);

  if (time === null) {
    return <p>Loading time...</p>;
  }

  return <p>Time: {time}</p>;
}
```

**Correct:** On the server, `time` is `null`, so "Loading time..." is rendered.  
On the client, after hydration, `useEffect` runs and sets the actual time.  
Server and client output match — no hydration error.

---

## Code Example: Observing Re-renders

This example lets you see re-renders happening in real time:

```tsx
"use client";

import { useState, useRef } from "react";

function RenderCounter({ label }: { label: string }) {
  const renderCount = useRef(0);
  renderCount.current += 1;

  return (
    <div className="render-counter">
      <strong>{label}</strong>
      <span>Rendered {renderCount.current} time(s)</span>
    </div>
  );
}

function ParentComponent() {
  const [parentCount, setParentCount] = useState(0);
  const [unrelatedCount, setUnrelatedCount] = useState(0);

  return (
    <div className="parent">
      <h2>Parent Component</h2>
      <RenderCounter label="Parent" />

      <div className="controls">
        <button onClick={() => setParentCount((prev) => prev + 1)}>
          Parent Count: {parentCount}
        </button>
        <button onClick={() => setUnrelatedCount((prev) => prev + 1)}>
          Unrelated Count: {unrelatedCount}
        </button>
      </div>

      <ChildComponent parentCount={parentCount} />
    </div>
  );
}

type ChildComponentProps = {
  parentCount: number;
};

function ChildComponent({ parentCount }: ChildComponentProps) {
  return (
    <div className="child">
      <h3>Child Component</h3>
      <RenderCounter label="Child" />
      <p>Parent count prop: {parentCount}</p>
    </div>
  );
}
```

**What you'll observe when running this:**

- Clicking "Parent Count" → Parent re-renders AND Child re-renders  
  (because Child receives `parentCount` as a prop, which changed)
- Clicking "Unrelated Count" → Parent re-renders AND Child re-renders  
  (even though `unrelatedCount` has nothing to do with Child — this is React's default behavior)

This "problem" is what `React.memo` solves — you'll learn that in file 064.

---

## React Fiber: The Engine Under the Hood

React's rendering engine is called **Fiber**.  
It was introduced in React 16 as a complete rewrite of the reconciliation algorithm.

```
Before Fiber (React ≤ 15):
  Reconciliation was a recursive, synchronous process.
  Once started, it couldn't be interrupted.
  
After Fiber (React ≥ 16):
  Work is broken into small units called "fibers."
  React can pause, resume, and prioritize work.
  This is what made Concurrent Mode possible.
```

You don't need to write Fiber code directly.  
But knowing it exists helps you understand **why** React can pause renders,  
**why** Suspense works the way it does, and **why** `startTransition` makes sense.

```
Fiber Work Loop:
  ┌─────────────┐
  │  Unit of    │ ← Process this fiber
  │  Work       │
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │  More time  │
  │  available? │
  └──────┬──────┘
         │
    Yes ─┤─ No
         │          │
         ▼          ▼
  Process      Yield to
  next fiber   browser
               (handle clicks,
               inputs, etc.)
               Then resume
```

---

## Common Mistakes

### Mistake 1: Causing Unnecessary Re-renders With Inline Functions

```tsx
function BadParent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <ExpensiveChild onAction={() => console.log("action")} />
      <button onClick={() => setCount((prev) => prev + 1)}>
        Count: {count}
      </button>
    </div>
  );
}
```

**Problem:** Every time `BadParent` re-renders (on every button click),  
a brand new function `() => console.log("action")` is created.  
`ExpensiveChild` receives a new function reference on every render,  
causing it to re-render unnecessarily.

```tsx
import { useCallback } from "react";

function GoodParent() {
  const [count, setCount] = useState(0);

  const handleAction = useCallback(() => {
    console.log("action");
  }, []);

  return (
    <div>
      <ExpensiveChild onAction={handleAction} />
      <button onClick={() => setCount((prev) => prev + 1)}>
        Count: {count}
      </button>
    </div>
  );
}
```

**Better:** `useCallback` memoizes the function. The same reference is passed  
to `ExpensiveChild` on every render, preventing unnecessary re-renders.

---

### Mistake 2: Using Array Index as List Key

```tsx
type Item = { name: string };

function BadList({ items }: { items: Item[] }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item.name}</li>
      ))}
    </ul>
  );
}
```

**Problem:** If items are added, removed, or reordered, the indices shift.  
React sees the wrong items as "the same" and may display stale data or lose  
focus/state in inputs.

```tsx
type Item = { id: string; name: string };

function GoodList({ items }: { items: Item[] }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

**Correct:** Use a unique, stable ID as the key.

---

### Mistake 3: Triggering State Updates During Render

```tsx
function BadComponent({ multiplier }: { multiplier: number }) {
  const [value, setValue] = useState(0);

  setValue(value * multiplier);

  return <p>{value}</p>;
}
```

**Problem:** Calling `setValue` directly in the component body triggers  
a state update during render, which triggers another render, which triggers  
another state update... infinite loop. React will throw an error.

```tsx
import { useEffect } from "react";

function GoodComponent({ multiplier }: { multiplier: number }) {
  const [value, setValue] = useState(0);

  useEffect(() => {
    setValue((prev) => prev * multiplier);
  }, [multiplier]);

  return <p>{value}</p>;
}
```

**Correct:** Use `useEffect` to run side effects after render.

---

## Best Practices

### Understand When Your Component Renders

```
Ask yourself for every component:
  1. What state does it own?
  2. What props does it receive?
  3. Does it use any context?
  4. When any of those change → it re-renders
```

### Keep Render Functions Pure

A component's render output should be determined entirely by its props and state.  
No side effects. No random values. No reading from external mutable variables.

```tsx
let globalCounter = 0;

function BadComponent() {
  globalCounter++;
  return <p>Rendered {globalCounter} times</p>;
}
```

```tsx
function GoodComponent() {
  const renderCount = useRef(0);
  renderCount.current++;
  return <p>Rendered {renderCount.current} times</p>;
}
```

### Lift State Appropriately

Don't put state higher than it needs to be.  
If state is only needed in one component, keep it there.  
If two sibling components need the same state, lift it to their common parent.

```
Bad:
  App (owns userSearchQuery state)
  ├── Header
  ├── Sidebar
  └── SearchResults (the only one that uses userSearchQuery)

Good:
  App
  ├── Header
  ├── Sidebar
  └── SearchResults (owns its own userSearchQuery state)
```

Unnecessary high-level state causes many unnecessary re-renders across the tree.

---

## Real Company Examples

### Netflix: Rendering Priority

Netflix uses concurrent features to prioritize what renders first:

```
High Priority (renders immediately):
  - Playback controls (play/pause)
  - Volume slider
  - Seek bar

Low Priority (renders when idle):
  - Recommendation carousels
  - "More like this" section
  - Metadata overlay
```

If you're watching a video and hit pause, the controls respond instantly.  
The recommendation section updating in the background doesn't block your controls.

### GitHub: Selective Re-rendering

GitHub's code review interface carefully manages re-renders:

```
FileDiff Component
├── FileHeader     (rarely re-renders — filename doesn't change)
├── DiffLines      (re-renders when lines are collapsed/expanded)
└── CommentThread  (re-renders when new comments arrive)
```

The key is that a new comment appearing in `CommentThread` does NOT  
cause `FileHeader` or `DiffLines` to re-render. Careful state ownership.

---

## Mini Exercise

Run the following component and observe the behavior:

```tsx
"use client";

import { useState } from "react";

type LogEntry = {
  id: number;
  message: string;
  timestamp: string;
};

function RenderTracker() {
  const [fastCount, setFastCount] = useState(0);
  const [slowCount, setSlowCount] = useState(0);
  const [renderLog, setRenderLog] = useState<LogEntry[]>([]);

  const logRender = (source: string) => {
    const entry: LogEntry = {
      id: Date.now(),
      message: `Re-render triggered by: ${source}`,
      timestamp: new Date().toISOString(),
    };
    setRenderLog((prev) => [entry, ...prev.slice(0, 9)]);
  };

  return (
    <div className="render-tracker">
      <h2>Render Tracker</h2>

      <div className="counter-section">
        <button
          onClick={() => {
            setFastCount((prev) => prev + 1);
            logRender("Fast Counter");
          }}
        >
          Fast Counter: {fastCount}
        </button>

        <button
          onClick={() => {
            setSlowCount((prev) => prev + 1);
            logRender("Slow Counter");
          }}
        >
          Slow Counter: {slowCount}
        </button>
      </div>

      <div className="render-log">
        <h3>Render Log (last 10)</h3>
        {renderLog.map((entry) => (
          <div key={entry.id} className="log-entry">
            <span className="log-time">{entry.timestamp}</span>
            <span className="log-message">{entry.message}</span>
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Observation tasks:**
1. Click each button and watch the render log
2. Notice that every state change causes a re-render
3. Understand that `renderLog` updating is also a state change

---

## Mini Quiz

**Question 1:** What is the Virtual DOM?

```
A) A faster version of the real DOM built into the browser
B) A JavaScript object in memory that represents the DOM tree
C) A new HTML standard from the W3C
D) A special browser API for faster rendering
```

*Answer: B*

---

**Question 2:** What is "Diffing" in the context of React?

```
A) Removing components from the tree
B) Comparing the old and new Virtual DOM to find changes
C) The process of converting JSX to JavaScript
D) A method of fetching data
```

*Answer: B*

---

**Question 3:** When does a React component re-render?

```
A) Only when the user clicks something
B) Every second automatically
C) When its state, props, or consumed context changes
D) Only when the page is refreshed
```

*Answer: C*

---

**Question 4:** What is Hydration?

```
A) Adding water to dry components
B) React attaching event handlers to server-rendered HTML to make it interactive
C) The process of fetching data after the page loads
D) Converting TypeScript to JavaScript
```

*Answer: B*

---

**Question 5:** Why should you not use array index as a list key?

```
A) Index keys are slower than string keys
B) React doesn't accept numbers as keys
C) When items are added, removed, or reordered, indices shift, causing incorrect reconciliation
D) Index keys cause hydration errors
```

*Answer: C*

---

**Question 6:** What is the Render Phase in React's rendering pipeline?

```
A) The phase where the browser paints the screen
B) The phase where component functions run and Virtual DOM is built (pure, may be interrupted)
C) The phase where effects run
D) The phase where the server sends HTML
```

*Answer: B*

---

## Summary

| Concept | Explanation |
|---------|-------------|
| Real DOM | The browser's representation of the HTML page (slow to update) |
| Virtual DOM | A JavaScript object mirror of the DOM (fast to compare) |
| Diffing | Comparing old vs new Virtual DOM to find what changed |
| Reconciliation | Applying only the changes to the real DOM |
| Rendering | React calling your component function to get JSX |
| Re-render | React calling your component function again after state/props change |
| Hydration | Attaching event listeners to server-rendered HTML on the client |
| Fiber | React's internal rendering engine (enables concurrent features) |
| Concurrent Mode | Allows React to pause, resume, and prioritize renders |
| key prop | Helps React identify list items across re-renders |

---

## Preparation for the Next Topic

Next file: **004-react-lifecycle-and-hooks.md**

In that file, you'll learn:
- What is the lifecycle of a React component?
- When does a component mount, update, and unmount?
- What are React Hooks and why were they invented?
- Deep dives into: `useState`, `useEffect`, `useRef`, `useContext`, `useReducer`
- The rules of Hooks and why they exist

Before reading: Think about what "lifecycle" might mean for a UI component.  
Just like living things are born, grow, and die — components also appear, change, and disappear.

---

*Next: [004-react-lifecycle-and-hooks.md](004-react-lifecycle-and-hooks.md)*
