## React Hooks Deep Dive

## Topics Covered

- [1. Why Hooks Exist](#1-why-hooks-exist)
- [2. Rules of Hooks](#2-rules-of-hooks)
- [3. useState Deep Dive](#3-usestate-deep-dive)
- [4. useEffect Deep Dive](#4-useeffect-deep-dive)
- [5. Effect Cleanup & Dependencies](#5-effect-cleanup--dependencies)
- [6. Stale Closures](#6-stale-closures)
- [7. useRef](#7-useref)
- [8. useMemo](#8-usememo)
- [9. useCallback](#9-usecallback)
- [10. useContext](#10-usecontext)
- [11. Custom Hooks](#11-custom-hooks)
- [12. Common Pitfalls](#12-common-pitfalls)

---

## 1. Why Hooks Exist

**Interview Focus:** Extremely common question - "Why did React introduce Hooks?"

Before Hooks, React had a hard split: class components handled state and lifecycle, while function components were just for rendering UI. Classes brought problems: `this` binding confusion, logic scattered across lifecycle methods, no good way to reuse stateful logic without HOCs or render props, and large components that became impossible to refactor.

Hooks solved all of this by letting function components have state, effects, and context while grouping logic by feature instead of lifecycle phase.

```jsx
// Before: duplicated logic across lifecycle methods
componentDidMount() { fetchUser(); }
componentDidUpdate() { fetchUser(); }

// After: grouped by what it does
useEffect(() => { fetchUser(); }, []);
```

**What hooks fundamentally are:** A way for React to attach persistent stateful memory to function components. Normally functions lose all variables after execution. Hooks give React-controlled persistent storage that survives across renders.

---

## 2. Rules of Hooks

**Interview Focus:** Critical understanding - often asked in both behavioral and practical questions.

Hooks must follow strict ordering rules because React tracks them by call sequence, not by name.

**Rule 1: Only call hooks at the top level** - never inside conditions, loops, or nested functions.

```jsx
// Bad
if (show) { useEffect(() => {}); }

// Good
useEffect(() => { if (show) { /* logic */ } }, [show]);
```

React stores hooks as a linked list. Each render, it walks the list in order. If a hook disappears conditionally, the positions shift and React attaches the wrong state to the wrong hook.

**Rule 2: Only call hooks in React functions** - function components or custom hooks only. React needs the render context to connect hooks to the Fiber tree.

---

## 3. useState Deep Dive

**Interview Focus:** Understanding functional updates and batching is commonly tested.

```jsx
const [count, setCount] = useState(0);
```

React creates a hook object with a state snapshot and an update queue. State updates are queued, not immediate. When you call `setCount(count + 1)`, React schedules a re-render.

**Functional updates** solve the stale value problem:

```jsx
// Both use the same stale value - doesn't work as expected
setCount(count + 1);
setCount(count + 1);

// React applies these sequentially, so they compound
setCount(c => c + 1);
setCount(c => c + 1);
```

**Why immutability matters:** React's reconciliation depends on reference comparison (`oldObj !== newObj`). If you mutate an object instead of creating a new one, React can't detect the change.

> For how React's rendering and batching work, see File 01: React Rendering & Internals.

---

## 4. useEffect Deep Dive

**Interview Focus:** Most misunderstood hook - frequently tested in practical scenarios.

`useEffect` is for synchronizing React with external systems: API calls, subscriptions, timers, DOM event listeners. It's NOT for deriving UI state from other state.

```jsx
useEffect(() => {
  console.log("DOM is updated");
});
```

Effects run after commit - React guarantees the DOM exists and the browser has painted before your effect fires. This keeps rendering pure.

**Why effects exist:** React rendering must stay side-effect-free. If you mutate `localStorage` during render, React may run that code multiple times, causing bugs. Effects separate side effects from rendering.

**Timing sequence:**
1. Render phase (pure)
2. Commit phase (DOM mutations)
3. Browser paint
4. Effects run (`useEffect`)

`useLayoutEffect` runs synchronously before paint, blocking the browser - only use it when you need to measure or mutate the DOM before the user sees it.

---

## 5. Effect Cleanup & Dependencies

**Interview Focus:** Understanding cleanup is essential - often tested with real-world scenarios like subscriptions.

```jsx
useEffect(() => {
  const id = setInterval(() => {}, 1000);
  return () => clearInterval(id); // cleanup
}, []);
```

**Effect Lifecycle:**
- Mount: effect runs
- Dependency change: cleanup runs, then new effect runs
- Unmount: cleanup runs one last time

Cleanup prevents duplicate subscriptions, memory leaks, and stale timers.

**Dependency Array:**

React compares dependencies using `Object.is()` (shallow equality). If any dependency changed, the effect reruns.

```jsx
useEffect(() => {}, [count]); // reruns when count changes
```

**Critical mindset:** The dependency array is not "when should this effect run?" It's "what values does this effect depend on?" If your effect uses `count`, it depends on `count` - period. Leaving it out creates stale closure bugs.

---

## 6. Stale Closures

**Interview Focus:** One of the most important hook concepts - frequently appears in senior interviews.

**First: what is a closure?** In JavaScript, when a function is created, it "closes over" the variables that existed at the time it was created. It captures a snapshot of those variables — not a live reference. Even if those variables change later, the function still holds the old version.

```js
function makeCounter() {
  let count = 0;
  return function() {
    console.log(count); // captures count from outer scope
  };
}
```

In React, every render creates a new scope with new variable values. Functions created inside that render (like effect callbacks) capture those values at that moment.

```jsx
useEffect(() => {
  setInterval(() => {
    console.log(count); // always logs 0
  }, 1000);
}, []);
```

Why? The effect runs once on mount and captures the `count` from that render (0). The interval keeps running with that captured value forever.

**Every render creates a new scope:**

```
Render 1 → count = 0
Render 2 → count = 1
Render 3 → count = 2
```

Effects belong to specific render snapshots. Closures preserve the variables from that snapshot.

**Solutions:**

1. Add the dependency - effect will re-create the interval with the fresh value
2. Use functional updates - no need to reference `count`: `setCount(c => c + 1)`
3. Use refs - mutable container that doesn't trigger re-renders: `ref.current = count`

---

## 7. useRef

**Interview Focus:** Understanding refs vs state is commonly tested.

`useRef` creates a mutable container that persists across renders:

```jsx
const ref = useRef(initialValue);
// Internally: { current: initialValue }
```

React preserves the same object between renders. Mutating `ref.current` does NOT trigger a re-render.

**Use cases:**
- DOM access: `<input ref={inputRef} />`
- Storing previous values
- Mutable timers or IDs
- Avoiding stale closures (store the latest value in a ref)

**forwardRef pattern** - for passing refs through components:

```jsx
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});
```

---

## 8. useMemo

**Interview Focus:** Understanding when to use memoization is a common senior-level question.

`useMemo` caches a computed value:

```jsx
const value = useMemo(() => expensiveCalculation(x), [x]);
```

React stores the result and dependencies. If dependencies haven't changed, React returns the cached value instead of re-computing.

**Important:** `useMemo` is a performance hint, not a semantic guarantee. React may discard the cache at any time. Don't rely on it for correctness - only for performance.

**When to use:**
- Actually expensive calculations
- Stable references to prevent child re-renders
- Values passed as dependencies to effects or other memoized hooks

**Don't overuse:** Memoization has a cost (memory, comparison overhead). Only use it when needed.

---

## 9. useCallback

**Interview Focus:** Often asked alongside React.memo - understanding reference equality is key.

`useCallback` is `useMemo` for functions:

```jsx
const fn = useCallback(() => {}, [dep]);
// Equivalent to: useMemo(() => fn, [dep])
```

Functions are recreated every render, which means new references. If you pass a new function reference to a memoized child, it rerenders even though the function body is identical. `useCallback` returns the same reference when dependencies haven't changed.

**Misconception:** `useCallback` doesn't prevent the function from being created - it's still defined every render. React just returns the previous cached reference instead of the new one.

Useful with `React.memo`, in dependency arrays, or when passing callbacks to expensive child components.

---

## 10. useContext

**Interview Focus:** Understanding Context performance issues is commonly tested.

`useContext` lets you read context values without prop drilling:

```jsx
const theme = useContext(ThemeContext);
```

When the context Provider's value changes, all components using `useContext` re-render.

**Common trap:**

```jsx
<Provider value={{ theme, user }} /> // new object every render
```

This triggers all consumers to re-render. Fix with `useMemo`:

```jsx
const value = useMemo(() => ({ theme, user }), [theme, user]);
```

**Best practices:**
- Split contexts - instead of one giant context, use separate contexts for theme, auth, settings
- Memoize context values to prevent unnecessary re-renders

> For deep dive on Context API and advanced patterns, see File 04: State Management.

---

## 11. Custom Hooks

**Interview Focus:** Building custom hooks is a common practical interview question.

**Custom hooks are just functions that call other hooks.** They encapsulate reusable stateful logic.

```jsx
function useUser() {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetchUser().then(setUser);
  }, []);
  return user;
}
```

**More realistic example — useAsync for data fetching:**

This pattern replaces repetitive loading/error/data boilerplate that you'd otherwise write in every component:

```jsx
function useAsync(asyncFn, deps = []) {
  const [state, setState] = useState({
    status: "idle", // idle | loading | success | error
    data: null,
    error: null,
  });

  useEffect(() => {
    let cancelled = false;

    setState({ status: "loading", data: null, error: null });

    asyncFn()
      .then((data) => {
        if (!cancelled) setState({ status: "success", data, error: null });
      })
      .catch((error) => {
        if (!cancelled) setState({ status: "error", data: null, error });
      });

    return () => { cancelled = true; }; // prevent stale updates
  }, deps);

  return state;
}

// Usage
const { status, data: users, error } = useAsync(() => fetchUsers(), []);

if (status === "loading") return <Spinner />;
if (status === "error") return <ErrorMessage />;
return <UserList users={users} />;
```

This handles: loading state, error state, stale request cancellation, and clean re-fetching on dependency change — all in one reusable hook.

**The real power:** Encapsulation. Custom hooks hide complex logic (subscriptions, caching, fetching, synchronization) behind a simple interface. Hooks compose behavior vertically (stack functions that each add a capability), making reuse much cleaner than class inheritance.

**Best practices:**
- Always prefix with "use" for linting
- Return stable values or objects
- Keep hooks focused on one responsibility
- Document what the hook does and expects

**Linting:** `eslint-plugin-react-hooks` automatically enforces hook rules and catches missing dependencies in useEffect. It's installed by default in Create React App and Vite. Always have it enabled — it catches 90% of stale closure bugs before they happen.

---

## 12. Common Pitfalls

**Interview Focus:** Demonstrating awareness of common mistakes shows experience.

**1. Infinite effect loops:**

```jsx
useEffect(() => {
  setState(x);
}, [x]); // setState updates x, which triggers effect, which updates x...
```

**2. Missing dependencies** - leads to stale closures. Add all referenced values to the array or restructure your code.

**3. State duplication:**

```jsx
const [filtered, setFiltered] = useState([]);
```

If you can compute `filtered` from existing state, don't store it separately - derive it.

**4. Effect abuse** - effects are for synchronizing with external systems, not for deriving UI state. If you're calling `setState` in an effect to derive UI state, you're probably doing it wrong.

**5. Over-memoization** - wrapping everything in `useMemo`/`useCallback` adds cognitive overhead and comparison cost. Measure before you optimize.

**6. Using refs for state that should trigger renders** - if changing a value should update the UI, use `useState`, not `useRef`.

---

## Final Mental Model

Hooks are not magic. They are:

> Ordered state containers attached to components during rendering.

Everything becomes clearer once you internalize:

- **Renders are snapshots** - each render sees a frozen version of state
- **Hooks depend on order** - React matches calls to stored state by position
- **Effects synchronize with external systems** - not for deriving UI state
- **Memoization is for reference equality** - prevents unnecessary re-renders
- **Every render has its own props, state, and handlers** - closures capture values from their render

---

## React Compiler (Future of Memoization)

**What it is:** The React Compiler (previously called "React Forget") is an upcoming compiler that automatically adds memoization to your code at build time.

**Why it matters:** Today, you manually write `useMemo`, `useCallback`, and `React.memo` to prevent unnecessary re-renders. The React Compiler analyzes your code and inserts these automatically — meaning you won't need to write them manually.

**Status:** Available as of React 19 (experimental in some setups). If your team upgrades, a lot of manual memoization becomes unnecessary.

**Practical impact for interviews:** When asked about performance, mention that the React Compiler is changing this area — but also show you understand *why* memoization works (for interviewers on older codebases, manual memoization knowledge still matters).
