# Hooks Deep Dive

## 1. Introduction to React Hooks

Hooks are special functions introduced in React 16.8 that allow functional components to use:

- state
- lifecycle features
- refs
- context
- side effects
- reusable logic

Before Hooks, these features were mostly available only in class components.

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

Hooks changed React development completely because now:

- functional components became powerful
- code became easier to reuse
- logic became easier to organize
- class component complexity reduced


## 2. Why Hooks Were Introduced

Before Hooks, React had several major problems.

### Problem 1 — Logic Reuse Was Difficult

People used:

- HOCs (Higher Order Components)
- Render Props
- Mixins (old)

These patterns often created:

- wrapper hell
- deeply nested trees
- hard debugging


### Problem 2 — Class Components Became Huge

Class components mixed unrelated logic together.

```jsx
componentDidMount()
componentDidUpdate()
componentWillUnmount()
```

Related logic was split across lifecycle methods.


### Problem 3 — Classes Were Confusing

Developers struggled with:

- `this`
- binding methods
- lifecycle behavior
- stale state issues

Hooks simplified all of this.


## 3. Rules of Hooks

Hooks follow strict rules because React internally depends on hook order.

### Rule 1 — Only Call Hooks at Top Level

❌ Wrong

```jsx
if (show) {
  useEffect(() => {});
}
```

✅ Correct

```jsx
useEffect(() => {
  if (show) {
    // logic
  }
}, [show]);
```


### Rule 2 — Only Call Hooks Inside React Functions

Hooks can only run inside:

- React components
- custom hooks

❌ Wrong

```jsx
function test() {
  useState(0);
}
```


### Why These Rules Exist

React identifies hooks by position.

```jsx
useState();
useEffect();
useMemo();
```

React assumes:

- first hook = state
- second hook = effect
- third hook = memo

If order changes between renders, React breaks internally.


## 4. Hook Call Order Mechanism

React does NOT identify hooks by variable name.

It identifies hooks by execution order.

```jsx
function App() {
  const [count] = useState(0);
  const [name] = useState("React");
}
```

Internally React stores:

```js
[
  { state: 0 },
  { state: "React" }
]
```

During next render:

- first `useState` → gets `0`
- second `useState` → gets `"React"`

If conditional hooks appear:

```jsx
if (condition) {
  useState();
}
```

Hook ordering shifts.

This causes:

- wrong state mapping
- corrupted hook behavior
- runtime errors


## 5. How Hooks Work Internally

Internally React maintains a linked list of hooks for every component fiber.

Each hook stores:

- memoized state
- queue
- dependencies
- next hook reference

Simplified idea:

```js
fiber.memoizedState = {
  state: 0,
  next: {
    state: "hello",
    next: null
  }
}
```

React uses:

- current fiber
- current hook pointer

during rendering.

This is why hooks are extremely fast.


## 6. useState Deep Dive

`useState` adds local component state.

```jsx
const [count, setCount] = useState(0);
```

### State Updates Are Scheduled

React does NOT immediately update state.

```jsx
setCount(count + 1);
console.log(count);
```

Old value prints because React schedules re-render.


### State Updates Trigger Re-render

When state changes:

1. update gets queued
2. React schedules render
3. component re-executes
4. new UI generated

### State Is Immutable

❌ Wrong

```jsx
user.name = "John";
```

✅ Correct

```jsx
setUser({
  ...user,
  name: "John"
});
```

React detects changes using references.


## 7. Functional State Updates

Functional updates solve stale state issues.

❌ Problem

```jsx
setCount(count + 1);
setCount(count + 1);
```

Expected: `+2`

Actual: `+1`

Because both use same stale value.



✅ Correct

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

Now React uses latest queued state.


### Production Insight

Use functional updates whenever next state depends on previous state.

Especially important in:

- intervals
- async updates
- event batching


## 8. Lazy State Initialization

Sometimes initial state is expensive.

❌ Bad

```jsx
const [data] = useState(expensiveFunction());
```

Runs every render.



✅ Better

```jsx
const [data] = useState(() => expensiveFunction());
```

Runs only on initial render.


### Use Cases

- parsing large JSON
- localStorage reads
- heavy calculations


## 9. useEffect Deep Dive

`useEffect` handles side effects.

Examples:

- API calls
- subscriptions
- timers
- DOM updates
- event listeners

```jsx
useEffect(() => {
  console.log("Mounted");
}, []);
```


### Effects Run After Render

React rendering must stay pure.

Effects run after DOM commit phase.

This prevents blocking rendering.


## 10. Effect Lifecycle

Effects have 3 phases.

### Mount

```jsx
useEffect(() => {
  console.log("Mounted");
}, []);
```


### Update

```jsx
useEffect(() => {
  console.log("Updated");
}, [count]);
```

Runs whenever dependency changes.


### Cleanup / Unmount

```jsx
useEffect(() => {
  const id = setInterval(() => {}, 1000);

  return () => clearInterval(id);
}, []);
```

Cleanup prevents memory leaks.


## 11. Dependency Array Behavior

Dependency array controls when effect re-runs.


### No Dependency Array

```jsx
useEffect(() => {});
```

Runs after every render.


### Empty Dependency Array

```jsx
useEffect(() => {}, []);
```

Runs once after mount.


### Specific Dependencies

```jsx
useEffect(() => {}, [count]);
```

Runs when `count` changes.


### Important

React uses shallow comparison (`Object.is`).

Objects/functions create new references every render.

This often causes unnecessary effect execution.


## 12. Cleanup Functions

Cleanup runs:

- before next effect
- during unmount

```jsx
useEffect(() => {
  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```


### Why Cleanup Matters

Without cleanup:

- memory leaks
- duplicate listeners
- stale subscriptions
- unexpected behavior

occur in production.


## 13. Stale Closures

Closures capture old values.

This is one of the most important React interview topics.


### Problem Example

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(id);
}, []);
```

`count` remains stale because effect captured initial render value.


### Solutions

#### Add Dependency

```jsx
useEffect(() => {}, [count]);
```


#### Use Ref

```jsx
const countRef = useRef(count);

countRef.current = count;
```


#### Functional Updates

```jsx
setCount(prev => prev + 1);
```


## 14. Infinite Re-render Problems

A component can accidentally re-render forever.


### Common Cause

```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

Effect updates state → state triggers effect again.

Infinite loop.


### Fix

Conditionally update state.

```jsx
useEffect(() => {
  if (count < 5) {
    setCount(count + 1);
  }
}, [count]);
```


## 15. Race Conditions in Effects

Async requests may finish in unexpected order.


### Problem

User searches quickly:

- request A starts
- request B starts
- B finishes first
- A finishes later and overwrites data

Old response wins incorrectly.


### Solution

Use cleanup or AbortController.

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(url, {
    signal: controller.signal
  });

  return () => controller.abort();
}, [url]);
```


## 16. useRef Deep Dive

`useRef` stores mutable values without causing re-render.

```jsx
const ref = useRef(null);
```


### Common Uses

#### DOM Access

```jsx
inputRef.current.focus();
```


#### Persist Values

```jsx
const timerRef = useRef();
```


#### Avoid Re-renders

Refs update without UI rendering.


## 17. Ref vs State

| Ref | State |
|---|---|
| No re-render | Causes re-render |
| Mutable | Immutable |
| Used for persistence | Used for UI |
| Fast updates | UI-driven updates |


### Rule

If UI should update → use state.

If value only needs persistence → use ref.


## 18. useMemo Deep Dive

`useMemo` memoizes expensive calculations.

```jsx
const sorted = useMemo(() => {
  return items.sort();
}, [items]);
```


### Why Needed

Without memoization:

- expensive calculations repeat
- CPU usage increases
- rendering slows


### Important

`useMemo` is a performance optimization.

NOT a semantic guarantee.

React may discard memoized values.


## 19. useCallback Deep Dive

`useCallback` memoizes functions.

```jsx
const handleClick = useCallback(() => {
  console.log("Clicked");
}, []);
```


### Why Needed

Functions get recreated every render.

This breaks referential equality.

Especially problematic when:

- passing callbacks to memoized children
- using callbacks in dependency arrays


## 20. useMemo vs useCallback

| Feature | useMemo | useCallback |
|---|---|---|
| Purpose | Memoizes computed values | Memoizes functions |
| Returns | Cached result/value | Cached function |
| Common Use Case | Expensive calculations | Stable callback references |
| Prevents | Recomputing values | Recreating functions |
| Helps With | Performance optimization | Referential equality |
| Typical Usage | Derived data | Passing callbacks to children |


### Internal Reality

```jsx
useCallback(fn, deps)
```

is basically:

```jsx
useMemo(() => fn, deps)
```



## 21. Referential Equality and Hooks

Objects/functions are compared by reference.

```jsx
{} !== {}
```

```jsx
() => {} !== () => {}
```

This affects:

- dependency arrays
- React.memo
- optimization behavior



### Example

```jsx
useEffect(() => {}, [{}]);
```

Runs every render because object reference changes.



### Fix

```jsx
const obj = useMemo(() => ({}), []);
```


## 22. React.memo with Hooks

`React.memo` prevents unnecessary child renders.

```jsx
export default React.memo(Component);
```



### Important

Memo only works if props stay referentially equal.

This is why `useCallback` often pairs with `React.memo`.



### Example

```jsx
const onClick = useCallback(() => {}, []);

<Child onClick={onClick} />
```



## 23. useReducer Deep Dive

`useReducer` manages complex state transitions.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```



### Reducer Pattern

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };

    default:
      return state;
  }
}
```



### Why useReducer Exists

Better for:

- complex state logic
- multiple related states
- predictable transitions



## 24. Complex State Management with useReducer

`useReducer` scales better than many `useState` calls.



### Good Use Cases

- forms
- authentication
- shopping cart
- workflows
- state machines



### Production Insight

Reducers centralize logic.

This improves:

- debugging
- testing
- maintainability



## 25. useContext Deep Dive

`useContext` consumes shared data.

```jsx
const value = useContext(AuthContext);
```

Removes prop drilling.



### Common Uses

- theme
- auth
- language
- global preferences



## 26. Context Re-render Behavior

Very important senior-level topic.

When context value changes:

ALL consuming components re-render.

Even if component uses only small part of value.



### Example

```jsx
<AuthContext.Provider value={{ user, theme }}>
```

Changing `theme` re-renders all consumers.



## 27. Context Performance Problems

Large contexts become performance bottlenecks.



### Common Fixes

#### Split Contexts

```jsx
ThemeContext
UserContext
```



#### Memoize Value

```jsx
const value = useMemo(() => ({
  user
}), [user]);
```



#### Use Selector Libraries

Examples:

- Zustand
- Redux
- use-context-selector



## 28. useLayoutEffect vs useEffect

Both are effects but timing differs.



### useEffect

Runs after paint.

UI appears first.



### useLayoutEffect

Runs before paint.

Blocks browser painting.



### Use Cases

#### useEffect

- API calls
- subscriptions
- logging

#### useLayoutEffect

- DOM measurements
- tooltip positioning
- preventing layout flicker



## 29. useImperativeHandle

Used with `forwardRef`.

Allows parent to expose controlled APIs.

```jsx
useImperativeHandle(ref, () => ({
  focus() {
    inputRef.current.focus();
  }
}));
```



### Why Useful

Prevents exposing entire DOM node.

Better encapsulation.



## 30. useId

Generates stable unique IDs.

```jsx
const id = useId();
```

Useful for:

- accessibility
- form labels
- SSR consistency



### SSR Importance

Prevents hydration mismatches.



## 31. Custom Hooks Architecture

Custom hooks extract reusable logic.

```jsx
function useFetch(url) {
}
```



### Benefits

- reusable logic
- cleaner components
- separation of concerns
- better testing



## 32. Hook Composition Patterns

Hooks can compose other hooks.

```jsx
function useAuth() {
  const user = useUser();
  const token = useToken();
}
```

Composition is a core React philosophy.



## 33. Sharing Logic with Custom Hooks

Custom hooks share behavior without changing component hierarchy.

Better than:

- HOCs
- render props

in many modern cases.



### Examples

- fetching
- forms
- debounce
- localStorage sync
- websocket handling



## 34. Hook Performance Pitfalls

Hooks can hurt performance when overused.



### Common Problems

#### Excessive useMemo/useCallback

Memoization itself has cost.



#### Huge Dependency Arrays

Complex dependencies create maintenance problems.



#### Context Over-rendering

Large contexts re-render entire trees.



#### Unnecessary Effects

Many effects can be replaced with pure rendering logic.



## 35. Hook Anti-patterns



### Copying Props into State

❌ Bad

```jsx
const [value, setValue] = useState(props.value);
```

Creates sync issues.



### Using Effects for Derived State

❌ Bad

```jsx
useEffect(() => {
  setFiltered(items.filter(Boolean));
}, [items]);
```

✅ Better

```jsx
const filtered = items.filter(Boolean);
```



### Overusing useEffect

Many beginners use effects for everything.

Effects are mainly for synchronization with external systems.



## 36. Debugging Hook Issues



### Common Debugging Areas

#### Missing Dependencies

Use ESLint hooks plugin.



#### Stale Closures

Check captured variables.



#### Infinite Loops

Check state updates inside effects.



#### Re-render Tracking

Use:

- React DevTools
- why-did-you-render
- React Profiler



## 37. Production-level Hook Best Practices



### Keep Components Pure

Rendering should stay predictable.



### Prefer Derived Values Over Extra State

Avoid unnecessary state duplication.



### Minimize Effects

Most logic does not need effects.



### Use Custom Hooks for Reuse

Encapsulate business logic.



### Optimize Only When Needed

Premature optimization creates complexity.



### Trust React First

Readable code is more important than micro-optimizations.



## 38. Common Senior-level Interview Questions



### Why must hooks be called in same order?

Because React internally maps hooks by execution order.



### Difference between useEffect and useLayoutEffect?

`useLayoutEffect` runs before paint and blocks rendering.

`useEffect` runs after paint.



### Why does stale closure happen?

Closures capture variables from old render scope.



### Why can dependency arrays cause bugs?

Because objects/functions change references every render.



### When should useReducer be preferred?

When state logic becomes complex and predictable transitions are needed.



### Why does context cause performance problems?

All consumers re-render when provider value changes.



### Difference between refs and state?

Refs persist values without re-rendering.

State triggers UI updates.



### Is useMemo guaranteed?

No.

React may discard memoized values.

It is only a performance optimization.



### Why are hooks powerful?

They allow:

- reusable logic
- cleaner architecture
- composition
- simpler stateful components
- better separation of concerns

without class complexity.