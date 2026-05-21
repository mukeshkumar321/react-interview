# Async React & Data Fetching

## 📚 Topics Covered

- [1. Introduction to Async React](#1-introduction-to-async-react)
- [2. Client-side vs Server-side Data Fetching](#2-client-side-vs-server-side-data-fetching)
- [3. Data Fetching Lifecycle in React](#3-data-fetching-lifecycle-in-react)
- [4. Fetch API in React](#4-fetch-api-in-react)
- [5. Axios in React Applications](#5-axios-in-react-applications)
- [6. Async State Handling](#6-async-state-handling)
- [7. Loading, Error, and Empty States](#7-loading-error-and-empty-states)
- [8. Data Fetching with useEffect](#8-data-fetching-with-useeffect)
- [9. Async/Await Patterns in React](#9-asyncawait-patterns-in-react)
- [10. AbortController and Request Cancellation](#10-abortcontroller-and-request-cancellation)
- [11. Race Conditions in Data Fetching](#11-race-conditions-in-data-fetching)
- [12. Preventing Stale Data Issues](#12-preventing-stale-data-issues)
- [13. Parallel API Requests](#13-parallel-api-requests)
- [14. Sequential API Requests](#14-sequential-api-requests)
- [15. Request Deduplication](#15-request-deduplication)
- [16. Polling and Real-time Fetching](#16-polling-and-real-time-fetching)
- [17. Retry Mechanisms](#17-retry-mechanisms)
- [18. Pagination Strategies](#18-pagination-strategies)
- [19. Infinite Scroll Implementation](#19-infinite-scroll-implementation)
- [20. Debounced Search Requests](#20-debounced-search-requests)
- [21. Optimistic UI Updates](#21-optimistic-ui-updates)
- [22. Caching Strategies](#22-caching-strategies)
- [23. Cache Invalidation](#23-cache-invalidation)
- [24. Prefetching Techniques](#24-prefetching-techniques)
- [25. TanStack Query Deep Dive](#25-tanstack-query-deep-dive)
- [26. Query Keys and Cache Management](#26-query-keys-and-cache-management)
- [27. Mutations in TanStack Query](#27-mutations-in-tanstack-query)
- [28. Suspense for Data Fetching](#28-suspense-for-data-fetching)
- [29. Error Boundary Integration](#29-error-boundary-integration)
- [30. Streaming Data Concepts](#30-streaming-data-concepts)
- [31. API Layer Architecture](#31-api-layer-architecture)
- [32. Secure API Communication](#32-secure-api-communication)
- [33. Data Fetching Performance Optimization](#33-data-fetching-performance-optimization)
- [34. Async React Anti-patterns](#34-async-react-anti-patterns)
- [35. Production-level Data Fetching Best Practices](#35-production-level-data-fetching-best-practices)
- [36. Common Senior-level Interview Questions](#36-common-senior-level-interview-questions)

---


## 1. Introduction to Async React

Modern React applications constantly communicate with APIs, databases, authentication services, analytics systems, and real-time servers.

Async React refers to handling operations that do not complete immediately, such as:

- API requests
- File uploads
- Authentication
- Background synchronization
- Real-time updates
- Lazy loading
- Streaming responses

React itself is synchronous while rendering UI, but applications often depend on asynchronous data sources.

Example:

```js
useEffect(() => {
  fetch("/api/users")
    .then((res) => res.json())
    .then((data) => setUsers(data));
}, []);
```

Key challenge in async React:

- UI should remain responsive
- Avoid unnecessary re-renders
- Handle loading/errors properly
- Prevent stale or duplicated requests
- Ensure cache consistency

Senior engineers focus heavily on:

- scalability
- resilience
- caching
- cancellation
- synchronization
- performance



## 2. Client-side vs Server-side Data Fetching

### Client-side Fetching

Data is fetched after page loads in browser.

Example:

```js
useEffect(() => {
  fetch("/api/products")
    .then((res) => res.json())
    .then(setProducts);
}, []);
```

#### Advantages

- Simpler architecture
- Dynamic interactions
- Reduced server rendering cost

#### Disadvantages

- Slower first content
- SEO limitations
- Loading spinners visible



### Server-side Fetching

Server fetches data before HTML reaches browser.

Common in:

- Next.js Server Components
- SSR
- RSC
- Streaming architectures

Example:

```js
async function ProductsPage() {
  const products = await fetch("/api/products").then((r) => r.json());

  return <ProductList products={products} />;
}
```

#### Advantages

- Faster perceived load
- Better SEO
- Reduced client work

#### Disadvantages

- Increased server cost
- More complex architecture
- Hydration challenges



### Interview Insight

Senior interviews often ask:

> "When should data fetching happen on server vs client?"

Good answer:

- SEO pages → server-side
- User dashboards → client-side
- Frequently changing personalized data → client-side
- Static/pre-rendered data → server-side



## 3. Data Fetching Lifecycle in React

Typical lifecycle:

```text
Idle
→ Loading
→ Success/Error
→ Refetching
→ Stale
→ Revalidating
```

Example state machine:

```js
const [status, setStatus] = useState("idle");
```

Common states:

| State | Meaning |
|---|---|
| idle | Request not started |
| loading | Request in progress |
| success | Data loaded |
| error | Request failed |
| refetching | Updating existing data |

Senior engineers model async flows explicitly.



## 4. Fetch API in React

Native browser API.

Example:

```js
async function getUsers() {
  const response = await fetch("/api/users");

  if (!response.ok) {
    throw new Error("Failed request");
  }

  return response.json();
}
```



### Important Fetch Concepts

#### Response is not automatically JSON

```js
await response.json();
```

#### fetch does not reject on HTTP errors

You must manually handle:

```js
if (!response.ok)
```

#### fetch supports AbortController

Important for cleanup.



### Production Tips

Always:

- handle errors
- handle cancellation
- avoid memory leaks
- validate responses
- set timeouts when needed



## 5. Axios in React Applications

Axios is a popular HTTP client.

Example:

```js
import axios from "axios";

const api = axios.create({
  baseURL: "/api",
});

const users = await api.get("/users");
```



### Advantages over Fetch

| Feature | Fetch | Axios |
|---|---||
| JSON auto parsing | No | Yes |
| Request interceptors | No | Yes |
| Timeout support | Manual | Built-in |
| Request cancellation | Yes | Yes |
| Progress tracking | Limited | Better |



### Axios Interceptors

Useful for:

- auth tokens
- refresh tokens
- logging
- retry handling

Example:

```js
api.interceptors.request.use((config) => {
  config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```



## 6. Async State Handling

Async UI needs multiple state layers.

Basic pattern:

```js
const [data, setData] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
```



### Common Mistake

Using too many separate booleans:

```js
isLoading
isFetching
isRefetching
isSubmitting
```

Senior-level improvement:

Use reducer/state machine.

```js
const initialState = {
  status: "idle",
  data: null,
  error: null,
};
```



## 7. Loading, Error, and Empty States

Professional UI handles all async states.



### Loading State

```jsx
if (loading) {
  return <Spinner />;
}
```



### Error State

```jsx
if (error) {
  return <ErrorMessage />;
}
```



### Empty State

```jsx
if (users.length === 0) {
  return <EmptyState />;
}
```



### Senior-level Insight

Most junior apps only handle loading.

Production apps handle:

- retry
- offline state
- partial failures
- stale cached data
- skeleton loading



## 8. Data Fetching with useEffect

Classic React pattern.

```js
useEffect(() => {
  async function fetchUsers() {
    const res = await fetch("/api/users");
    const data = await res.json();

    setUsers(data);
  }

  fetchUsers();
}, []);
```



### Problems with useEffect Fetching

- race conditions
- duplicate requests
- stale closures
- manual caching
- difficult retries
- hard synchronization

This is why libraries like TanStack Query exist.



## 9. Async/Await Patterns in React

Preferred over `.then()` chains.



### Good Pattern

```js
async function loadData() {
  try {
    setLoading(true);

    const data = await fetchUsers();

    setUsers(data);
  } catch (err) {
    setError(err);
  } finally {
    setLoading(false);
  }
}
```



### Senior-level Advice

Always use:

- try/catch
- finally
- centralized API layer
- typed responses



## 10. AbortController and Request Cancellation

Very important in production.

Without cancellation:

- memory leaks occur
- stale requests overwrite fresh data



### Example

```js
useEffect(() => {
  const controller = new AbortController();

  fetch("/api/users", {
    signal: controller.signal,
  });

  return () => controller.abort();
}, []);
```



### Interview Question

> Why is request cancellation important?

Answer:

- prevents state updates after unmount
- avoids race conditions
- reduces bandwidth waste
- improves UX



## 11. Race Conditions in Data Fetching

Race condition occurs when multiple requests finish out of order.

Example:

```text
Request A starts
Request B starts
Request B finishes
Request A finishes later
```

Now stale data overwrites fresh data.



### Solution

- cancellation
- request tracking
- latest-request wins strategy

Example:

```js
let currentRequestId = 0;
```

Libraries like TanStack Query solve this automatically.



## 12. Preventing Stale Data Issues

Stale data appears when UI displays outdated information.



### Strategies

#### Timestamp validation

```js
if (requestId !== latestRequestId) return;
```

#### Cache invalidation

#### Background refetching

#### Optimistic synchronization



### Senior Insight

Stale data bugs are extremely common in large-scale apps.



## 13. Parallel API Requests

Multiple requests run simultaneously.

```js
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
]);
```



### Benefits

- faster loading
- reduced waiting time



### Interview Tip

Use parallel fetching when requests are independent.



## 14. Sequential API Requests

One request depends on another.

```js
const user = await fetchUser();
const posts = await fetchPosts(user.id);
```



### Use Cases

- dependent queries
- auth flows
- relational APIs



## 15. Request Deduplication

Avoid duplicate identical requests.

Without deduplication:

```text
5 components
→ 5 same API calls
```



### Solutions

#### Shared cache

#### Query libraries

#### Memoized requests

TanStack Query automatically deduplicates requests.



## 16. Polling and Real-time Fetching

### Polling

Repeated requests at intervals.

```js
setInterval(fetchNotifications, 5000);
```



### Real-time Alternatives

- WebSockets
- SSE
- Firebase
- GraphQL subscriptions



### Senior-level Insight

Polling is simple but expensive.

Real-time systems scale better for highly interactive apps.



## 17. Retry Mechanisms

Network failures are normal.



### Retry Example

```js
async function retry(fn, retries = 3) {
  try {
    return await fn();
  } catch (err) {
    if (retries === 0) throw err;

    return retry(fn, retries - 1);
  }
}
```



### Production Improvements

- exponential backoff
- jitter
- retry only safe requests
- avoid retry storms



## 18. Pagination Strategies

Large datasets should never load entirely.



### Offset Pagination

```text
?page=2&limit=20
```

Simple but inefficient for large datasets.



### Cursor Pagination

```text
?cursor=abc123
```

Better for large-scale systems.



### Interview Insight

Cursor pagination is preferred in scalable systems.



## 19. Infinite Scroll Implementation

Load more data as user scrolls.



### Common Approach

```js
IntersectionObserver
```



### Problems

- memory growth
- accessibility issues
- difficult scroll restoration



### Senior-level Advice

Use virtualization with infinite scroll.

Libraries:

- react-window
- react-virtualized



## 20. Debounced Search Requests

Avoid request spam while typing.



### Example

```js
const debouncedSearch = debounce(searchUsers, 300);
```



### Why Important

Without debounce:

```text
hello
→ 5 API requests
```

With debounce:

```text
hello
→ 1 API request
```



## 21. Optimistic UI Updates

UI updates before server confirms success.

Example:

```js
setLiked(true);

await likePost();
```



### Benefits

- instant UX
- smoother interactions



### Risks

Need rollback on failure.

```js
try {
  await mutation();
} catch {
  rollback();
}
```



## 22. Caching Strategies

Caching improves performance drastically.



### Types

#### Memory cache

#### Browser cache

#### CDN cache

#### Query cache

#### Service worker cache



### React Query Cache Example

```js
staleTime: 1000 * 60
```



## 23. Cache Invalidation

Hardest problem in frontend engineering.



### Common Strategies

#### Time-based

```js
staleTime
```

#### Event-based

```js
invalidateQueries(["users"]);
```

#### Manual updates



### Interview Insight

Senior engineers understand:

> Wrong cache invalidation causes stale UI bugs.



## 24. Prefetching Techniques

Load data before user needs it.



### Example

```js
queryClient.prefetchQuery(...)
```



### Common Use Cases

- hover prefetch
- route prefetch
- pagination prefetch



### Benefits

- instant navigation
- smoother UX



## 25. TanStack Query Deep Dive

TanStack Query (formerly React Query) is the most popular async state management library for React.



### Core Features

- caching
- retries
- deduplication
- background refetching
- optimistic updates
- pagination
- infinite queries



### Basic Example

```js
const { data, isLoading } = useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
});
```



### Why Senior Engineers Prefer It

It removes:

- manual cache handling
- loading state duplication
- synchronization complexity
- retry logic duplication



## 26. Query Keys and Cache Management

Query keys uniquely identify cached data.

```js
["users", userId]
```



### Good Query Keys

```js
["posts", postId]
["todos", filters]
```



### Bad Query Keys

```js
["data"]
```

Too generic.



### Senior Insight

Query key architecture becomes critical in large apps.



## 27. Mutations in TanStack Query

Mutations handle create/update/delete operations.

Example:

```js
const mutation = useMutation({
  mutationFn: createPost,
});
```



### Mutation Lifecycle

```text
onMutate
→ mutation
→ onSuccess/onError
→ invalidateQueries
```



### Optimistic Mutation Example

```js
onMutate: async () => {
  await queryClient.cancelQueries(["posts"]);
}
```



## 28. Suspense for Data Fetching

React Suspense can pause rendering until data resolves.

Example:

```jsx
<Suspense fallback={<Spinner />}>
  <Users />
</Suspense>
```



### Benefits

- cleaner async UI
- fewer loading states
- better composition



### Limitations

- ecosystem maturity
- harder debugging
- server/client coordination



## 29. Error Boundary Integration

Error boundaries catch rendering failures.

```jsx
<ErrorBoundary>
  <Users />
</ErrorBoundary>
```



### Important

Error boundaries do NOT catch:

- async errors in event handlers
- failed fetches automatically

Libraries integrate errors into boundaries.



## 30. Streaming Data Concepts

Streaming sends partial UI/data progressively.

Common in:

- React Server Components
- AI chat apps
- live dashboards



### Benefits

- faster perceived performance
- progressive rendering
- reduced blocking



### Interview Insight

Streaming is becoming increasingly important in React architecture.



## 31. API Layer Architecture

Senior apps separate API logic from UI.



### Bad

```js
fetch(...)
```

inside every component.



### Good Structure

```text
services/
  userApi.js
  postApi.js
```



### Benefits

- reuse
- testing
- consistency
- centralized auth
- easier retries



## 32. Secure API Communication

Frontend security matters.



### Best Practices

#### Use HTTPS

#### Avoid storing JWT in localStorage when possible

#### Sanitize user input

#### Use CSRF protection

#### Handle token refresh securely



### Never Trust Frontend Validation

Backend must validate everything.



## 33. Data Fetching Performance Optimization

### Key Techniques

#### Request deduplication

#### Caching

#### Pagination

#### Virtualization

#### Debouncing

#### Memoization

#### Prefetching

#### Background refetching



### Network Optimization

- compress payloads
- avoid overfetching
- use GraphQL selectively
- batch requests



## 34. Async React Anti-patterns

### Fetching Directly in Render

❌ Wrong

```js
const data = fetchUsers();
```



### Missing Cleanup

❌ Wrong

```js
useEffect(() => {
  fetch(...);
}, []);
```

without cancellation.



### Infinite Refetch Loops

❌ Wrong

```js
useEffect(() => {
  fetchData();
}, [data]);
```



### Overusing Global State

Not all server state belongs in Redux.



### Ignoring Loading/Error States

Bad UX.



## 35. Production-level Data Fetching Best Practices

### Recommended Stack

- TanStack Query
- centralized API layer
- Axios/fetch wrapper
- Suspense where appropriate



### Key Principles

#### Separate server state from client state

#### Cache aggressively

#### Invalidate carefully

#### Retry safely

#### Cancel stale requests

#### Prefer declarative fetching

#### Monitor API performance

#### Handle offline scenarios



### Senior-level Thinking

Good async architecture focuses on:

- consistency
- resilience
- scalability
- perceived performance
- user experience



## 36. Common Senior-level Interview Questions

### Conceptual Questions

1. Difference between client-side and server-side fetching?
2. Why is server state different from client state?
3. What problems does TanStack Query solve?
4. Explain stale data problems.
5. What causes race conditions?
6. Why use AbortController?
7. How does caching improve UX?
8. Difference between polling and WebSockets?
9. What is optimistic UI?
10. What is Suspense?



### Practical Questions

#### Build debounced search

#### Implement infinite scrolling

#### Create retry mechanism

#### Prevent duplicate requests

#### Design scalable API layer

#### Handle authentication refresh tokens

#### Optimize dashboard fetching



### Architecture Questions

1. When would you use Redux vs React Query?
2. How do you structure API services?
3. How do you handle offline mode?
4. How would you scale async data layer for enterprise apps?
5. How do you prevent stale cache bugs?



## Final Senior-level Takeaway

Junior developers think:

> "How do I fetch data?"

Senior engineers think:

- how to cache efficiently
- how to prevent stale UI
- how to synchronize async state
- how to optimize network usage
- how to scale data architecture
- how to recover gracefully from failures
- how to improve perceived performance