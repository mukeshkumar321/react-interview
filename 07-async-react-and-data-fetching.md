# Async React & Data Fetching

## Topics Covered

- [1. Async React Fundamentals](#1-async-react-fundamentals)
- [2. Data Fetching with useEffect](#2-data-fetching-with-useeffect)
- [3. Async/Await Patterns](#3-asyncawait-patterns)
- [4. AbortController & Cancellation](#4-abortcontroller--cancellation)
- [5. Race Conditions](#5-race-conditions)
- [6. Parallel vs Sequential Requests](#6-parallel-vs-sequential-requests)
- [7. TanStack Query (React Query)](#7-tanstack-query-react-query)
- [8. Caching Strategies](#8-caching-strategies)
- [9. Pagination](#9-pagination)
- [10. Infinite Scroll](#10-infinite-scroll)
- [11. Error Handling](#11-error-handling)
- [12. Loading States](#12-loading-states)
- [13. Performance Optimization](#13-performance-optimization)
- [14. Common Patterns](#14-common-patterns)

---

Modern React applications constantly communicate with APIs, databases, authentication services, and real-time servers. Async React refers to handling operations that don't complete immediately - API requests, file uploads, authentication, and background synchronization.

---

## 1. Async React Fundamentals

**Interview Focus:** Understanding client-side vs server-side fetching is crucial for modern React.

React itself is synchronous while rendering UI, but applications often depend on asynchronous data sources.

**Client-side fetching:** Data is fetched after the page loads in the browser.

```js
useEffect(() => {
  fetch("/api/products")
    .then(res => res.json())
    .then(setProducts);
}, []);
```

Advantages: simpler architecture, dynamic interactions. Disadvantages: slower first content, SEO limitations, visible loading spinners.

**Server-side fetching:** The server fetches data before HTML reaches the browser (Next.js Server Components, SSR).

```js
async function ProductsPage() {
  const products = await fetch("/api/products").then(r => r.json());
  return <ProductList products={products} />;
}
```

Advantages: faster perceived load, better SEO, reduced client work. Disadvantages: increased server cost, more complex architecture.

**Interview insight:** SEO pages use server-side; user dashboards use client-side; frequently changing personalized data uses client-side; static/pre-rendered data uses server-side.

**Async state lifecycle:**

```
Idle → Loading → Success/Error → Refetching → Stale → Revalidating
```

Senior engineers model async flows explicitly instead of using scattered boolean flags.

---

## 2. Data Fetching with useEffect

**Interview Focus:** Classic React pattern - understanding its limitations is important.

Classic pattern:

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

**Problems with useEffect fetching:**
- Race conditions
- Duplicate requests
- Stale closures
- Manual caching
- Difficult retries
- Hard synchronization

This is why libraries like TanStack Query exist.

**Basic async state handling:**

```js
const [data, setData] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
```

Better approach - combined status:

```js
const initialState = {
  status: "idle",
  data: null,
  error: null,
};
```

---

## 3. Async/Await Patterns

**Interview Focus:** Preferred over `.then()` chains - commonly tested in practical exercises.

Good pattern:

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

**Best practices:**
- Always use try/catch
- Always use finally for cleanup
- Use centralized API layer
- Type responses properly

**Fetch API gotchas:**

```js
async function getUsers() {
  const response = await fetch("/api/users");

  if (!response.ok) {
    throw new Error("Failed request");
  }

  return response.json();
}
```

Important: Response is not automatically JSON (requires `await response.json()`); fetch does not reject on HTTP errors (must manually handle `if (!response.ok)`).

---

## 4. AbortController & Cancellation

**Interview Focus:** Very important in production - prevents memory leaks and stale data.

Without cancellation, memory leaks occur and stale requests overwrite fresh data.

```js
useEffect(() => {
  const controller = new AbortController();

  fetch("/api/users", {
    signal: controller.signal,
  });

  return () => controller.abort();
}, []);
```

**Why request cancellation matters:**
- Prevents state updates after unmount
- Avoids race conditions
- Reduces bandwidth waste
- Improves UX

**Common scenario:** User types in search box, triggering multiple requests. Without cancellation, older requests can overwrite newer results.

---

## 5. Race Conditions

**Interview Focus:** Senior-level understanding - common source of production bugs.

Race condition occurs when multiple requests finish out of order:

```txt
Request A starts
Request B starts
Request B finishes
Request A finishes later → stale data overwrites fresh data
```

**Solutions:**

1. **Cancellation:** Abort previous requests
2. **Request tracking:** Track latest request ID
3. **Latest-request-wins:**

```js
let latestRequestId = 0;

async function search(query) {
  const requestId = ++latestRequestId;
  const data = await fetchSearch(query);
  
  if (requestId !== latestRequestId) return; // stale
  setResults(data);
}
```

Libraries like TanStack Query solve this automatically.

---

## 6. Parallel vs Sequential Requests

**Interview Focus:** Performance optimization question - commonly asked.

**Parallel requests:** Multiple requests run simultaneously.

```js
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
]);
```

Benefits: faster loading, reduced waiting time. Use when requests are independent.

**Sequential requests:** One request depends on another.

```js
const user = await fetchUser();
const posts = await fetchPosts(user.id);
```

Use cases: dependent queries, auth flows, relational APIs.

**Interview insight:** Waterfall fetching (unnecessary sequential requests) is a major performance anti-pattern. Parallelize when possible.

---

## 7. TanStack Query (React Query)

**Interview Focus:** Industry-standard library - understanding it is essential for senior roles.

TanStack Query is the most popular async state management library for React.

**Core features:** Caching, retries, deduplication, background refetching, optimistic updates, pagination, and infinite queries.

Basic example:

```js
const { data, isLoading, error } = useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
});
```

**Why senior engineers prefer it:**
- Removes manual cache handling
- Eliminates loading state duplication
- Solves synchronization complexity
- Handles retry logic automatically
- Prevents race conditions

**Query keys** uniquely identify cached data:

```js
["users", userId]
["posts", { page: 1, filter: "active" }]
```

Good query keys: specific and hierarchical. Bad query keys: too generic like `["data"]`.

**Mutations** handle create/update/delete:

```js
const mutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    queryClient.invalidateQueries(["posts"]);
  },
});
```

Mutation lifecycle: `onMutate → mutation → onSuccess/onError → invalidateQueries`.

---

## TanStack Query vs SWR

SWR (from Vercel/Next.js team) is a lighter alternative to TanStack Query.

| Feature | TanStack Query | SWR |
|---|---|---|
| Bundle size | Larger | Smaller |
| Mutations | Full support | Minimal |
| Caching control | Very granular | Simpler |
| Devtools | Yes | Limited |
| Pagination / infinite | Built-in hooks | Manual |
| Setup complexity | Moderate | Very simple |

```js
// SWR basic usage
import useSWR from "swr";

const fetcher = (url) => fetch(url).then(r => r.json());
const { data, error, isLoading } = useSWR("/api/users", fetcher);
```

**When to use SWR:** Simple apps, Next.js apps (same team), minimal setup needed.
**When to use TanStack Query:** Complex caching, mutations with optimistic updates, detailed control over invalidation, devtools.

---

## 8. Caching Strategies

**Interview Focus:** Understanding caching is critical for performance and scalability.

Caching improves performance drastically by avoiding redundant network requests.

**Types of cache:**
- Memory cache (in-app)
- Browser cache (HTTP headers)
- CDN cache (edge servers)
- Query cache (React Query)

React Query cache example:

```js
staleTime: 1000 * 60 // 1 minute
```

**Cache invalidation:** Hardest problem in frontend engineering.

Strategies:
- Time-based: `staleTime`
- Event-based: `invalidateQueries(["users"])`
- Manual updates: `setQueryData`

**Interview insight:** Wrong cache invalidation causes stale UI bugs - extremely common in production.

**Request deduplication:** Avoid duplicate identical requests. Without deduplication, 5 components making the same request create 5 network calls. TanStack Query automatically deduplicates requests.

---

## 9. Pagination

**Interview Focus:** Common practical interview question - implementing pagination correctly.

Large datasets should never load entirely.

**Offset pagination:**

```js
?page=2&limit=20
```

Simple but inefficient for large datasets.

**Cursor pagination:**

```js
?cursor=abc123
```

Better for large-scale systems. Cursor pagination is preferred in scalable systems.

**React Query pagination:**

```js
const { data } = useQuery({
  queryKey: ["users", page],
  queryFn: () => fetchUsers(page),
  keepPreviousData: true,
});
```

`keepPreviousData` shows old data while fetching new page - better UX.

---

## 10. Infinite Scroll

**Interview Focus:** Popular pattern - often asked in practical exercises.

Load more data as user scrolls using `IntersectionObserver`.

```js
const {
  data,
  fetchNextPage,
  hasNextPage,
} = useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: ({ pageParam = 0 }) => fetchPosts(pageParam),
  getNextPageParam: (lastPage) => lastPage.nextCursor,
});

// Trigger on scroll
useEffect(() => {
  const observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting && hasNextPage) {
      fetchNextPage();
    }
  });
  
  observer.observe(loadMoreRef.current);
}, []);
```

**Problems:** Memory growth, accessibility issues, difficult scroll restoration.

**Senior-level advice:** Use virtualization with infinite scroll (react-window or react-virtualized) for large lists.

**Debounced search** - avoid request spam while typing:

```js
const debouncedSearch = debounce(searchUsers, 300);
```

Without debounce, typing "hello" triggers 5 API requests. With debounce, it triggers 1.

---

## 11. Error Handling

**Interview Focus:** Production-level concern - error handling separates junior from senior engineers.

Professional UI handles all async states: loading, error, empty, and success.

```jsx
if (loading) return <Spinner />;
if (error) return <ErrorMessage error={error} />;
if (users.length === 0) return <EmptyState />;
return <UserList users={users} />;
```

**Senior-level insight:** Most junior apps only handle loading. Production apps handle retry, offline state, partial failures, stale cached data, and skeleton loading.

**Error boundaries** catch rendering failures:

```jsx
<ErrorBoundary>
  <Users />
</ErrorBoundary>
```

Important: Error boundaries do NOT catch async errors in event handlers or failed fetches automatically. You need to integrate error handling with boundaries manually or use libraries.

**Retry mechanisms:**

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

Production improvements: exponential backoff, jitter, retry only safe requests (GET), avoid retry storms.

---

## 12. Loading States

**Interview Focus:** UX concern - shows attention to user experience.

**Types of loading states:**
- Initial loading: First data fetch
- Refetching: Updating existing data
- Loading more: Pagination/infinite scroll
- Optimistic updates: Instant UI updates

**Skeleton loading** (better than spinners):

```jsx
{isLoading ? <Skeleton /> : <UserCard user={user} />}
```

**Loading state best practices:**
- Show skeleton for initial load
- Show spinner for actions (submit, delete)
- Keep previous data visible while refetching
- Indicate background updates subtly

---

## 13. Performance Optimization

**Interview Focus:** Senior-level expectation - understanding optimization strategies.

**Key techniques:**
- Request deduplication
- Caching
- Pagination
- Virtualization (for long lists)
- Debouncing (for search)
- Memoization
- Prefetching
- Background refetching

**Prefetching** - load data before user needs it:

```js
queryClient.prefetchQuery({
  queryKey: ["user", id],
  queryFn: () => fetchUser(id),
});
```

Common use cases: hover prefetch, route prefetch, pagination prefetch.

**Network optimization:**
- Compress payloads
- Avoid overfetching (use GraphQL or selective fields)
- Batch requests when possible
- Use CDN for static assets

---

## 14. Common Patterns

**Interview Focus:** Practical patterns that appear frequently in production.

**Optimistic UI updates:** UI updates before server confirms success.

```js
const mutation = useMutation({
  mutationFn: likePost,
  onMutate: async (postId) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries(["posts"]);
    
    // Snapshot previous value
    const previous = queryClient.getQueryData(["posts"]);
    
    // Optimistically update
    queryClient.setQueryData(["posts"], (old) => 
      old.map(post => 
        post.id === postId ? { ...post, liked: true } : post
      )
    );
    
    return { previous };
  },
  onError: (err, variables, context) => {
    // Rollback on error
    queryClient.setQueryData(["posts"], context.previous);
  },
});
```

Benefits: instant UX, smoother interactions. Risks: need rollback on failure.

**API layer architecture:** Senior apps separate API logic from UI.

```txt
services/
  userApi.js
  postApi.js
```

Benefits: reuse, testing, consistency, centralized auth, easier retries.

**Common anti-patterns:**
- Fetching directly in render: `const data = fetchUsers();`
- Missing cleanup: `useEffect(() => { fetch(...); }, []);` without cancellation
- Infinite refetch loops: `useEffect(() => { fetchData(); }, [data]);`
- Overusing global state: Not all server state belongs in Redux
- Ignoring loading/error states: Bad UX

**Best practices:**
- Separate server state from client state
- Cache aggressively
- Invalidate carefully
- Retry safely
- Cancel stale requests
- Prefer declarative fetching (React Query over manual useEffect)
- Monitor API performance
- Handle offline scenarios

---

## Final Takeaway

Junior developers think: "How do I fetch data?"

Senior engineers think:
- How to cache efficiently?
- How to prevent stale UI?
- How to synchronize async state?
- How to optimize network usage?
- How to scale data architecture?
- How to recover gracefully from failures?
- How to improve perceived performance?

Good async architecture focuses on consistency, resilience, scalability, perceived performance, and user experience.
