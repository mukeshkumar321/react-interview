# React Rendering & Internals


## 1. Introduction to React Rendering

React rendering is the process of converting component logic into UI updates on the screen.

When React renders:

1. It executes component functions
2. Creates React Elements (JS objects)
3. Builds/updates Virtual DOM
4. Compares changes
5. Updates the real DOM efficiently

React does **not** directly manipulate the DOM on every change.

Instead, React uses an optimized rendering pipeline.


### Example

```jsx
function App() {
  return <h1>Hello React</h1>;
}
```

React converts this into:

```js
{
  type: "h1",
  props: {
    children: "Hello React"
  }
}
```

This object becomes part of the Virtual DOM tree.


### Important Interview Insight

Rendering does NOT always mean DOM updates.

A component can re-render without changing the actual DOM.

This is one of the most misunderstood React concepts.


## 2. How React Creates and Updates UI

React follows a predictable flow:

```text
State Change
    ↓
Component Re-render
    ↓
New Virtual DOM
    ↓
Diffing
    ↓
Minimal DOM Updates
```


### Initial Render

During first render:

- React creates component tree
- Builds Virtual DOM
- Creates real DOM nodes
- Mounts them to browser DOM


### Update Render

When state/props change:

- React creates a new Virtual DOM tree
- Compares with previous tree
- Updates only changed nodes


### Example

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

On click:

- Component function runs again
- New Virtual DOM created
- Only text node updates

Button DOM node is reused.


## 3. Virtual DOM

Virtual DOM is a lightweight JavaScript representation of the real DOM.


### Why React Uses Virtual DOM

Direct DOM operations are expensive because:

- Layout recalculations happen
- Paint operations occur
- Browser rendering pipeline is costly

React minimizes these operations.


### Virtual DOM Structure

```js
{
  type: "div",
  props: {
    children: [...]
  }
}
```

React compares previous and next Virtual DOM trees.


### Important Clarification

Virtual DOM itself is not magical.

The real optimization comes from:

- Efficient diffing
- Batched updates
- Smart reconciliation


### Interview Insight

React is fast not because of Virtual DOM alone,
but because React avoids unnecessary real DOM mutations.


## 4. Real DOM vs Virtual DOM

| Real DOM | Virtual DOM |
|---|---|
| Actual browser DOM | JS representation |
| Slow updates | Fast comparisons |
| Direct mutations expensive | Cheap object operations |
| Causes repaint/reflow | No browser cost |
| Managed by browser | Managed by React |


### Real DOM Problem

Changing many DOM nodes directly causes:

- Reflow
- Repaint
- Layout thrashing


### React Optimization

Instead of updating entire DOM:

```text
Old VDOM
   vs
New VDOM
```

React updates only changed parts.


## 5. Initial Render vs Re-render

### Initial Render

First time component appears.

Steps:

```text
Component executes
→ VDOM created
→ DOM nodes created
→ Mounted
```


### Re-render

Occurs when component updates.

Steps:

```text
Component executes again
→ New VDOM
→ Diffing
→ Minimal updates
```


### Important Insight

Re-render ≠ remount

Re-render preserves:

- State
- DOM nodes (if possible)

Remount resets everything.


### Example

```jsx
<Component />
```

Re-render:

- Function runs again

Remount:

- Entire component recreated


## 6. What Causes Re-rendering

React components re-render when:


### 1. State Changes

```jsx
setCount(5);
```

Most common reason.


### 2. Parent Re-renders

Parent render triggers child render by default.


### 3. Props Change

```jsx
<User name="John" />
```

New props trigger render.


### 4. Context Value Changes

All consumers re-render.


### 5. Force Update

Class components:

```js
this.forceUpdate()
```

Rarely used.


## 7. Component Re-render Flow

### Flow

```text
Trigger Update
    ↓
Component Function Executes
    ↓
Hooks Re-evaluated
    ↓
JSX Returned
    ↓
VDOM Comparison
    ↓
Commit Changes
```


### Important Concept

React re-renders top-down.

If parent renders:

- Children render too
- Unless memoized


## 8. Parent vs Child Re-rendering

### Default Behavior

```jsx
<Parent>
   <Child />
</Parent>
```

If Parent re-renders:

- Child also re-renders

Even if child props unchanged.


### Why?

Because child component function executes again.


### Preventing Child Re-renders

Use:

```jsx
React.memo()
```


### Example

```jsx
const Child = React.memo(() => {
  console.log("Child render");
  return <div>Child</div>;
});
```


### Important Interview Insight

React.memo prevents unnecessary renders,
not unnecessary DOM updates.


## 9. State Update Lifecycle

When calling state setter:

```jsx
setCount(count + 1);
```

React does NOT immediately update state synchronously.


### Lifecycle

```text
setState called
    ↓
Update queued
    ↓
React schedules render
    ↓
Render phase
    ↓
Commit phase
```


### Example

```jsx
setCount(count + 1);
console.log(count);
```

Old value logs because update is scheduled.


## 10. State Batching

React batches multiple updates together.


### Example

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

Single render occurs.

Final value:

```text
+3
```


### Why Batching Exists

Reduces:

- Re-renders
- DOM updates
- Performance cost


### React 18 Automatic Batching

Now batching works inside:

- Promises
- setTimeout
- Native events


### Interview Insight

Before React 18:

Only React event handlers were batched.


## 11. Reconciliation Algorithm

Reconciliation is React’s process of determining what changed.


### Goal

Efficiently update UI with minimal DOM work.


### React Assumptions

#### 1. Different element types → different trees

```jsx
<div />
<span />
```

React destroys old tree.


#### 2. Keys identify stable children

Used for lists.


## 12. Diffing Algorithm

React compares old vs new trees.

This process is called diffing.


### Naive Comparison Cost

Tree comparison complexity:

```text
O(n³)
```

Too expensive.


### React Optimization

React reduces complexity to:

```text
O(n)
```

Using heuristics.


### Rules

#### Same Type

```jsx
<div> → <div>
```

Reuse node.


#### Different Type

```jsx
<div> → <span>
```

Destroy + recreate.


## 13. Role of Keys in Rendering

Keys help React identify list items.


### Correct Example

```jsx
users.map(user => (
  <User key={user.id} />
));
```


### Wrong Example

```jsx
key={index}
```

Can cause:

- Wrong state preservation
- UI bugs
- Performance issues


### Why Stable Keys Matter

React uses keys to:

- Match old/new children
- Preserve component state
- Avoid remounting


### Senior-level Insight

Bad keys create hidden rendering bugs.

Especially in:

- Forms
- Animations
- Dynamic lists


## 14. Mounting, Updating, and Unmounting

### Mounting

Component appears first time.


### Updating

Component re-renders due to changes.


### Unmounting

Component removed from DOM.

Cleanup occurs.


### Example

```jsx
useEffect(() => {
  return () => {
    console.log("cleanup");
  };
}, []);
```

Runs during unmount.


## 15. React Fiber Architecture

Fiber is React’s internal rendering engine introduced in React 16.


### Why Fiber Was Needed

Old React stack renderer was:

- synchronous
- blocking
- non-interruptible

Large renders froze UI.


### Fiber Enables

- Incremental rendering
- Prioritization
- Pausing work
- Resuming work
- Concurrent rendering


### Important Insight

Fiber made React scheduling possible.


## 16. Fiber Tree Structure

Each component becomes a Fiber node.


### Fiber Contains

- component type
- props
- state
- child
- sibling
- parent references


### Structure

```text
App
 ├── Header
 ├── Main
 │    ├── Sidebar
 │    └── Content
```

Stored as linked Fiber nodes.


## 17. Render Phase vs Commit Phase

React work happens in 2 phases.


### 1. Render Phase

React:

- calculates changes
- builds work-in-progress tree
- can pause/interupt work

No DOM updates yet.


### 2. Commit Phase

React:

- updates DOM
- runs effects
- applies mutations

Cannot be interrupted.


### Important Interview Insight

useEffect runs after commit phase.


## 18. Concurrent Rendering Basics

Concurrent rendering allows React to prepare UI in background.


### Benefits

- smoother UI
- interruptible rendering
- better responsiveness


### Example

Low-priority updates can pause:

```text
Typing > Heavy Rendering
```

User interactions stay responsive.


### APIs

```jsx
startTransition()
useTransition()
```


## 19. React Strict Mode

Strict Mode helps detect unsafe patterns.


### Enabled By

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```


### Checks

- unsafe lifecycle methods
- side effects
- deprecated APIs
- unexpected mutations


## 20. Double Rendering in Strict Mode

In development mode:

React intentionally double invokes certain functions.


### Why?

To detect impure rendering logic.


### Example

```jsx
function App() {
  console.log("render");
  return <div>Hello</div>;
}
```

Logs twice in development.


### Important Clarification

Only happens in:

- development
- Strict Mode

Not production.


## 21. React.memo Internals

`React.memo` memoizes component rendering.


### Example

```jsx
const Card = React.memo(function Card(props) {
  return <div>{props.title}</div>;
});
```


### Internally

React performs shallow comparison:

```js
prevProps === nextProps
```

For primitive values.

Objects compared by reference.


## 22. Referential Equality in React

Objects/functions create new references every render.


### Example

```jsx
const obj = {};
```

New reference each render.


### Problem

Memoized children re-render.


### Solution

```jsx
useMemo()
useCallback()
```


### Interview Insight

Most unnecessary renders happen due to unstable references.


## 23. Rendering Performance Bottlenecks

Common causes:


### 1. Unnecessary Re-renders

Parent renders cascading down.


### 2. Large Lists

Thousands of DOM nodes.

Solution:

- virtualization
- windowing


### 3. Expensive Calculations

Heavy computations during render.


### 4. Unstable References

Inline objects/functions.


### 5. Context Overuse

All consumers re-render.


## 24. React Profiler

Profiler helps measure rendering performance.


### Available In

React DevTools.


### Shows

- render duration
- render frequency
- unnecessary renders


### Important Metrics

#### Commit Duration

Time to update UI.

#### Render Count

How often components render.


## 25. Hydration in React

Hydration attaches React to server-rendered HTML.

Used in SSR frameworks like:

- Next.js
- Remix


### Flow

```text
Server HTML
    ↓
Browser receives HTML
    ↓
React attaches listeners
```


### Benefit

Faster first paint.

Better SEO.


## 26. Hydration Mismatch Problems

Occurs when server HTML differs from client render.


### Example

```jsx
<Date.now() />
```

Different values server/client.


### Causes

- random values
- browser-only APIs
- timezone differences


### Solutions

- move logic to useEffect
- avoid non-deterministic rendering


## 27. Suspense Rendering Basics

Suspense handles async rendering gracefully.


### Example

```jsx
<Suspense fallback={<Loader />}>
  <Profile />
</Suspense>
```


### Benefits

- loading states
- async UI coordination
- smoother rendering


## 28. Error Boundaries

Error boundaries catch rendering errors.


### Example

```jsx
class ErrorBoundary extends React.Component {
  componentDidCatch(error) {
    console.log(error);
  }
}
```


### They Catch

- render errors
- lifecycle errors
- constructor errors


### They Do NOT Catch

- async errors
- event handler errors


## 29. Rendering Anti-patterns

### 1. Inline Object Creation

```jsx
<Component style={{ color: "red" }} />
```

New object every render.


### 2. Using Index as Key

Causes unstable identity.


### 3. Heavy Logic Inside Render

Blocks rendering.


### 4. Overusing Context

Massive re-renders.


### 5. Premature Memoization

Can increase complexity.


## 30. Production Rendering Best Practices


### Keep Components Pure

Same input → same output.


### Use Stable References

```jsx
useMemo()
useCallback()
```

Only when needed.


### Split Large Components

Improves maintainability and rendering isolation.


### Virtualize Large Lists

Use:

- react-window
- react-virtualized


### Avoid Deep Prop Drilling

Prefer:

- composition
- context segmentation
- state libraries


### Profile Before Optimizing

Never optimize blindly.


## 31. Common Senior-level Interview Questions


### Q1. Difference between render and commit phase?

Render phase calculates changes.
Commit phase applies changes to DOM.


### Q2. Why does React use keys?

To preserve identity and optimize reconciliation.


### Q3. Why do child components re-render?

Because parent re-render triggers child execution unless memoized.


### Q4. Why is React Fiber important?

It enables interruptible rendering and scheduling.


### Q5. Why does React re-render even if DOM doesn't change?

React must compare output before deciding whether DOM updates are needed.


### Q6. What causes unnecessary re-renders?

- unstable references
- parent renders
- context updates
- incorrect memoization


### Q7. Difference between React.memo and useMemo?

`React.memo`
→ memoizes component rendering

`useMemo`
→ memoizes computed value


### Q8. Why are index keys problematic?

They break identity during reordering.

Can cause state bugs.


### Q9. What happens during hydration?

React attaches event listeners to server-rendered HTML.


### Q10. What is reconciliation?

React’s algorithm for updating UI efficiently by comparing trees.