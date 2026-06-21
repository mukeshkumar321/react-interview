## Performance Optimization

## Topics Covered

- [1. Rendering Behavior](#1-rendering-behavior)
- [2. Re-render Prevention](#2-re-render-prevention)
- [3. Memoization Strategy](#3-memoization-strategy)
- [4. State Architecture](#4-state-architecture)
- [5. Context Optimization](#5-context-optimization)
- [6. Component Architecture for Performance](#6-component-architecture-for-performance)
- [7. Code Splitting & Bundle Optimization](#7-code-splitting--bundle-optimization)
- [8. Data Fetching Performance](#8-data-fetching-performance)
- [9. Large List & UI Performance](#9-large-list--ui-performance)
- [10. Profiling & Debugging](#10-profiling--debugging)
- [11. Concurrent Features](#11-concurrent-features)
- [12. Real Production Bottlenecks](#12-real-production-bottlenecks)
- [Performance Anti-patterns](#performance-anti-patterns)
- [Senior-level Performance Mindset](#senior-level-performance-mindset)
- [Common Senior-level Interview Questions](#common-senior-level-interview-questions)

---

Performance optimization in React is about **reducing unnecessary work**. React apps become slow because of unnecessary re-renders, large component trees, expensive calculations, poor state architecture, huge bundles, inefficient data fetching, and excessive context updates.

The goal isn't "making React faster everywhere." It's rendering less, calculating less, loading less, and updating only what changed.

---

## 1. Rendering Behavior

Understanding when and why React renders is the foundation of performance optimization.

**Core rule:** When a component updates, **all its children re-render by default** — even if they don't use the changed state.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <Child /> {/* rerenders when count changes, even though it doesn't use count */}
    </>
  );
}
```

**Common misconception:** "React only re-renders changed components." Not true. React re-renders downward from updated parents unless optimization (like `React.memo`) prevents it.

**Re-render ≠ DOM update.** React may re-run a component, compare the output, find no differences, and skip DOM mutations entirely.

**For complete rendering internals including reconciliation, Fiber architecture, and batching, see File 01: React Rendering & Internals.**

---

## 2. Re-render Prevention

**React.memo** memoizes components — React skips rendering if props are shallowly equal to the previous render:

```jsx
const Child = React.memo(function Child({ value }) {
  return <div>{value}</div>;
});
```

React compares references: `prevProps === nextProps`. Primitives work (`1 === 1`), but objects/functions don't (`{} === {}`). This causes re-renders:

```jsx
<Child config={{ dark: true }} /> // new object every render
```

Unstable references (objects, arrays, functions created inline) are a major source of unnecessary renders.

**useCallback** memoizes functions so they have a stable reference:

```jsx
const handleClick = useCallback(() => {
  doSomething();
}, []);
```

**Misconception:** `useCallback` doesn't stop the function from being created. The function is still defined every render — React just returns the cached reference when dependencies haven't changed.

**useMemo** caches expensive calculations:

```jsx
const filtered = useMemo(() => {
  return items.filter(item => item.active);
}, [items]);
```

Without memoization, the filtering runs every render. But memoization has its own cost (dependency comparison, memory, cache management), so don't memoize trivial work:

```jsx
const doubled = useMemo(() => count * 2, [count]); // unnecessary
```

**Optimization order:**

1. Measure performance
2. Identify bottlenecks
3. Fix architecture issues
4. Memoize hotspots only

---

## 3. Memoization Strategy

Memoization is one of the most misunderstood topics. Many developers wrap everything in `useMemo`/`useCallback`, which hurts readability more than it helps performance.

| Tool | Purpose |
|---|---|
| `React.memo` | Memoize component |
| `useMemo` | Memoize value |
| `useCallback` | Memoize function |

**Derived state:** Don't store values you can compute from existing state.

❌ Bad:
```jsx
const [filteredUsers, setFilteredUsers] = useState([]);
```

This creates synchronization problems, extra renders, and duplicate state.

✅ Better:
```jsx
const filteredUsers = useMemo(() => {
  return users.filter(user => user.active);
}, [users]);
```

**Referential equality:** React compares by reference, not deep value. Inline object literals break memoization:

```jsx
const style = { color: "red" }; // new object every render
```

Use `useMemo` only when the reference is passed to a memoized component or used in a dependency array.

**Selector pattern** (common in Redux/Zustand):

```js
// Bad: subscribes to entire store
const state = useStore();

// Good: subscribes only to user slice
const user = useStore(state => state.user);
```

Only re-renders when `user` changes, not when other parts of the store update.

---

## 4. State Architecture

Bad state architecture causes most large-scale React performance issues.

**State colocation** — keep state close to where it's used. If state for a modal lives at the app root, the entire app re-renders when the modal opens. Instead, let the modal manage its own state:

```jsx
<Modal /> {/* state is local */}
```

React updates the subtree below the changed state. Smaller subtree → fewer renders → better performance.

**Lifting state too high** is a common anti-pattern:

```
App (all state lives here)
 ├── Navbar
 ├── Sidebar
 └── Dashboard
```

Every state update triggers a full tree re-render.

**Normalize deeply nested state:**

❌ Bad:
```js
state.project.tasks[taskId].todos[todoId]
```

Problems: expensive immutable copies, unstable references, difficult updates.

✅ Better:
```js
{
  tasksById: { [id]: task },
  todosById: { [id]: todo }
}
```

**Avoid duplicating state.** Storing `selectedUser` and `users` separately causes sync issues — store the ID instead and derive the user.

---

## 5. Context Optimization

Context solves prop drilling, but large contexts often become bottlenecks.

**The problem:** When a Provider's value changes, **all consumers re-render** — even if they only use a small slice of the context.

```jsx
<AuthContext.Provider value={{ user, theme }}>
```

Changing `theme` re-renders components that only care about `user`.

**Why:** Context uses reference identity. A new object (`{ user, theme }`) means the value changed, so React notifies all consumers.

**Solutions:**

1. **Split contexts:**
   ```jsx
   <UserContext.Provider>
   <ThemeContext.Provider>
   <CartContext.Provider>
   ```

   Updates become isolated.

2. **Memoize the value:**
   ```jsx
   const value = useMemo(() => ({ user }), [user]);
   <AuthContext.Provider value={value}>
   ```

**Key insight:** Context is **not** a high-performance state management solution. It's dependency injection. For highly dynamic global state, use Redux, Zustand, or Jotai instead.

**For complete Context API internals, optimization patterns, and Context vs Redux tradeoffs, see File 04: State Management, Section 7.**

---

## 6. Component Architecture for Performance

Good architecture naturally improves performance. Bad architecture creates endless optimization problems.

**Split large components.** A single `<DashboardPage>` containing charts, tables, forms, filters, and modals means any state update may re-render everything. Splitting creates isolated rendering boundaries and makes memoization easier.

**Controlled vs uncontrolled inputs:** Large controlled forms can lag because every keystroke updates state, triggers a render, and updates the tree. Libraries like React Hook Form optimize this by using uncontrolled strategies internally (refs + minimal renders).

**Avoid inline objects/arrays:**

```jsx
<Component style={{ color: "red" }} /> // breaks memoization
```

Extract to a constant or use `useMemo` if the component is memoized.

---

## 7. Code Splitting & Bundle Optimization

JavaScript bundle size massively impacts performance. The browser must download, parse, compile, and execute all that JS — large bundles block the main thread, especially on mobile.

**React.lazy** loads components on demand:

```jsx
const AdminPage = React.lazy(() => import("./AdminPage"));

<Suspense fallback={<Loader />}>
  <AdminPage />
</Suspense>
```

**Route-based splitting** is one of the highest-impact optimizations. Instead of a 3MB `main.js`, split into `home.chunk.js`, `dashboard.chunk.js`, `settings.chunk.js`. Users only download what they need.

**Dynamic imports** for heavy features:

```jsx
import("./analytics").then(module => {});
```

Useful for admin panels, charts, editors, and feature modules that most users never see.

**Tree shaking** removes unused code:

```js
// Bad: imports entire lodash
import _ from "lodash";

// Good: imports only debounce
import debounce from "lodash/debounce";
```

Use **bundle analyzers** (webpack-bundle-analyzer, Vite visualizer) to detect duplicate dependencies, huge packages, and dead code.

---

## 8. Data Fetching Performance

Network performance often matters more than rendering performance.

**Waterfall requests** are slow:

```jsx
useEffect(() => {
  fetchUser().then(user => {
    fetchPosts(user.id); // waits for user fetch
  });
}, []);
```

Sequential fetching multiplies latency.

**Parallel fetching:**

```js
Promise.all([fetchUser(), fetchPosts()]);
```

**Caching** with React Query or SWR optimizes deduplication, background refresh, and stale data handling. They avoid unnecessary requests — network latency is often the real bottleneck.

**Debouncing** delays execution until the user stops typing:

```jsx
const debouncedSearch = debounce(searchFn, 300);
```

Useful for search, API calls, and auto-save.

**Throttling** limits execution frequency:

```jsx
const throttledScroll = throttle(handleScroll, 200);
```

Useful for scroll, resize, and mouse tracking events.

---

## 9. Large List & UI Performance

Large lists are one of the most common production bottlenecks. Rendering thousands of DOM nodes is expensive — the browser struggles with layout, paint, memory, and event handling.

**Virtualization** (react-window, react-virtualized) renders only visible items:

```
Instead of: 10,000 rows
Render only: ~20 visible rows
```

As the user scrolls, old rows are removed and new rows are rendered. Benefits: fewer DOM nodes, smoother scrolling, reduced memory usage.

**react-window basic example:**

```jsx
import { FixedSizeList } from "react-window";

function VirtualList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>{items[index].name}</div>
  );

  return (
    <FixedSizeList
      height={600}       // visible container height
      itemCount={items.length}
      itemSize={50}      // each row height in px
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

The `style` prop from react-window is critical — it positions each row absolutely. Without it, virtualization breaks.

**Key stability matters.** Using `key={index}` causes remounting, state bugs, and unnecessary DOM work when items reorder. Use stable IDs:

```jsx
items.map(item => <Row key={item.id} />)
```

---

## 10. Profiling & Debugging

Never optimize blindly. Always measure first.

**React Profiler** (React DevTools) shows render duration, frequency, reasons, and component hierarchy. It's the most important React optimization tool. Developers often optimize the wrong components — the Profiler reveals actual bottlenecks.

**Chrome DevTools Performance tab** analyzes scripting, rendering, painting, memory leaks, and long tasks.

**Lighthouse** measures performance, accessibility, SEO, and best practices.

**Web Vitals** (production metrics):

| Metric | Meaning |
|---|---|
| **LCP** | Largest Contentful Paint — how long until main content loads |
| **INP** | Interaction to Next Paint — how fast the UI responds to input |
| **CLS** | Cumulative Layout Shift — how much the layout jumps around |

**Production monitoring:** Tools like Sentry, Datadog, New Relic, and Vercel Analytics track real-user performance in the wild.

---

## 11. Concurrent Features

React 18 introduced concurrent rendering — a major internal architecture improvement. Old React rendered synchronously, so large renders blocked the UI thread.

Concurrent rendering lets React pause, interrupt, prioritize, and resume work. Enabled by Fiber architecture, it keeps interactions responsive even during heavy rendering.

**useTransition** marks low-priority updates:

```jsx
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setSearchQuery(value); // non-urgent
});
```

Input stays responsive while the search results update later. Useful for filtering lists, tab switching, and search UIs.

**useDeferredValue** defers expensive updates:

```jsx
const deferredQuery = useDeferredValue(query);
```

The input updates immediately, but heavy UI renders with the deferred value.

**Important:** Concurrent rendering is **not** multi-threading. JavaScript still runs on a single thread. React just schedules work more intelligently.

---

## 12. Real Production Bottlenecks

Real production problems rarely come from missing `useMemo`. They're architectural:

- **Giant context providers** — large sections re-render constantly
- **Excessive global state** — too many subscribers update unnecessarily
- **Rendering huge tables without virtualization**
- **Heavy logic inside render** — expensive calculations run every render
- **Fetching inside `useEffect` everywhere** — creates loading waterfalls
- **Large controlled forms** — keystrokes lag
- **Huge third-party libraries** — bundle size kills mobile performance
- **Memory leaks** — uncleaned intervals, listeners, subscriptions, or pending requests cause degradation over time

---

## Performance Anti-patterns

- **Premature optimization** — measure before optimizing
- **Overusing `useMemo`/`useCallback`** — increases complexity with little benefit
- **Globalizing everything** — not all state belongs globally
- **Giant components** — increase render cost and reduce optimization opportunities
- **Heavy logic during render** — expensive calculations inside render paths hurt performance

---

## React Compiler — The Future of Memoization

**What it is:** The React Compiler (React 19+) automatically adds `useMemo`, `useCallback`, and `React.memo` to your components at build time. You write plain code; the compiler figures out what needs to be memoized.

**Why it matters for performance:** Manual memoization is error-prone and easy to get wrong. The compiler statically analyzes your code and applies memoization optimally — better than most developers do by hand.

**Before compiler:**
```jsx
const filteredUsers = useMemo(
  () => users.filter(u => u.active),
  [users]
);
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

**After compiler (you just write):**
```jsx
const filteredUsers = users.filter(u => u.active);
const handleClick = () => doSomething(id);
// Compiler handles the rest automatically
```

**Interview tip:** Mention the React Compiler when talking about memoization. It shows you're current. But still explain manual memoization clearly — most production codebases haven't upgraded to React 19 yet.

---

## Senior-level Performance Mindset

The best optimization is preventing unnecessary work. Senior engineers focus on:

- **Architecture** — rendering boundaries, state design, update isolation
- **Network optimization** — caching, parallel fetching, code splitting
- **Bundle size** — tree shaking, lazy loading, dynamic imports
- **Scalability** — patterns that work at 10x scale

Not random hook memoization.

---

## Common Senior-level Interview Questions

**Q: Difference between `useMemo` and `useCallback`?**

| useMemo | useCallback |
|---|---|
| Memoizes values | Memoizes functions |
| Returns computed result | Returns memoized function reference |
| For expensive calculations | For stable callbacks |

**Q: When should you use `React.memo`?**

When the component renders frequently, rendering is expensive, and props remain stable.

**Q: Why can memoization hurt performance?**

Memoization has memory cost, comparison cost, and dependency tracking overhead. Over-memoizing adds complexity without measurable benefit.

**Q: What causes unnecessary re-renders?**

Parent re-renders, unstable references (inline objects/functions), context updates, and poorly structured state.

**Q: How do you optimize large lists?**

Virtualization (react-window), windowing, or pagination.

**Q: What is referential equality?**

React compares by reference, not deep value: `{} === {}` is `false`. New references break memoization.

**Q: Why is state colocation important?**

It minimizes unnecessary rendering by reducing the affected subtree size.

**Q: How does code splitting improve performance?**

Reduces initial bundle size by loading code on demand, speeding up first load and reducing parse/compile time.

**Q: Difference between throttling and debouncing?**

| Debouncing | Throttling |
|---|---|
| Delays execution until inactivity | Limits execution frequency |
| Runs after user stops | Runs at intervals |
| Good for search | Good for scroll |

**Q: What tools debug React performance?**

React Profiler, Chrome DevTools, Lighthouse, Web Vitals, production monitoring (Sentry, Datadog).
