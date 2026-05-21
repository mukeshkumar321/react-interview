# Performance Optimization

## 1. Introduction to React Performance

React performance optimization is the process of reducing unnecessary work done by React and the browser.

Performance issues usually appear when:

- components re-render too often
- expensive calculations run repeatedly
- large lists are rendered
- too much JavaScript is shipped
- state updates trigger deep UI trees
- network-heavy resources slow rendering

Performance optimization is not about making React “fast everywhere.”

It is about:

- rendering less
- calculating less
- loading less
- updating only what changed



### Core Principle

React performance mainly depends on:

1. How often components render
2. How expensive each render is



### Common Performance Symptoms

| Problem | Example |
|---|---|
| UI lag | Typing feels delayed |
| Scroll lag | Large table freezes |
| Slow initial load | Huge JS bundle |
| Input delay | Search box stutters |
| Frequent re-renders | Entire page updates unnecessarily |
| Memory issues | App becomes slower over time |



## 2. Why React Applications Become Slow

React apps usually become slow because of bad rendering patterns rather than React itself.



### Major Causes

#### 1. Unnecessary Re-renders

Most common issue.

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <Child />
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </>
  );
}
```

`Child` re-renders whenever `Parent` re-renders.



#### 2. Large Component Trees

If state exists too high in the tree:

```txt
App
 ├── Header
 ├── Sidebar
 ├── Main
 ├── Footer
```

Updating one small value may re-render the entire tree.



#### 3. Expensive Computations

```jsx
const sortedData = heavySortFunction(data);
```

Runs every render.



#### 4. Large Lists

Rendering 10,000 DOM nodes is expensive.



#### 5. Context Overuse

Single large context causes all consumers to re-render.



#### 6. Huge Bundles

Too much JavaScript increases:

- parse time
- execution time
- hydration time



## 3. Identifying Unnecessary Re-renders

Before optimizing, identify what is re-rendering.



### Signs of Unnecessary Re-renders

- typing lag
- components updating without visible changes
- console logs firing repeatedly
- profiler showing repeated renders



### Simple Debugging

```jsx
function Child() {
  console.log("Child Rendered");

  return <div>Child</div>;
}
```



### Common Reasons

| Cause | Result |
|---|---|
| Parent re-render | Child re-render |
| New object reference | Memo fails |
| New function reference | Callback props change |
| Context update | All consumers update |



## 4. React Rendering Cost

Every render has a cost.

React rendering includes:

1. Running component function
2. Creating virtual DOM
3. Diffing old vs new tree
4. Updating actual DOM



### Important Concept

DOM updates are expensive.

But even before DOM updates:

- JS execution
- reconciliation
- component execution

also cost CPU time.



### Cheap vs Expensive Components

#### Cheap

```jsx
<div>Hello</div>
```

#### Expensive

```jsx
const filtered = bigArray.filter(...);
const sorted = filtered.sort(...);
```



## 5. Component-level Performance Bottlenecks

Sometimes one component slows the entire app.



### Common Bottlenecks

#### Large Lists

```jsx
users.map(user => <Row key={user.id} />)
```



#### Heavy Calculations

```jsx
const analytics = calculateAnalytics(data);
```



#### Large Forms

Hundreds of controlled inputs can lag.



#### Deep Trees

Massive nested components increase reconciliation work.



## 6. State Colocation Strategy

Keep state as close as possible to where it is used.



### Bad

```jsx
<App>
  state for small modal
</App>
```

Entire app re-renders.



### Good

```jsx
<Modal>
  local modal state
</Modal>
```

Only modal re-renders.



### Benefits

- fewer renders
- isolated updates
- simpler architecture



### Interview Insight

Senior engineers avoid “globalizing everything.”

Global state should only contain truly shared state.



## 7. Preventing Unnecessary Re-renders

Optimization starts with reducing render frequency.



### Techniques

#### 1. State Colocation

Move state lower.



#### 2. Stable References

Avoid recreating:

- objects
- arrays
- functions



#### 3. Memoization

Use:

- React.memo
- useMemo
- useCallback

carefully.



#### 4. Split Components

Smaller components isolate renders.



## 8. React.memo Optimization

`React.memo` prevents re-render if props are unchanged.



### Example

```jsx
const Child = React.memo(function Child({ name }) {
  console.log("Rendered");

  return <div>{name}</div>;
});
```



### How It Works

Performs shallow prop comparison.

If props are same:

```txt
skip render
```



### Problem

This still re-renders:

```jsx
<Child user={{ name: "John" }} />
```

Because object reference changes every render.



### Production Insight

`React.memo` is useful only when:

- component renders frequently
- render is expensive
- props are stable



## 9. useMemo Optimization

`useMemo` memoizes computed values.



### Example

```jsx
const sortedUsers = useMemo(() => {
  return users.sort(sortFn);
}, [users]);
```



### Purpose

Avoid expensive recalculations.



### Good Use Cases

- sorting
- filtering
- analytics
- derived data



### Bad Use Cases

Do NOT memoize everything.

This is unnecessary:

```jsx
const value = useMemo(() => count + 1, [count]);
```



## 10. useCallback Optimization

`useCallback` memoizes functions.



### Example

```jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```



### Why It Matters

Functions recreate every render.

This breaks memoization:

```jsx
<Child onClick={() => {}} />
```



### Common Usage

Used with:

- React.memo
- dependency arrays
- event handlers



## 11. Memoization Tradeoffs

Memoization is not free.



### Memoization Adds Cost

React must:

- store cached values
- compare dependencies
- maintain memory



### Overusing Memoization Causes

- complexity
- memory overhead
- harder debugging
- negligible gains



### Senior-level Understanding

Optimize only after identifying bottlenecks.

Not every render is bad.



## 12. Referential Equality Optimization

React compares references, not deep values.



### Primitive Comparison

```js
1 === 1 // true
```



### Object Comparison

```js
{} === {} // false
```



### Problem

```jsx
const style = {
  color: "red"
};
```

New object every render.



### Solution

```jsx
const style = useMemo(() => ({
  color: "red"
}), []);
```



## 13. Context Performance Optimization

Context updates re-render all consumers.



### Problem

```jsx
<AuthContext.Provider value={{ user, theme }}>
```

Changing theme re-renders user consumers too.



### Optimization Strategies

#### Split contexts

```txt
UserContext
ThemeContext
```



#### Memoize values

```jsx
const value = useMemo(() => ({
  user
}), [user]);
```



## 14. Splitting Context for Performance

Large global contexts are performance killers.



### Bad

```txt
AppContext
 ├── auth
 ├── theme
 ├── cart
 ├── notifications
```

Every update affects many consumers.



### Good

Separate contexts by domain.

```txt
AuthContext
ThemeContext
CartContext
```



### Benefits

- isolated renders
- better scalability
- cleaner architecture



## 15. Large List Rendering Problems

Large lists cause:

- memory pressure
- slow rendering
- scroll lag



### Example

```jsx
items.map(item => (
  <Row key={item.id} />
))
```

10,000 rows = huge DOM tree.



## 16. List Virtualization

Render only visible items.



### Popular Libraries

- react-window
- react-virtualized



### Concept

Instead of rendering:

```txt
10,000 rows
```

render only:

```txt
20 visible rows
```



### Major Benefit

Huge performance improvement.



## 17. Windowing Techniques

Windowing = rendering subset of visible content.



### How It Works

As user scrolls:

- old rows removed
- new rows added



### Benefits

- reduced DOM nodes
- faster rendering
- smoother scrolling



## 18. Lazy Loading Components

Load components only when needed.



### Example

```jsx
const Settings = lazy(() => import("./Settings"));
```



### Benefit

Reduces initial bundle size.



### Best Use Cases

- routes
- modals
- dashboards
- admin panels



## 19. Code Splitting

Split large bundles into smaller chunks.



### Without Code Splitting

```txt
main.js = 3MB
```



### With Code Splitting

```txt
home.chunk.js
dashboard.chunk.js
settings.chunk.js
```



### Benefits

- faster initial load
- reduced JS parsing
- better caching



## 20. Dynamic Imports

Dynamic imports load modules on demand.



### Example

```jsx
import("./analytics");
```



### Common Usage

- feature modules
- admin pages
- charts
- editors



## 21. Suspense for Performance

`Suspense` handles lazy-loaded components gracefully.



### Example

```jsx
<Suspense fallback={<Loader />}>
  <Dashboard />
</Suspense>
```



### Benefits

- smoother UX
- loading boundaries
- async rendering support



## 22. Image Optimization in React

Images often dominate page size.



### Optimization Techniques

#### Responsive Images

```html
<img srcset="small.jpg 500w, large.jpg 1000w" />
```



#### Lazy Loading

```html
<img loading="lazy" />
```



#### Modern Formats

Use:

- WebP
- AVIF



### Production Insight

Large unoptimized images destroy Lighthouse scores.



## 23. Debouncing in React

Debouncing delays execution until user stops typing.



### Example

```jsx
const debouncedSearch = debounce(searchFn, 300);
```



### Best Use Cases

- search inputs
- API calls
- auto-save



### Benefit

Reduces unnecessary requests.



## 24. Throttling in React

Throttling limits execution frequency.



### Example

```jsx
const throttledScroll = throttle(handleScroll, 200);
```



### Best Use Cases

- scroll events
- resize events
- mouse tracking



## 25. Optimizing Expensive Computations

Heavy calculations should not run every render.



### Example

```jsx
const analytics = useMemo(() => {
  return heavyCalculation(data);
}, [data]);
```



### Alternative Strategies

- caching
- web workers
- server-side processing



## 26. Event Handler Optimization

Inline handlers recreate functions.



### Bad

```jsx
<button onClick={() => handle(id)} />
```



### Better

```jsx
const onClick = useCallback(() => {
  handle(id);
}, [id]);
```



### Important

Optimize only if needed.

Inline handlers are not automatically bad.



## 27. Avoiding Memory Leaks

Memory leaks happen when unused resources stay alive.



### Common Causes

#### Uncleaned Timers

```jsx
useEffect(() => {
  const id = setInterval(fn, 1000);

  return () => clearInterval(id);
}, []);
```



#### Event Listeners

```jsx
window.addEventListener("resize", fn);
```

Must cleanup.



#### Uncancelled Requests

Abort pending fetches.



## 28. Bundle Size Optimization

Smaller bundles improve:

- load time
- parse time
- performance



### Techniques

#### Remove Unused Libraries

Avoid huge dependencies.



#### Use Dynamic Imports

Load only needed code.



#### Prefer Smaller Alternatives

Example:

```txt
moment.js → dayjs
```



## 29. Tree Shaking

Tree shaking removes unused code.



### Example

#### Bad

```js
import _ from "lodash";
```

#### Better

```js
import debounce from "lodash/debounce";
```



### Important

ES modules enable tree shaking.



## 30. React Profiler Deep Dive

React Profiler helps identify slow components.



### What It Shows

- render duration
- render frequency
- component hierarchy
- wasted renders



### Key Metrics

| Metric | Meaning |
|---|---|
| Render Time | Time spent rendering |
| Commit Time | DOM update duration |
| Re-render Count | Frequency of updates |



## 31. Performance Profiling Techniques

Performance profiling means measuring before optimizing.



### Common Tools

| Tool | Purpose |
|---|---|
| React Profiler | Component renders |
| Chrome DevTools | JS & rendering |
| Lighthouse | Overall performance |
| Web Vitals | Real-user metrics |



## 32. Measuring Render Performance

Measure actual performance impact.



### Useful APIs

```js
performance.now()
```



### React Profiler Example

```jsx
<Profiler
  id="Dashboard"
  onRender={callback}
>
  <Dashboard />
</Profiler>
```



## 33. Production Performance Monitoring

Development performance differs from production.



### Monitor Real Users

Track:

- LCP
- FID
- CLS
- TTFB



### Tools

- Sentry
- Datadog
- New Relic
- Vercel Analytics



### Senior-level Insight

Real-user monitoring matters more than local benchmarks.



## 34. Performance Anti-patterns



### 1. Premature Optimization

Optimizing everything wastes time.



### 2. Overusing useMemo/useCallback

Can reduce readability.



### 3. Giant Contexts

Cause widespread re-renders.



### 4. Massive Global State

Triggers unnecessary updates.



### 5. Rendering Huge Lists Directly

Leads to UI lag.



### 6. Heavy Logic Inside Render

Bad:

```jsx
const result = expensiveFn();
```



## 35. Performance Optimization Best Practices



### Core Rules

#### Measure First

Never optimize blindly.



#### Keep Components Small

Smaller render surface.



#### Colocate State

Avoid global state abuse.



#### Memoize Carefully

Only when beneficial.



#### Use Virtualization

For large lists.



#### Split Bundles

Improve initial load.



#### Clean Up Effects

Prevent memory leaks.



### Golden Rule

The best optimization is preventing unnecessary work.



## 36. Common Senior-level Interview Questions



### Q1. Difference between useMemo and useCallback?

| useMemo | useCallback |
|---|---|
| Memoizes values | Memoizes functions |
| Returns computed result | Returns memoized function |
| Used for expensive calculations | Used for stable callbacks |



### Q2. When should you use React.memo?

Use when:

- component renders frequently
- rendering is expensive
- props remain stable



### Q3. Why can memoization sometimes hurt performance?

Because memoization itself has:

- comparison cost
- memory cost
- dependency tracking cost



### Q4. What causes unnecessary re-renders?

- parent re-renders
- unstable references
- context updates
- recreated functions/objects



### Q5. How do you optimize large lists?

Use:

- virtualization
- pagination
- windowing



### Q6. What is referential equality?

React compares object references instead of deep values.

```js
{} === {} // false
```



### Q7. How does code splitting improve performance?

It reduces initial JavaScript bundle size by loading code on demand.



### Q8. Why is state colocation important?

It minimizes unnecessary renders by keeping updates localized.



### Q9. Difference between debouncing and throttling?

| Debouncing | Throttling |
|---|---|
| Delays execution | Limits execution frequency |
| Runs after inactivity | Runs at intervals |
| Good for search | Good for scroll |



### Q10. What tools do you use for React performance debugging?

- React Profiler
- Chrome DevTools
- Lighthouse
- Web Vitals
- Performance tab in browser DevTools