## State Management

## Topics Covered

- [1. Understanding State Types](#1-understanding-state-types)
- [2. useState Deep Dive](#2-usestate-deep-dive)
- [3. State Structure & Architecture](#3-state-structure--architecture)
- [4. Lifting State Up](#4-lifting-state-up)
- [5. useReducer Deep Dive](#5-usereducer-deep-dive)
- [6. Context API & Optimization](#6-context-api--optimization)
- [7. Redux Fundamentals](#7-redux-fundamentals)
- [8. Redux Toolkit (RTK)](#8-redux-toolkit-rtk)
- [9. Async State Management](#9-async-state-management)
- [10. Common Interview Problems](#10-common-interview-problems)

---

State management is one of the biggest differences between beginner React developers and senior React engineers. Most React interview problems eventually become state problems - not "How do I render this UI?" but rather where data should live, who owns it, what causes re-rendering, how updates flow, and what belongs in local state versus global state.

---

## 1. Understanding State Types

**Interview Focus:** Understanding different state types is fundamental - commonly asked in architecture discussions.

State is mutable data that changes over time and affects UI rendering. React UI is simply a function of state: `UI = f(state)`.

**Types of state:**

| Type | Purpose | Examples |
|---|---|---|
| **Local State** | Owned by one component | Dropdowns, inputs, toggles |
| **Shared State** | Needed by multiple components | Current user, cart, theme |
| **Server State** | Data from backend | API responses, cached requests |
| **Derived State** | Calculated from existing state | Fullname from first/last name |

**Important principle:** State should be minimal, normalized, predictable, colocated, and treated as a single source of truth. Bad state architecture causes bugs, duplication, inconsistent UI, and performance issues.

---

## 2. useState Deep Dive

**Interview Focus:** Functional updates and batching are commonly tested.

`useState` is React's simplest state primitive:

```jsx
const [state, setState] = useState(initialValue);
```

**Key concepts:**
- Functional updates for sequential operations: `setCount(c => c + 1)`
- Lazy initialization for expensive calculations: `useState(() => expensiveCompute())`
- State is snapshot-based within a render - multiple updates in the same render use the same snapshot

> For complete useState internals including hook ordering and closures, see File 02: Hooks Deep Dive.

State updates trigger React's rendering process. React uses automatic batching to combine multiple state updates into a single render.

> For complete rendering internals including reconciliation, Fiber architecture, and batching, see File 01: React Rendering & Internals.

---

## 3. State Structure & Architecture

**Interview Focus:** Senior-level question - "How would you structure state for a complex feature?"

Bad state architecture causes bugs, duplication, inconsistent UI, and performance issues.

**Keep state minimal** - don't store derived values:

```jsx
// Bad
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);

// Good
const total = items.reduce((sum, item) => sum + item.price, 0);
```

**Normalize state** - deeply nested structures cause problems:

```js
// Bad
posts = [
  { id: 1, author: { id: 5, name: "John" } }
]

// Good (normalized)
{
  posts: { byId: {}, allIds: [] },
  users: { byId: {} }
}
```

Redux commonly uses normalized state because it enables smaller updates, fewer object recreations, and better memoization.

**Colocate state** - keep state close to where it's used. Don't use global state for a modal toggle - use local component state. Over-globalization creates unnecessary re-renders.

**Single source of truth** - never duplicate ownership. If the parent and child both store the same data, synchronization bugs will happen.

---

## 4. Lifting State Up

**Interview Focus:** Basic React pattern - often appears in practical coding questions.

When two components need the same data, move state to their common parent:

```jsx
function Parent() {
  const [value, setValue] = useState("");

  return (
    <>
      <Input value={value} onChange={setValue} />
      <Preview value={value} />
    </>
  );
}
```

React uses unidirectional data flow - data flows from parent to child. The parent becomes the source of truth, which improves predictability and debugging.

**Problem:** Excessive lifting creates prop drilling, unnecessary renders, and deeply coupled components. This led to Context, Redux, and Zustand.

---

## 5. useReducer Deep Dive

**Interview Focus:** Often compared with useState - understanding when to use each is important.

`useReducer` is state management using reducer functions. Complex state becomes hard to manage with `useState` - logic spreads everywhere, and maintaining consistency becomes difficult.

```jsx
function reducer(state, action) {
  switch(action.type) {
    case "increment":
      return { count: state.count + 1 };
    default:
      return state;
  }
}

const [state, dispatch] = useReducer(reducer, initialState);
```

**Why reducers scale better:** They centralize updates, making them predictable, testable, and easier to debug. Redux uses the same architecture.

**Reducers must be pure** - mutation breaks everything:

```js
// Bad
state.count++;

// Good
return { ...state, count: state.count + 1 };
```

Use `useReducer` for complex transitions, multiple related fields, and state machines - not simple toggles.

**Discriminated unions in actions** — a "discriminated union" means a TypeScript union type where each variant has a unique identifying field (like `type`). TypeScript can look at that field and narrow down exactly which variant you're dealing with:

```ts
// Each action variant is "discriminated" by its unique `type` field
type Action =
  | { type: "increment" }               // no payload
  | { type: "decrement" }               // no payload
  | { type: "set"; payload: number };   // has payload

// TypeScript knows: if type === "set", then payload exists
dispatch({ type: "set", payload: 5 }); // safe, autocompleted
dispatch({ type: "set" });             // TypeScript error — missing payload
```

This makes reducers fully type-safe and prevents dispatching invalid actions.

---

## 6. Context API & Optimization

**Interview Focus:** Understanding Context performance issues is critical - commonly tested in senior interviews.

Context solves prop drilling by allowing components deep in the tree to access data without passing props through every layer:

```jsx
const UserContext = createContext();

<UserContext.Provider value={user}>
  <App />
</UserContext.Provider>

// Deep in tree
const user = useContext(UserContext);
```

**Internals:** React tracks which components consume context. When the context value changes, all consumers re-render - this is critical to understand.

**Major performance problem:** Creating a new object every render triggers all consumers:

```jsx
<UserContext.Provider value={{ user }}> {/* new object every render */}
```

**Solutions:**

1. Memoize the value:
```jsx
const value = useMemo(() => ({ user }), [user]);
```

2. Split contexts - instead of one giant `GlobalContext`, use separate contexts for theme, auth, and settings. This isolates updates to smaller subtrees.

**Critical insight:** Context is not state management - it's dependency injection. State still lives somewhere else (usually in `useState` or `useReducer`). This is a huge misconception in interviews.

**Context vs Redux:** Context transports data; Redux is a centralized state container with predictable updates, middleware, time travel debugging, and devtools. They have very different responsibilities.

---

## 7. Redux Fundamentals

**Interview Focus:** Understanding when to use Redux vs alternatives is commonly asked.

Redux solves predictable global state with four core principles: single store, immutable updates, actions describe changes, and reducers compute new state.

**Redux flow:**

```txt
UI → dispatch(action) → reducer → new state → subscribers notified → UI updates
```

**Store:** Centralized state container.

**Actions:** Plain objects describing what happened:

```js
{ type: "cart/add", payload: item }
```

**Reducers:** Pure functions that compute new state:

```js
function reducer(state, action) {
  return newState;
}
```

**Why immutability matters:** Redux compares references to detect changes. Mutation breaks change detection, memoization, and time travel debugging. React depends on reference equality for optimization (`oldObj !== newObj`).

Redux optimizes for predictability, debugging, tooling, and enterprise scale - not minimal boilerplate.

---

## 8. Redux Toolkit (RTK)

**Interview Focus:** Modern Redux standard - most companies use RTK now.

Redux Toolkit is the official modern Redux. Most Redux boilerplate disappeared because of RTK.

**Old Redux problems:** Too much code for action types, action creators, switch statements, and manual immutable updates.

**RTK solution:**

```js
const slice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment(state) {
      state.count++; // looks like mutation, but Immer handles it
    }
  }
});
```

**Why mutation works:** RTK uses Immer, which creates immutable copies internally. You write `state.count++` and Immer produces a new state object without actual mutation.

**createSlice:** Combines reducer, actions, and action types into one API - huge simplification.

**configureStore:** Provides better defaults including Redux DevTools, middleware, thunk, and immutability checks.

RTK became standard because classic Redux had excessive ceremony, poor DX, and repetitive patterns.

**RTK Query** — built-in data fetching and caching layer inside RTK. Think of it as React Query but integrated directly into Redux:

```js
// Define an API slice
const userApi = createApi({
  reducerPath: "userApi",
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  endpoints: (builder) => ({
    getUsers: builder.query({ query: () => "/users" }),
    createUser: builder.mutation({
      query: (body) => ({ url: "/users", method: "POST", body }),
    }),
  }),
});

export const { useGetUsersQuery, useCreateUserMutation } = userApi;

// Usage in component
const { data, isLoading } = useGetUsersQuery();
```

RTK Query auto-handles: caching, loading/error states, cache invalidation on mutations, request deduplication, and background refetching — very similar to TanStack Query but integrated into Redux.

---

## 9. Async State Management

**Interview Focus:** Understanding async state patterns is essential for real-world apps.

Frontend apps constantly handle async state: API requests, uploads, pagination, retries, and caching. Typical async states include idle, loading, success, and error.

**Common mistake:**

```jsx
const [data, setData] = useState([]);
```

This has no loading or error handling. Better:

```jsx
{
  data,
  loading,
  error
}
```

**Redux Thunk:** Allows async logic in Redux. Thunks are functions that dispatch actions:

```js
export const fetchUsers = () => async dispatch => {
  dispatch(start());
  const data = await api.get();
  dispatch(success(data));
};
```

**Why async middleware exists:** Redux reducers must stay synchronous and pure. Middleware handles side effects like API calls, logging, and analytics.

**Common async problems:**
- Race conditions - older request finishes later, overwriting fresh data
- Duplicate requests - same request triggered multiple times without deduplication
- Waterfall fetching - sequential requests multiply latency

**Modern solution:** Libraries like TanStack Query and RTK Query handle caching, fetching, deduplication, invalidation, and retries automatically, drastically reducing boilerplate.

---

## 10. Common Interview Problems

**Interview Focus:** These questions appear in almost every React interview.

**"Where should this state live?"**
Most common interview question. Answer depends on: who needs it, lifespan, sharing requirements, and persistence needs. Local state for component-specific data, lifted state for sibling sharing, Context for deep tree sharing, Redux/Zustand for complex global coordination.

**"When should you use Context vs Redux?"**
Context is good for theme, auth, and locale. Redux is good for complex global coordination, caching, enterprise workflows, and debugging-heavy apps. Context causes all consumers to re-render; Redux has selective subscriptions via selectors.

**"Why does the component re-render?"**
Expected answers: state change, parent render, context change, or store subscription update.

**"Why is immutability important?"**
React and Redux rely on reference equality. Mutation breaks memoization, re-render optimization, and debugging. Without immutability, React can't detect what changed.

**"Why can stale closures happen?"**
Functions capture old render snapshots. Example:

```jsx
setTimeout(() => {
  console.log(count); // logs old count
}, 1000);
```

Fix by adding dependencies to useEffect, using functional updates, or using refs.

**"Why does Context cause performance problems?"**
All consumers re-render when the provider value changes, even if they only use a small slice of the context. Solution: split contexts or use selectors with external state libraries.

**"What is the difference between client state and server state?"**
Client state is UI-owned (modals, forms, toggles). Server state is backend-owned (API data, cache, synchronization). This distinction is extremely important in senior interviews. Server state should often be handled by libraries like React Query, not stored in Redux.

---

## Final Mental Model

React state management is fundamentally about:

**Who owns the data?** Local component, shared parent, context provider, or global store?

**Who can update it?** Single owner, multiple dispatchers, or external systems?

**Who depends on it?** Single component, component tree, or entire app?

**How are updates propagated?** Props, context, subscriptions, or events?

**How do we minimize rendering cost?** Colocate state, memoize values, split contexts, use selectors?

The best React engineers design clean state ownership, predictable update flow, scalable architecture, minimal rendering cost, and maintainable data flow.

---

## Zustand & Jotai — Lightweight Alternatives

**Why they exist:** Redux is powerful but heavy. Context has performance limitations. Zustand and Jotai sit in between — simple global state without Redux's boilerplate.

**Zustand** — minimal store, no providers needed:

```js
import { create } from "zustand";

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

// Use directly in any component — no Provider wrapping needed
function Counter() {
  const { count, increment } = useStore();
  return <button onClick={increment}>{count}</button>;
}
```

**Benefits over Redux:** No actions, no reducers, no Provider setup. Just a store with state and setters. Still supports devtools and middleware.

**Benefits over Context:** Components only re-render when their selected slice changes (unlike Context where everyone re-renders on any change).

**Jotai** — atomic state (like Recoil). Break state into small independent atoms:

```js
import { atom, useAtom } from "jotai";

const countAtom = atom(0);

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**When to use what:**

| Library | Best For |
|---|---|
| `useState` + `useReducer` | Local/feature state |
| Context | Rarely-changing global data (theme, auth) |
| Zustand | Simple global state, small-medium apps |
| Redux Toolkit | Complex global state, large enterprise apps with strict patterns |
| TanStack Query / RTK Query | Server/async state (API data) |
