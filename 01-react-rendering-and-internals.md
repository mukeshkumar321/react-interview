## React Rendering & Internals

## 📚 Topics Covered

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
- [Final Mental Model](#final-mental-model)

## 1. React Rendering Flow

When people start learning React, they often think:

> “State changed → React updates DOM.”

That is technically true, but internally React does much more than that.

The entire React rendering flow exists because directly manipulating the DOM is expensive, hard to scale, and difficult to optimize in large applications.

A simple DOM update is not the real problem.

The real problem is:

- deciding what actually changed
- updating only the necessary parts
- preserving component state correctly
- avoiding UI blocking
- scheduling updates efficiently

React’s rendering system was designed to solve these problems.



### The High-Level Flow

A React update generally follows this flow:

```txt
State/Props Change
        ↓
React schedules update
        ↓
Render Phase
        ↓
Create new React Elements
        ↓
Reconciliation
        ↓
Diff old vs new tree
        ↓
Prepare DOM mutations
        ↓
Commit Phase
        ↓
Update Real DOM
        ↓
Browser paints screen
```



### What Actually Triggers Rendering?

Rendering happens when:

- `setState` is called
- props change
- context changes
- parent re-renders
- force update happens

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

When `setCount()` runs:

React does NOT immediately change the DOM.

Instead, React:

1. schedules an update
2. creates a new virtual tree
3. compares old vs new
4. computes changes
5. commits updates later



### React Elements Are Just Objects

JSX:

```jsx
<h1>Hello</h1>
```

becomes:

```js
{
  type: "h1",
  props: {
    children: "Hello"
  }
}
```

These lightweight objects form the Virtual DOM tree.

React rendering is essentially:

```txt
UI Function → JS Objects → Comparison → DOM Update
```



### Render Phase

During rendering:

React executes components again.

Example:

```jsx
function App() {
  return <h1>Hello</h1>;
}
```

React literally calls:

```js
App();
```

This produces new React elements.

Important:

Rendering does NOT mean DOM update.

It only means:

> “Calculate what UI should look like.”



### Why React Re-renders Entire Components

A common misconception:

> “React updates only changed lines.”

No.

React re-executes the entire component function.

Example:

```jsx
function User({ name }) {
  console.log("render");

  return <h1>{name}</h1>;
}
```

Changing one prop reruns the whole component function.

Why?

Because React components are pure functions.

React cannot partially execute JavaScript functions safely.

So React reruns the function and compares output.



### Component Tree Traversal

React renders top-down.

Example:

```jsx
<App>
  <Header />
  <Sidebar />
  <Content />
</App>
```

If `App` re-renders:

React traverses children too.

But DOM updates only happen where differences exist.

This distinction is critical:

| Phase | Meaning |
|---|---|
| Re-render | Execute component again |
| DOM update | Actual browser mutation |

They are NOT the same thing.



### Scheduling

Modern React does not immediately process every update.

It schedules them.

This allows:

- batching
- prioritization
- interruptible rendering
- concurrent rendering

Example:

Typing in an input gets higher priority than rendering a huge list.

Without scheduling:

UI would freeze.



### Internal Pipeline

Internally React roughly does:

```txt
Update Queue
    ↓
Scheduler
    ↓
Fiber Tree Processing
    ↓
Render Phase
    ↓
Effect Collection
    ↓
Commit Phase
```

Fiber made this architecture possible.



### Important Misconception

#### “React updates the DOM efficiently because Virtual DOM is fast.”

Not exactly.

The Virtual DOM itself is not the optimization.

The optimization is:

- batching
- reconciliation
- scheduling
- selective DOM mutation
- async rendering

The Virtual DOM is mainly a representation layer.



## 2. Virtual DOM

The Virtual DOM is one of the most misunderstood parts of React.

People often think:

> “React is fast because of Virtual DOM.”

That explanation is incomplete.

The Virtual DOM is primarily an abstraction layer that helps React reason about UI changes predictably.



### The Core Problem

Direct DOM manipulation is expensive because:

- DOM operations trigger layout/repaint
- browser rendering pipeline is slow compared to JS
- managing large UI trees manually becomes complex

Example without React:

```js
const div = document.createElement("div");
div.textContent = "Hello";
document.body.appendChild(div);
```

This works.

But scaling this across thousands of dynamic UI updates becomes difficult.

React wanted:

- declarative UI
- predictable rendering
- efficient updates



### What Virtual DOM Actually Is

The Virtual DOM is simply:

> A JavaScript representation of the UI.

Example:

```jsx
<h1>Hello</h1>
```

becomes:

```js
{
  type: "h1",
  props: {
    children: "Hello"
  }
}
```

React builds trees of these objects.



### Old Tree vs New Tree

When state changes:

React creates a NEW tree.

Then compares:

```txt
Previous Tree
vs
New Tree
```

This comparison process is reconciliation.



### Why Use JS Objects?

Because JavaScript operations are cheap.

Comparing objects is faster than blindly mutating the DOM.

React can compute:

```txt
What changed?
What stayed same?
What should update?
```

before touching the browser.



### Virtual DOM Is NOT the Real Optimization

This is extremely important.

The Virtual DOM alone is NOT magical.

A naive Virtual DOM implementation can still be slow.

React became fast because of:

- efficient reconciliation
- Fiber architecture
- scheduling
- batching
- prioritization
- minimized DOM mutations



### React Still Updates Real DOM Manually

React eventually performs manual DOM operations like:

```js
element.textContent = "Updated";
```

The browser does not understand Virtual DOM.

Only React does.



### Misconception: “React compares entire DOM trees deeply”

Not exactly.

That would be too slow.

React uses heuristics.

This led to the Diffing Algorithm.



## 3. Reconciliation

Reconciliation is the process React uses to determine:

> “What changed between previous UI and next UI?”

Without reconciliation:

React would need to rebuild the entire DOM every update.

That would be extremely inefficient.



### Example

Old UI:

```jsx
<ul>
  <li>A</li>
  <li>B</li>
</ul>
```

New UI:

```jsx
<ul>
  <li>A</li>
  <li>C</li>
</ul>
```

React determines:

```txt
First item unchanged
Second item changed
```

Only second node updates.



### Tree Comparison

React compares trees node-by-node.

Example:

```txt
App
 ├── Header
 ├── Sidebar
 └── Content
```

If only `Content` changes:

React avoids touching `Header`.



### Key Assumptions

React’s reconciliation depends on two assumptions:

#### 1. Different element types produce different trees

```jsx
<div />
<span />
```

React assumes entire subtree changed.



#### 2. Keys identify stable children

Example:

```jsx
users.map(user => (
  <User key={user.id} />
));
```

Keys help React preserve identity.

Without stable keys:

- state bugs happen
- unnecessary remounting occurs
- performance degrades



### State Preservation

Reconciliation determines whether component state survives.

Example:

```jsx
<Input />
```

If React considers it “same component”:

state preserved.

If React thinks it is new:

state resets.

This is why keys matter deeply.



### Mount vs Update vs Unmount

Reconciliation decides:

| Operation | Meaning |
|---|---|
| Mount | Create new node |
| Update | Reuse existing node |
| Unmount | Remove node |



### Why React Doesn't Use Optimal Tree Diffing

A mathematically optimal tree diff is extremely expensive:

```txt
O(n³)
```

Too slow for UI.

React instead uses heuristics for near-linear performance.



## 4. Diffing Algorithm

React’s diffing algorithm is the practical implementation of reconciliation.

Its goal:

> Efficiently compare old and new trees.



### Naive Approach Problem

Imagine comparing every node deeply.

Complexity becomes:

```txt
O(n³)
```

Impossible for large apps.

React instead uses heuristic diffing.



### Rule 1: Different Types = Replace Entire Tree

Example:

```jsx
<div>
  <Counter />
</div>
```

to:

```jsx
<span>
  <Counter />
</span>
```

React destroys old subtree and creates new one.

Why?

Different element types usually represent different UI structures.



### Rule 2: Same Type = Reuse Node

Example:

```jsx
<button className="a" />
```

to:

```jsx
<button className="b" />
```

React updates only changed props.

DOM node reused.



### Child Diffing

Without keys:

React compares by index.

Example:

```jsx
[A, B, C]
```

to:

```jsx
[X, A, B, C]
```

React thinks everything changed.

Very inefficient.



### Keys Solve This

With keys:

```jsx
[
  {id:1},
  {id:2}
]
```

React can match identities correctly.



### Why Index Keys Cause Bugs

Example:

```jsx
key={index}
```

If list order changes:

React preserves wrong state.

Example:

- input values shift
- animations break
- incorrect DOM reuse



### Performance Impact

Good keys help React:

- reuse components
- avoid remounting
- preserve state
- minimize DOM operations



## 5. Render vs Commit Phase

This distinction is fundamental to understanding React internals.

Most developers think rendering means DOM updates.

It does not.

React splits work into two phases.



### Render Phase

Purpose:

> Calculate what should change.

React:

- executes components
- builds new tree
- compares old vs new
- prepares effects

No DOM mutations happen here.

This phase must stay pure.



### Why Purity Matters

React may:

- pause rendering
- restart rendering
- discard rendering

Especially in concurrent rendering.

If rendering had side effects:

React would break.



### Example Problem

Bad:

```jsx
function App() {
  localStorage.setItem("x", "1");
  return <div />;
}
```

Render now has side effects.

React may execute it multiple times.



### Commit Phase

Purpose:

> Apply changes to the real world.

Now React:

- updates DOM
- runs effects
- attaches refs

Commit phase is synchronous.

Cannot be interrupted.



### Why Commit Is Separate

Because DOM mutation is expensive and must remain consistent.

React can pause rendering.

But once commit starts:

it must finish.



### Effects Timing

| Hook | Timing |
|---|---|
| useEffect | After paint |
| useLayoutEffect | Before paint |



## 6. Fiber Architecture

Fiber is React’s internal rendering engine introduced in React 16.

Before Fiber:

React rendering was synchronous.

Problem:

Large renders blocked the main thread.

UI froze.



### The Core Idea

Fiber breaks rendering work into small units.

Instead of:

```txt
Render everything now
```

React can now:

```txt
Pause
Resume
Prioritize
Abort
Reuse
```



### Fiber Node

Each component gets a Fiber node.

Fiber stores:

- component type
- props
- state
- parent/child/sibling links
- effects
- priority

Fiber is basically:

> React’s internal linked-list tree structure.



### Double Buffering

React maintains:

| Tree | Purpose |
|---|---|
| Current Tree | Visible UI |
| Work-In-Progress Tree | New updates |

After commit:

trees swap.



### Why Fiber Was Revolutionary

Fiber enabled:

- concurrent rendering
- Suspense
- transitions
- interruptible rendering
- scheduling priorities

Modern React depends heavily on Fiber.



## 7. Concurrent Rendering

Concurrent rendering allows React to prepare UI in the background without blocking the browser.

This does NOT mean multithreading.

JavaScript still runs on one thread.



### The Problem

Without concurrency:

Large rendering tasks freeze UI.

Example:

- typing lags
- scrolling stutters



### Concurrent Solution

React can:

- pause rendering
- yield to browser
- continue later

This improves responsiveness.



### Priority-Based Rendering

React assigns priorities.

Example:

| Task | Priority |
|---|---|
| Typing | High |
| Animation | High |
| Huge list render | Lower |



### Interruptible Rendering

React may start rendering:

```txt
BigComponent
```

Then pause for:

```txt
Input update
```

Then resume later.



### Important Detail

Concurrent rendering affects:

Render Phase only.

Commit phase stays synchronous.



### Transitions

Example:

```jsx
startTransition(() => {
  setSearchQuery(value);
});
```

Marks update as non-urgent.

Typing remains responsive.



## 8. Batching

Batching means:

> Multiple state updates combine into one render.



### Example

```jsx
setCount(c => c + 1);
setFlag(true);
```

React batches them.

Only one render occurs.



### Why Batching Exists

Without batching:

Every state update would trigger rendering.

Very inefficient.



### Automatic Batching

React 18 expanded batching to:

- promises
- timeouts
- async callbacks

Before React 18:

only React event handlers were batched.



### Internal Benefit

Batching reduces:

- render count
- reconciliation work
- DOM mutations

Huge performance improvement.



## 9. Strict Mode

Strict Mode is a development-only tool.

It intentionally makes React more aggressive to detect unsafe patterns.



### Why Components Render Twice

Example:

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```

React may double invoke rendering.

Purpose:

Detect impure logic.



### What Strict Mode Detects

- side effects during render
- unsafe lifecycles
- deprecated APIs
- effect cleanup bugs



### Important Misconception

Strict Mode does NOT affect production.

Only development behavior changes.



### Why Effects Re-run

React intentionally mounts/unmounts effects to verify cleanup correctness.

This helps detect memory leaks.



## 10. React Lifecycle Internals

Class components had explicit lifecycle methods:

```txt
Mount
Update
Unmount
```

Modern React internally still follows similar phases.



### Mount Phase

Component created first time.

Internally React:

- creates Fiber
- executes component
- creates DOM nodes
- commits UI



### Update Phase

Triggered by:

- state changes
- props changes
- context changes

React:

- rerenders component
- reconciles tree
- commits updates



### Unmount Phase

Component removed.

React:

- removes effects
- detaches refs
- removes DOM nodes



### Hooks Mapping

| Class Lifecycle | Hooks Equivalent |
|---|---|
| componentDidMount | useEffect |
| componentDidUpdate | useEffect |
| componentWillUnmount | cleanup |



### Important Internal Detail

Hooks are stored internally as linked lists on Fiber nodes.

Order matters.

That is why hooks cannot be conditional.



## 11. Event System

React does not attach event listeners directly to every element.

Instead, it uses an event delegation system.



### The Problem

Attaching listeners everywhere is expensive.

Example:

```txt
10,000 buttons
= 10,000 listeners
```

Not scalable.



### Event Delegation

React attaches a small number of listeners at the root.

Events bubble upward.

React intercepts them.



### Synthetic Events

React wraps native browser events inside Synthetic Events.

Example:

```jsx
<button onClick={handleClick} />
```

React creates normalized event objects.

Benefits:

- cross-browser consistency
- unified API
- controlled propagation



### Event Pooling (Old React)

Older React reused event objects for performance.

This caused issues:

```js
console.log(event.target);
```

inside async callbacks failed.

React later removed pooling.



### Event Priority

Modern React integrates events with scheduler priorities.

Example:

| Event | Priority |
|---|---|
| Click | High |
| Scroll | High |
| Transition | Lower |

This helps concurrent rendering stay responsive.



## Final Mental Model

React is fundamentally:

```txt
UI = Function(State)
```

But internally React is also:

```txt
Scheduler
+ Fiber Architecture
+ Reconciliation Engine
+ Priority System
+ DOM Renderer
```

Modern React is less about “Virtual DOM library” and more about:

> A highly optimized UI scheduling and rendering engine.