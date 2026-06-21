## React Rendering & Internals

## Topics Covered

- [1. React Rendering Flow](#1-react-rendering-flow)
- [2. Virtual DOM](#2-virtual-dom)
- [3. Reconciliation](#3-reconciliation)
- [4. Diffing Algorithm](#4-diffing-algorithm)
- [5. Render vs Commit Phase](#5-render-vs-commit-phase)
- [6. Fiber Architecture](#6-fiber-architecture)
- [7. Concurrent Rendering](#7-concurrent-rendering)
- [8. Batching](#8-batching)
- [9. Strict Mode](#9-strict-mode)
- [10. React Lifecycle Internals](#10-react-lifecycle-internals)
- [11. Event System](#11-event-system)
- [12. Error Boundaries](#12-error-boundaries)
- [13. Portals](#13-portals)
- [Final Mental Model](#final-mental-model)

---

## 1. React Rendering Flow

When state changes, React doesn't immediately touch the DOM. It runs through a pipeline designed to figure out the *minimum* changes needed:

```txt
State/Props Change → Schedule update → Render Phase → Reconcile → Commit Phase → Browser paint
```

**What triggers a render:**
- `setState` / `useState` updater called
- Props change
- Context changes
- Parent re-renders

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

When `setCount` runs, React schedules an update — not an immediate DOM change. It creates a new virtual tree, compares it against the old one, and only then decides what needs to change in the DOM.

**JSX is just plain objects.** Under the hood:

```jsx
<h1>Hello</h1>
// becomes:
{ type: "h1", props: { children: "Hello" } }
```

React builds a tree of these lightweight objects and operates on them in memory before touching the browser.

**Re-render ≠ DOM update.** React re-executes the whole component function to produce a new tree, but only *actual differences* lead to DOM mutations. The component may run again while the browser changes nothing.

**Scheduling** — Modern React batches and prioritizes updates instead of processing them immediately. Typing in an input gets higher priority than rendering a large list. This is what keeps the UI responsive.

---

## 2. Virtual DOM

The Virtual DOM is React's in-memory JS representation of the UI — a tree of plain objects describing what should be on screen. It's **not** the reason React is fast.

A naive Virtual DOM implementation can still be slow. React's actual performance comes from:

- **Efficient reconciliation** — smart tree diffing
- **Fiber architecture** — interruptible rendering
- **Batching** — grouping multiple state updates into one render
- **Scheduling** — priority-based update processing
- **Minimal DOM mutations** — only changed nodes get touched

The Virtual DOM's real value is **predictability**: React computes the full diff in JS before committing anything to the browser. Expensive DOM operations (which trigger layout and repaint) only happen once, at the very end.

---

## 3. Reconciliation

Reconciliation is how React answers: *"What changed between the old tree and the new one?"*

```jsx
// Old                  New
<ul>                    <ul>
  <li>A</li>              <li>A</li>
  <li>B</li>              <li>C</li>
</ul>                   </ul>
```

React walks both trees together — first item unchanged, second item changed. Only the second `<li>` gets updated in the DOM.

**Two core assumptions:**

1. **Different element types → replace the entire subtree.** If `<div>` becomes `<span>`, React doesn't try to patch it. It destroys and recreates.

2. **Keys identify stable children.** Without keys, React matches list items by index. This breaks when items reorder — state shifts to the wrong element.

```jsx
users.map(user => <User key={user.id} />); // stable identity
```

**Why this matters for state:** If React considers a component "the same" across renders, its state survives. If it thinks it's new (different type or key), state resets. A component can lose its form inputs seemingly out of nowhere — keys are almost always the culprit.

A mathematically optimal tree diff runs in O(n³), which is too slow for any real UI. React uses heuristics to achieve near-linear performance — which is what the next section (Diffing Algorithm) covers in detail.

---

## 4. Diffing Algorithm

React's diffing is the practical implementation of reconciliation using two simple rules.

**Rule 1: Different types → destroy and recreate.**

```jsx
// <div><Counter /></div>  →  <span><Counter /></span>
// Counter unmounts and remounts — state is lost
```

Different element types usually mean a completely different UI structure, so React doesn't attempt to reuse anything.

**Rule 2: Same type → reuse the DOM node, update only changed props.**

```jsx
<button className="a" />  →  <button className="b" />
// DOM node reused, only className attribute changes
```

**Child diffing and the key problem:**

Without keys, React matches children by position. Inserting at the front shifts every index — React thinks everything changed.

```jsx
// Without keys: inserting X before A causes A, B, C to all re-render
[A, B, C] → [X, A, B, C]

// With stable keys: only X is new, A/B/C are matched correctly
<Item key={item.id} />
```

**Why `key={index}` causes bugs:** When items reorder, React ties the old state (stored at index 0, 1, 2...) to whatever item now sits at that position. Input values appear to jump between items; animations break; components show stale data.

---

## 5. Render vs Commit Phase

React splits work into two distinct phases. Conflating them causes most rendering-related bugs.

**Render Phase** — pure calculation. React calls your component functions, builds a new fiber tree, and diffs it against the old one. No DOM changes happen here. In concurrent mode, React may run this phase multiple times — pausing, restarting, or discarding it entirely. That's why rendering must stay pure:

```jsx
// Bad: side effects during render
function App() {
  localStorage.setItem("x", "1"); // may run multiple times
  return <div />;
}
```

**Commit Phase** — applying changes to the world. React flushes DOM mutations, runs `useLayoutEffect`, attaches refs, then runs `useEffect` after the browser paints. This phase is **synchronous and uninterruptible** — once it starts, it finishes.

| Hook | When it runs |
|---|---|
| `useLayoutEffect` | After DOM update, before browser paint |
| `useEffect` | After browser paint |

Key insight: **concurrent rendering only affects the render phase.** The commit phase always runs to completion.

---

## 6. Fiber Architecture

**What is Fiber?** Before React 16, React rendered the entire component tree in one go, synchronously — it couldn't pause in the middle. If you had a large tree, the browser was frozen until React finished. Fiber is the complete rewrite of React's rendering engine that solved this.

Think of Fiber as React's internal task scheduler. Instead of "render everything now", Fiber breaks rendering into small units of work and processes them one by one — pausing between units if the browser needs to handle something more urgent (like a user keystroke).

Fiber represents each component as a node in a **linked list**. Each node stores component type, props, state, parent/child/sibling pointers, pending effects, and priority. This structure lets React process work in small units and pause at any point to yield control back to the browser.

**Double buffering:** React maintains two trees simultaneously — the *current* tree (what's on screen) and the *work-in-progress* tree (the update being built). After commit, they swap. This means React can abandon an in-progress update without corrupting the live UI.

Fiber is what made Suspense, transitions, and concurrent rendering possible.

---

## 7. Concurrent Rendering

**Why does this exist?** JavaScript runs on a single thread — meaning React and your UI (scrolling, typing, animations) share the same thread. Before React 18, if React was in the middle of a big render, everything else had to wait. Your keystrokes would lag. Concurrent rendering fixes this.

Concurrent rendering lets React prepare updates in the background without blocking the browser. "Concurrent" here does NOT mean parallel threads — JavaScript is still single-threaded. It means React can *yield* between units of work, letting the browser handle urgent tasks (like input), then resume rendering.

Without concurrency, a heavy render (large list, complex tree) blocks the main thread. Typing lags. Scrolling stutters. With concurrency, React pauses low-priority renders to handle high-priority input events, then resumes.

| Task | Priority |
|---|---|
| User input | High |
| Animations | High |
| Background list render | Lower |

`startTransition` marks an update as non-urgent:

```jsx
startTransition(() => {
  setSearchQuery(value); // React can defer this
});
```

This keeps typing instant even while filtering a large list. The search still runs — just not at the cost of input responsiveness.

---

## 8. Batching

Batching groups multiple state updates into a single render.

```jsx
setCount(c => c + 1);
setFlag(true);
// → one render, not two
```

Before React 18, batching only worked inside React event handlers. Updates inside `setTimeout`, Promises, or async functions each triggered their own render. **React 18 introduced automatic batching everywhere.**

To opt out when you explicitly need two separate renders:

```jsx
import { flushSync } from 'react-dom';
flushSync(() => setCount(c => c + 1)); // forces immediate render
```

---

## 9. Strict Mode

`<React.StrictMode>` is a development-only tool — it does nothing in production.

In development, React deliberately double-invokes render functions and effects to surface two classes of bugs:

1. **Impure renders** — side effects during rendering fire twice, making the problem visible
2. **Effect cleanup bugs** — effects mount, unmount, and remount; missing cleanup causes errors or leaks

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```

The double-invocation mirrors how concurrent rendering can pause and restart renders. If code breaks in Strict Mode, it would break in concurrent mode too.

---

## 10. React Lifecycle Internals

Hooks don't have named lifecycle methods, but React still follows the same three phases internally.

| Phase | Trigger | What happens |
|---|---|---|
| Mount | First render | Fiber created, component runs, DOM built, effects fire |
| Update | State/props/context change | Component reruns, tree reconciled, DOM patched, effects run |
| Unmount | Component removed | Effects cleaned up, refs detached, DOM removed |

**Class lifecycle → hooks mapping:**

| Class | Hooks equivalent |
|---|---|
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [dep])` |
| `componentWillUnmount` | `useEffect(() => { return cleanup }, [])` |

**Why hooks can't be conditional:** React stores hook state as a linked list on each fiber node. The order of calls must be identical every render so React can match each hook call to its stored state. A conditional hook breaks the index alignment, corrupting state for every hook that follows it.

---

## 11. Event System

React doesn't attach event listeners to individual elements. It attaches a small set of listeners at the **root** and uses event delegation — native browser events bubble up, React intercepts them centrally.

```txt
10,000 buttons → 1 root listener (not 10,000)
```

React wraps native events in **SyntheticEvents** — normalized objects with a consistent API across browsers. For most code, you'll never notice the difference.

**Event pooling** was an older optimization where React reused event objects between handlers. It caused confusing bugs in async code (`event.target` would be null after an `await`). It was removed in React 17.

**Events and scheduling:** React integrates event handling with the Fiber scheduler, assigning priorities. A click event can preempt a background render. This is how concurrent rendering stays responsive without you writing any explicit throttling code.

---

## 12. Error Boundaries

Error boundaries are React components that catch JavaScript errors in their child component tree, log them, and display a fallback UI instead of crashing the entire app.

**Class component implementation** (hooks don't support error boundaries yet):

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

**Usage:**

```jsx
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

**What error boundaries catch:**
- Errors during rendering
- Errors in lifecycle methods
- Errors in constructors of child components

**What they DON'T catch:**
- Event handlers (use try/catch instead)
- Asynchronous code (setTimeout, promises)
- Server-side rendering errors
- Errors in the error boundary itself

**Why they matter:** Without error boundaries, a single error in a small component crashes your entire React tree. Error boundaries contain the damage — the rest of the app keeps working.

**Interview tip:** For 4 YOE, you're expected to know error boundaries exist and when to use them. Production apps should have error boundaries at strategic points (route level, feature boundaries).

**Production reality:** Writing the class manually every time is tedious. The `react-error-boundary` npm package is the standard way to add error boundaries in real apps — it gives you a functional, reusable `<ErrorBoundary>` component with reset support, so you almost never write the class yourself.

```jsx
import { ErrorBoundary } from "react-error-boundary";

<ErrorBoundary fallback={<ErrorPage />}>
  <FeatureSection />
</ErrorBoundary>
```

---

## 13. Portals

Portals let you render children into a DOM node that exists outside the parent component's DOM hierarchy.

**Use case:** Modals, tooltips, dropdowns — UI that needs to break out of `overflow: hidden` or `z-index` stacking contexts.

```jsx
ReactDOM.createPortal(child, domNode)
```

**Example:**

```jsx
function Modal({ children }) {
  return ReactDOM.createPortal(
    <div className="modal">
      {children}
    </div>,
    document.getElementById('modal-root')
  );
}
```

**HTML structure:**

```html
<body>
  <div id="root"><!-- React app --></div>
  <div id="modal-root"><!-- Portals render here --></div>
</body>
```

**Key behavior:** Even though the modal renders in a different DOM tree, it still behaves like a normal React child:
- Events bubble up through React tree (not DOM tree)
- Context works normally
- State flows down as expected

**Why use portals:**
- Escape CSS constraints (overflow, z-index)
- Render at document root for accessibility
- Keep React tree structure clean

**Interview insight:** Know the difference between DOM hierarchy and React component hierarchy. Portals break DOM hierarchy while preserving React tree.

---

## Final Mental Model

```txt
UI = f(state)
```

But React internally is:

```txt
Scheduler + Fiber + Reconciler + Priority Queue + DOM Renderer
```

The "Virtual DOM library" framing misses the point. React is an **optimized rendering scheduler** — the Virtual DOM is just how it represents pending work before executing it.
