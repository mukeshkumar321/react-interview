## 🪝 React Hooks Deep Dive

## 📚 Topics Covered

- [1. Why Hooks Exist](#1-why-hooks-exist)
- [2. Rules of Hooks](#2-rules-of-hooks)
- [3. useState Internals](#3-usestate-internals)
- [4. React Rendering & State Updates](#4-react-rendering--state-updates)
- [5. useEffect Deep Dive](#5-useeffect-deep-dive)
- [6. Effect Lifecycle & Cleanup](#6-effect-lifecycle--cleanup)
- [7. Dependency Array Internals](#7-dependency-array-internals)
- [8. Stale Closures](#8-stale-closures)
- [9. useRef Internals](#9-useref-internals)
- [10. useMemo Deep Dive](#10-usememo-deep-dive)
- [11. useCallback Deep Dive](#11-usecallback-deep-dive)
- [12. React.memo & Memoization Strategy](#12-reactmemo--memoization-strategy)
- [13. Context API + useContext Internals](#13-context-api--usecontext-internals)
- [14. Custom Hooks Architecture](#14-custom-hooks-architecture)
- [15. useReducer Deep Dive](#15-usereducer-deep-dive)
- [16. Hook Execution Order](#16-hook-execution-order)
- [17. Hook Storage Internals (Fiber + Linked List)](#17-hook-storage-internals-fiber--linked-list)
- [18. Batching with Hooks](#18-batching-with-hooks)
- [19. Concurrent Rendering + Hooks](#19-concurrent-rendering--hooks)
- [20. Strict Mode & Hooks Behavior](#20-strict-mode--hooks-behavior)
- [21. Advanced Hooks](#21-advanced-hooks)
- [22. Hook Performance Optimization](#22-hook-performance-optimization)
- [23. Common Hooks Pitfalls](#23-common-hooks-pitfalls)
- [24. Senior-Level Hooks Patterns](#24-senior-level-hooks-patterns)
- [Final Mental Model](#final-mental-model)



## 1. Why Hooks Exist

Before Hooks, React components were mainly divided into:

- Class Components → state + lifecycle
- Function Components → UI only

Example:

```jsx
class User extends React.Component {
  state = { count: 0 };

  componentDidMount() {
    console.log("Mounted");
  }

  render() {
    return <button>{this.state.count}</button>;
  }
}
```

Problems with classes:

- `this` confusion
- lifecycle duplication
- logic scattered across methods
- hard to reuse stateful logic
- large components became messy
- HOCs/render props caused wrapper hell

Hooks solved this by allowing:

- state in functions
- lifecycle behavior in functions
- reusable stateful logic
- better composition
- simpler mental model

Now logic can be grouped by feature instead of lifecycle phase.

Bad class organization:

```jsx
componentDidMount() {
  fetchUser();
}

componentDidUpdate() {
  fetchUser();
}
```

Hooks:

```jsx
useEffect(() => {
  fetchUser();
}, []);
```

Hooks are fundamentally:

> A way for React Fiber to attach persistent stateful memory to function components.

Functions normally lose variables after execution.

Hooks give React-controlled persistent storage between renders.



## 2. Rules of Hooks

Hooks must follow strict ordering rules.

### Rule 1 — Only Call Hooks at Top Level

❌ Wrong:

```jsx
if (show) {
  useEffect(() => {});
}
```

✅ Correct:

```jsx
useEffect(() => {
  if (show) {
    // logic
  }
}, [show]);
```

Why?

React tracks hooks by call order.

Example:

```jsx
useState(); // Hook 1
useEffect(); // Hook 2
useMemo(); // Hook 3
```

If one hook disappears conditionally:

```jsx
if (x) useEffect();
```

Then hook positions shift.

React would attach wrong state to wrong hooks.



### Rule 2 — Only Call Hooks in React Functions

Allowed:

- function components
- custom hooks

Not allowed:

- regular utility functions
- loops
- event handlers
- class components

Reason:

React needs render context to connect hooks with Fiber.



## 3. useState Internals

`useState` looks simple:

```jsx
const [count, setCount] = useState(0);
```

Internally React creates:

- hook object
- update queue
- state snapshot

Simplified structure:

```js
Hook = {
  memoizedState: 0,
  queue: [],
  next: Hook
}
```

React stores hooks inside Fiber nodes.

Every render walks the hook linked list in order.



### State Updates Are Queued

```jsx
setCount(count + 1);
```

does NOT immediately mutate state.

React creates an update object:

```js
Update = {
  action: count + 1
}
```

Then schedules re-render.



### Functional Updates

Problem:

```jsx
setCount(count + 1);
setCount(count + 1);
```

Both use same stale value.

Solution:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
```

React applies updates sequentially.



### Why State Is Immutable

React depends on reference comparison.

```js
oldObj !== newObj
```

Mutation breaks change detection.



## 4. React Rendering & State Updates

When state changes:

```jsx
setCount(5);
```

React does NOT immediately update DOM.

Pipeline:

1. enqueue update
2. schedule render
3. build new Fiber tree
4. compare old/new trees
5. commit changes

Rendering is pure calculation.

DOM updates happen later.



### Render Phase

React executes component functions again.

```jsx
function App() {
  const [count] = useState(0);

  return <div>{count}</div>;
}
```

This entire function reruns.



### Commit Phase

Only actual DOM mutations occur here.

Example:

```jsx
<div>1</div>
→
<div>2</div>
```

React manually updates text node.



## 5. useEffect Deep Dive

`useEffect` synchronizes React with external systems.

Examples:

- API calls
- subscriptions
- timers
- DOM listeners

Not for deriving UI state.



### Mental Model

Effects run AFTER commit.

```jsx
useEffect(() => {
  console.log("DOM updated");
});
```

React guarantees DOM exists first.



### Why Effects Exist

React rendering must stay pure.

Bad:

```jsx
function App() {
  localStorage.setItem("x", 1);
}
```

Render should not cause side effects.

Effects separate side effects from rendering.



### Timing

Sequence:

1. render phase
2. commit phase
3. browser paint
4. passive effects run



## 6. Effect Lifecycle & Cleanup

Effects have lifecycle behavior.

```jsx
useEffect(() => {
  const id = setInterval(() => {}, 1000);

  return () => clearInterval(id);
}, []);
```

Cleanup prevents leaks.



### Actual Lifecycle

Mount:

```txt
effect runs
```

Dependency change:

```txt
cleanup old effect
run new effect
```

Unmount:

```txt
cleanup runs
```



### Why Cleanup Happens Before New Effect

Prevents duplicate subscriptions.

Without cleanup:

- duplicate listeners
- memory leaks
- stale intervals



## 7. Dependency Array Internals

```jsx
useEffect(() => {}, [count]);
```

React stores previous dependencies.

Internally:

```js
prevDeps = [1]
nextDeps = [2]
```

Comparison uses:

```js
Object.is()
```

NOT deep equality.



### Why Objects Cause Re-renders

```jsx
useEffect(() => {}, [{}]);
```

Every render creates new object reference.

Thus effect reruns.



### Misconception

Dependency array is NOT:

> “When should effect run?”

It is:

> “What values does this effect depend on?”

Huge difference.



## 8. Stale Closures

One of the most important Hooks concepts.

Example:

```jsx
useEffect(() => {
  setInterval(() => {
    console.log(count);
  }, 1000);
}, []);
```

Why stale?

Effect captures initial render snapshot.

Closures preserve old variables.



### React Render Snapshots

Every render creates new scope.

```txt
Render 1 → count = 0
Render 2 → count = 1
Render 3 → count = 2
```

Effects belong to specific render snapshots.



### Solutions

#### Add dependency

```jsx
useEffect(() => {}, [count]);
```

#### Use functional updates

```jsx
setCount(c => c + 1);
```

#### Use refs

```jsx
ref.current
```



## 9. useRef Internals

`useRef` creates persistent mutable container.

```jsx
const ref = useRef();
```

Internally:

```js
{
  current: value
}
```

React preserves same object between renders.



### Important Difference

Changing ref does NOT trigger render.

```jsx
ref.current = 10;
```

Why?

React does not track refs for reconciliation.

Refs are escape hatches from rendering system.



### Common Uses

- DOM access
- previous values
- mutable timers
- avoiding stale closures



## 10. useMemo Deep Dive

`useMemo` memoizes computed values.

```jsx
const value = useMemo(() => expensive(), [x]);
```

React stores:

```js
{
  value,
  deps
}
```

If deps unchanged:

```txt
skip recomputation
```



### Important Reality

`useMemo` is performance optimization ONLY.

Not semantic guarantee.

React may discard cache.



### Bad Usage

```jsx
const arr = useMemo(() => [1,2,3], []);
```

Unnecessary optimization.

Memoization itself has cost.



### Good Usage

- expensive calculations
- stable references
- preventing child rerenders



## 11. useCallback Deep Dive

`useCallback` memoizes functions.

```jsx
const fn = useCallback(() => {}, []);
```

Equivalent to:

```jsx
useMemo(() => fn, deps)
```



### Why Functions Matter

Functions are recreated every render.

```jsx
() => {}
!== 
() => {}
```

New references can trigger child rerenders.



### Useful With

- `React.memo`
- dependency arrays
- event optimization



### Misconception

`useCallback` does NOT prevent function creation.

Function still created.

React simply reuses previous cached reference.



## 12. React.memo & Memoization Strategy

`React.memo` skips rerender if props unchanged.

```jsx
export default React.memo(Button);
```

Comparison uses shallow equality.



### Why This Matters

Parent rerender normally rerenders children.

Memoization can isolate expensive trees.



### Common Failure

```jsx
<Button onClick={() => {}} />
```

New function every render.

Memo fails.



### Real Optimization Strategy

Optimization should follow:

1. avoid unnecessary state
2. colocate state
3. reduce rerenders
4. memoize expensive components

Memoization everywhere is harmful.



## 13. Context API + useContext Internals

Context solves prop drilling.

```jsx
<AuthContext.Provider value={user}>
```

Consumers subscribe to provider value.



### Internals

React stores current context value on provider Fiber.

Consumers register dependency.

When provider value changes:

```txt
all consumers rerender
```



### Important Problem

```jsx
value={{ user }}
```

New object every render.

Causes all consumers to rerender.



### Optimization

```jsx
const value = useMemo(() => ({ user }), [user]);
```



### Misconception

Context is NOT state management.

It is dependency injection mechanism.



## 14. Custom Hooks Architecture

Custom hooks are reusable stateful logic.

```jsx
function useUser() {
  const [user, setUser] = useState(null);

  return user;
}
```

They are NOT special to React.

Just functions using hooks.



### Real Power

Encapsulation.

Custom hooks can hide:

- subscriptions
- caching
- fetching
- synchronization
- complex state logic



### Design Philosophy

Hooks compose behavior vertically.

Classes composed horizontally via inheritance.

Hooks solved logic reuse elegantly.



## 15. useReducer Deep Dive

Alternative to `useState`.

```jsx
const [state, dispatch] = useReducer(reducer, initial);
```

Inspired by Redux.



### Why It Exists

Complex state transitions become predictable.

Bad:

```jsx
setState({...})
setState({...})
setState({...})
```

Better:

```jsx
dispatch({ type: "ADD" });
```



### Internals

Similar to `useState`.

Difference:

React calls reducer to compute next state.



### Benefits

- centralized transitions
- predictable updates
- easier debugging
- scalable logic



## 16. Hook Execution Order

Hooks rely entirely on execution order.

React internally does:

```txt
Hook 1
Hook 2
Hook 3
```

NOT names.

This is why hooks cannot be conditional.



### Important Understanding

React does NOT attach hooks to variables.

This:

```jsx
const [count]
```

is irrelevant internally.

Only call position matters.



## 17. Hook Storage Internals (Fiber + Linked List)

One of the most advanced hooks topics.

Each component has Fiber node.

Fiber stores linked list of hooks.

Simplified:

```js
Fiber.memoizedState → Hook1 → Hook2 → Hook3
```

Each hook contains:

```js
{
  memoizedState,
  queue,
  next
}
```

During rendering React walks linked list sequentially.



### Why Linked List?

Hooks count is dynamic.

Linked list allows efficient insertion/traversal.



## 18. Batching with Hooks

React batches multiple updates.

```jsx
setA(1);
setB(2);
```

React performs single rerender.



### Why Batching Exists

Without batching:

```txt
render 1
render 2
render 3
```

Very expensive.



### React 18 Automatic Batching

Before React 18:

- only React events batched

Now:

- promises
- timeouts
- async callbacks
- native events

also batch automatically.



## 19. Concurrent Rendering + Hooks

Concurrent rendering allows interruptible rendering.

React can:

- pause rendering
- resume later
- prioritize urgent updates

Hooks had to be redesigned carefully for this.



### Important Principle

Render phase can restart.

Therefore hooks must remain pure.

Bad:

```jsx
useMemo(() => {
  mutateSomething();
}, []);
```

Concurrent rendering may run multiple times.



### Transition Updates

```jsx
startTransition(() => {
  setSearch(query);
});
```

Marks low-priority updates.



## 20. Strict Mode & Hooks Behavior

Strict Mode intentionally double-invokes renders in development.

Why?

To detect unsafe side effects.

Example:

```jsx
useEffect(() => {
  console.log("effect");
}, []);
```

May run twice in development.



### Important Clarification

Production does NOT double-run.

Strict mode simulates remounting behavior.



### What It Detects

- impure rendering
- missing cleanup
- unsafe mutations



## 21. Advanced Hooks

Important advanced hooks:

### useLayoutEffect

Runs synchronously before browser paint.

Useful for measurements.



### useImperativeHandle

Customizes exposed ref API.



### useTransition

Marks non-urgent updates.



### useDeferredValue

Defers expensive rendering.



### useSyncExternalStore

Designed for external store consistency.

Important for concurrent rendering safety.



### useInsertionEffect

Used mainly by CSS-in-JS libraries.

Runs before layout effects.



## 22. Hook Performance Optimization

Most performance problems are NOT solved with memoization.

Real optimizations:

- reduce unnecessary renders
- split components
- colocate state
- virtualize large lists
- avoid giant contexts



### Expensive Patterns

#### Derived state in effects

Bad:

```jsx
useEffect(() => {
  setFiltered(data.filter(...));
}, [data]);
```

Better:

```jsx
const filtered = useMemo(...)
```



### Overusing Context

Large global context causes rerender storms.



## 23. Common Hooks Pitfalls

### Infinite Effects

```jsx
useEffect(() => {
  setState(x);
}, [x]);
```

Can create loops.



### Missing Dependencies

Leads to stale closures.



### Over-Memoization

Too much `useMemo/useCallback` harms readability and performance.



### State Duplication

```jsx
const [filtered, setFiltered]
```

when value can be derived.



### Effect Abuse

Effects should synchronize external systems.

Not replace normal rendering logic.



## 24. Senior-Level Hooks Patterns



### State Colocation

Keep state closest to usage.

Avoid unnecessary lifting.



### Reducer + Context Architecture

Common scalable pattern.

```txt
Context → dispatch
Reducer → state transitions
```



### Headless Hooks

Hooks containing logic only.

UI separated completely.

Example:

```jsx
useModal()
```



### Resource Hooks

Encapsulate:

- fetching
- caching
- retry logic
- loading state



### Subscription Hooks

Encapsulate event listeners safely.



### Controlled vs Uncontrolled Hooks

Senior engineers carefully decide:

- render-driven state
- mutable refs
- external stores

based on performance needs.



## Final Mental Model

Hooks are not magic functions.

They are:

> Ordered state containers attached to Fiber nodes during rendering.

Everything about hooks becomes easier once you understand:

- renders are snapshots
- hooks depend on order
- React stores hooks in linked lists
- effects synchronize external systems
- memoization is reference optimization
- concurrent rendering requires purity
- React rerenders entire component functions repeatedly