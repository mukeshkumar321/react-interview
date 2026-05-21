# Next.js



## 1. Introduction to Next.js

### What is Next.js?

Next.js is a React framework built by Vercel that provides:

- Full-stack React capabilities
- Server-side rendering
- Static site generation
- Routing
- API handling
- Performance optimizations
- Production-ready architecture

React itself is only a UI library.

Next.js adds:
- routing
- rendering strategies
- backend capabilities
- optimization
- deployment support



### Why Next.js Became Popular

Traditional React apps had problems:

- Poor SEO
- Large bundle sizes
- Slow initial page load
- Manual routing setup
- Complex webpack configuration
- Difficult server rendering setup

Next.js solved these problems with built-in architecture.



### Core Features of Next.js

| Feature | Purpose |
|---|---|
| File-based routing | Automatic route creation |
| SSR | Better SEO + faster initial load |
| SSG | Pre-render pages at build time |
| ISR | Update static pages incrementally |
| API Routes | Backend APIs inside app |
| Image Optimization | Faster image loading |
| Middleware | Request interception |
| Streaming | Progressive UI rendering |
| Server Components | Reduced client JS |
| Edge Runtime | Faster global execution |



## 2. Why Next.js is Used in Modern React Applications

### Main Reasons Companies Use Next.js

#### 1. SEO Optimization

Traditional SPA React apps struggle with SEO because content is rendered in the browser.

Next.js can render HTML on the server before sending it to the client.

Good for:
- Ecommerce
- Blogs
- Marketing sites
- Documentation
- SaaS landing pages



#### 2. Better Performance

Next.js improves:
- Initial load speed
- Core Web Vitals
- Bundle size
- Image loading
- Code splitting



#### 3. Full-stack Capabilities

Next.js allows:
- frontend
- backend
- APIs
- authentication
- middleware

inside one project.



#### 4. Better Developer Experience

Next.js provides:
- zero config
- automatic routing
- fast refresh
- TypeScript support
- built-in optimization



#### 5. Production Scalability

Used heavily by:
- Netflix
- TikTok
- Twitch
- Hulu
- Notion

because it scales well.



## 3. Next.js Architecture Overview

Modern Next.js architecture contains:

```text
Browser
   ↓
Next.js Server
   ↓
React Server Components
   ↓
Data Fetching Layer
   ↓
Database / APIs
```



### Major Architectural Layers

| Layer | Responsibility |
|---|---|
| Routing Layer | URL handling |
| Rendering Layer | SSR/SSG/CSR |
| Data Layer | Fetching + caching |
| Server Layer | APIs + middleware |
| Client Layer | Interactivity |
| Edge Layer | Fast request execution |



### App Directory Architecture

Modern Next.js uses:

```text
app/
 ├── layout.tsx
 ├── page.tsx
 ├── dashboard/
 │    ├── layout.tsx
 │    ├── page.tsx
 │    └── settings/
```



## 4. App Router vs Pages Router

### Pages Router (Old)

Located inside:

```text
/pages
```

Uses:
- getServerSideProps
- getStaticProps
- API routes



### App Router (Modern)

Located inside:

```text
/app
```

Introduced in Next.js 13+.

Uses:
- React Server Components
- layouts
- streaming
- nested routing
- server actions



### Comparison

| Feature | Pages Router | App Router |
|---|---||
| Routing | File-based | File-based |
| Layouts | Manual | Nested layouts |
| Data Fetching | Old APIs | async components |
| Streaming | Limited | Built-in |
| Server Components | No | Yes |
| Performance | Good | Better |
| Future Support | Legacy | Recommended |



### Senior-level Insight

Most new enterprise applications use App Router because:
- better architecture
- lower JS bundle
- improved scalability
- React 18 integration



## 5. File-based Routing System

### Concept

Each file becomes a route automatically.

Example:

```text
app/about/page.tsx
```

becomes:

```text
/about
```



### Example Structure

```text
app/
 ├── page.tsx
 ├── about/
 │    └── page.tsx
 ├── contact/
 │    └── page.tsx
```

Routes:
- `/`
- `/about`
- `/contact`



### Benefits

- No manual route configuration
- Easier scalability
- Predictable architecture



## 6. Nested Routing

### What is Nested Routing?

Routes can exist inside routes.

Example:

```text
/dashboard/settings
```

Folder structure:

```text
app/
 └── dashboard/
      ├── page.tsx
      └── settings/
           └── page.tsx
```



### Benefits

- Better organization
- Shared layouts
- Independent loading states



## 7. Dynamic Routes

### Dynamic URL Segments

Example:

```text
/products/123
/products/456
```

Folder:

```text
app/products/[id]/page.tsx
```



### Accessing Params

```tsx
export default function ProductPage({
  params,
}: {
  params: { id: string };
}) {
  return <div>{params.id}</div>;
}
```



### Catch-all Routes

```text
[...slug]
```

Example:

```text
/docs/react/hooks/useEffect
```



## 8. Layouts in Next.js

### What are Layouts?

Layouts are shared UI wrappers.

Example:
- navbar
- sidebar
- footer



### Example

```tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <Sidebar />
      {children}
    </div>
  );
}
```



### Benefits

- Persistent UI
- Avoid re-rendering
- Better UX



## 9. Server Components in Next.js

### What are Server Components?

Components rendered on the server by default.

```tsx
export default async function Page() {
  const data = await fetchData();

  return <div>{data.title}</div>;
}
```



### Benefits

| Benefit | Reason |
|---|---|
| Smaller JS bundle | No client JS sent |
| Better performance | Server execution |
| Direct DB access | Safer |
| Faster initial load | HTML streamed |



### Important Rule

Server Components:
- cannot use useState
- cannot use useEffect
- cannot access browser APIs



## 10. Client Components in Next.js

### What are Client Components?

Interactive browser-side components.

Must use:

```tsx
"use client";
```



### Example

```tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```



### When to Use Client Components

Use for:
- state
- effects
- event handlers
- browser APIs
- animations



## 11. Server vs Client Rendering

### Server Rendering

HTML generated on server.

Advantages:
- SEO
- faster first paint



### Client Rendering

Browser downloads JS first.

Advantages:
- interactive apps
- dynamic UI



### Interview Insight

Best practice:
- default to server
- move only interactive parts to client

This minimizes JS bundle size.



## 12. Rendering Strategies in Next.js

Next.js supports multiple rendering modes.

| Strategy | When Rendering Happens |
|---|---|
| CSR | Browser |
| SSR | Request time |
| SSG | Build time |
| ISR | Build + incremental updates |



## 13. SSR (Server-side Rendering)

### What is SSR?

Page generated on every request.

Example:

```tsx
export default async function Page() {
  const data = await fetch(API, {
    cache: "no-store",
  });

  return <div>SSR Page</div>;
}
```



### Use Cases

- personalized dashboards
- authenticated pages
- frequently changing data



### Drawback

Higher server cost.



## 14. SSG (Static Site Generation)

### What is SSG?

Pages generated during build time.



### Benefits

- extremely fast
- CDN cacheable
- cheap hosting



### Use Cases

- blogs
- docs
- marketing pages



## 15. ISR (Incremental Static Regeneration)

### What is ISR?

Allows static pages to update periodically.



### Example

```tsx
fetch(API, {
  next: {
    revalidate: 60,
  },
});
```

Page refreshes every 60 seconds.



### Production Advantage

Combines:
- static speed
- dynamic freshness



## 16. Streaming and Suspense in Next.js

### Problem Before Streaming

Users waited for entire page data before seeing UI.



### Streaming Solution

Send UI in chunks.



### Example

```tsx
<Suspense fallback={<Loading />}>
  <Products />
</Suspense>
```



### Benefits

- Faster perceived performance
- Better UX
- Progressive rendering



## 17. Data Fetching in Server Components

### Example

```tsx
async function getUsers() {
  const res = await fetch(API);

  return res.json();
}

export default async function Page() {
  const users = await getUsers();

  return <Users users={users} />;
}
```



### Advantages

- No client fetching
- Better security
- Reduced bundle size



## 18. Data Fetching in Client Components

### Example

```tsx
"use client";

useEffect(() => {
  fetchUsers();
}, []);
```



### Best Practice

Prefer:
- Server Components for initial data
- Client Components for live updates



## 19. Server Actions

### What are Server Actions?

Functions executed directly on server.



### Example

```tsx
"use server";

export async function createUser() {
  await db.user.create();
}
```



### Benefits

- No separate API endpoint
- Simplified architecture
- Better security



## 20. Route Handlers

### What are Route Handlers?

Backend endpoints inside app router.

Example:

```text
app/api/users/route.ts
```



### Example

```tsx
export async function GET() {
  return Response.json({
    users: [],
  });
}
```



## 21. Middleware in Next.js

### What is Middleware?

Runs before request completion.



### Common Use Cases

- authentication
- redirects
- geolocation
- rate limiting



### Example

```tsx
export function middleware(request) {
  return NextResponse.next();
}
```



## 22. Authentication Patterns

### Common Approaches

| Method | Usage |
|---|---|
| JWT | APIs |
| Session Cookies | Secure auth |
| OAuth | Google/GitHub login |
| Auth Providers | Clerk/Auth.js |



### Modern Pattern

- auth in middleware
- session validation on server
- protected layouts



## 23. Caching Mechanisms in Next.js

### Types of Cache

| Cache | Purpose |
|---|---|
| Request Memoization | Prevent duplicate fetches |
| Data Cache | Store fetch results |
| Full Route Cache | Cache HTML |
| Router Cache | Client-side cache |



### Senior Insight

Caching mistakes are major production issues.

Understand:
- stale data
- invalidation
- revalidation

deeply.



## 24. Revalidation Strategies

### Time-based Revalidation

```tsx
revalidate: 60
```



### On-demand Revalidation

```tsx
revalidateTag("products");
```



### Production Use

Useful for:
- CMS updates
- ecommerce inventory
- dashboards



## 25. Hydration in Next.js

### What is Hydration?

React attaches event handlers to server-rendered HTML.



### Flow

```text
Server HTML
   ↓
Browser Downloads JS
   ↓
React Hydrates
   ↓
Interactive UI
```



## 26. Hydration Mismatch Problems

### What Causes Mismatch?

Server HTML differs from client HTML.



### Common Causes

| Cause | Example |
|---|---|
| Date.now() | Different timestamps |
| Random values | Math.random() |
| Browser APIs | window object |
| Conditional rendering | Different states |



### Fixes

- move browser logic into useEffect
- avoid random values during SSR
- use dynamic imports



## 27. SEO Optimization in Next.js

### Why SEO is Better

Next.js sends pre-rendered HTML.

Search engines can crawl content immediately.



### SEO Best Practices

- metadata optimization
- semantic HTML
- sitemap
- robots.txt
- structured data



## 28. Metadata API

### Example

```tsx
export const metadata = {
  title: "Dashboard",
  description: "Admin dashboard",
};
```



### Dynamic Metadata

```tsx
export async function generateMetadata() {
  return {
    title: "Product",
  };
}
```



## 29. Image Optimization

### Using next/image

```tsx
import Image from "next/image";
```



### Benefits

- lazy loading
- responsive images
- automatic optimization
- modern formats



### Interview Insight

Using regular `<img>` in production is usually considered a mistake in Next.js apps.



## 30. Font Optimization

### next/font

```tsx
import { Inter } from "next/font/google";
```



### Benefits

- no layout shift
- self-hosted fonts
- optimized loading



## 31. Script Optimization

### next/script

Allows:
- lazy loading
- defer loading
- priority control



### Example

```tsx
<Script
  src="analytics.js"
  strategy="lazyOnload"
/>
```



## 32. Code Splitting in Next.js

### Automatic Code Splitting

Next.js splits bundles automatically by route.



### Dynamic Imports

```tsx
const Chart = dynamic(() => import("./Chart"));
```



### Benefits

- smaller initial bundle
- faster load times



## 33. Environment Variables

### Example

```env
DATABASE_URL=abc
NEXT_PUBLIC_API=xyz
```



### Important Rule

Only variables prefixed with:

```text
NEXT_PUBLIC_
```

are exposed to browser.



## 34. API Layer Architecture in Next.js

### Common Architecture

```text
Client
   ↓
Server Actions / Route Handlers
   ↓
Service Layer
   ↓
Database
```



### Best Practice

Never place DB logic directly in components.

Use:
- services
- repositories
- validation layers



## 35. Error Handling in Next.js

### Error Boundaries

Use:

```text
error.tsx
```

inside route segment.



### Example

```tsx
"use client";

export default function Error() {
  return <div>Error occurred</div>;
}
```



## 36. Loading UI and Error UI

### Loading UI

```text
loading.tsx
```

automatically works with Suspense.



### Benefits

- instant feedback
- skeleton loading
- better UX



## 37. Deployment Strategies

### Common Platforms

| Platform | Usage |
|---|---|
| Vercel | Most optimized |
| Netlify | Static hosting |
| AWS | Enterprise infra |
| Docker | Container deployment |



### Production Considerations

- CDN
- caching
- edge locations
- observability
- logging



## 38. Edge Runtime Basics

### What is Edge Runtime?

Code runs near users globally.



### Benefits

- lower latency
- faster response
- better scalability



### Common Use Cases

- middleware
- personalization
- redirects



## 39. Performance Optimization in Next.js

### Major Optimization Areas

| Area | Optimization |
|---|---|
| Images | next/image |
| Fonts | next/font |
| JS Bundle | Server Components |
| Data | caching |
| Rendering | streaming |
| Navigation | prefetching |



### Senior-level Insight

Performance optimization is mostly:
- reducing JS
- reducing hydration
- reducing unnecessary client components



## 40. Common Next.js Production Issues

### Common Problems

| Problem | Cause |
|---|---|
| Hydration mismatch | Different server/client output |
| Large bundle size | Too many client components |
| Slow SSR | Heavy DB queries |
| Cache inconsistency | Incorrect revalidation |
| Memory leaks | Persistent server processes |
| SEO issues | Wrong metadata |



### Real-world Senior Insight

Most production problems are:
- caching issues
- rendering misunderstandings
- client/server boundary mistakes



## 41. Next.js Best Practices

### Recommended Practices

#### 1. Prefer Server Components

Move only interactive parts to client.



#### 2. Keep Client Bundle Small

Avoid unnecessary:
- libraries
- state management
- browser JS



#### 3. Use Proper Caching

Understand:
- static
- dynamic
- revalidation

properly.



#### 4. Use Suspense Boundaries

Improves loading UX.



#### 5. Optimize Images and Fonts

Always use:
- next/image
- next/font



#### 6. Protect Sensitive Logic

Never expose secrets to client.



#### 7. Separate Layers

Maintain:
- UI layer
- service layer
- database layer



## 42. Common Senior-level Interview Questions

### Core Architecture

1. Why is Next.js better than traditional React SPA?
2. Explain App Router architecture.
3. Difference between Server and Client Components?
4. Why are Server Components important?



### Rendering

5. Explain SSR vs SSG vs ISR.
6. When would you choose SSR?
7. What causes hydration mismatch?
8. Explain streaming in React 18.



### Performance

9. How does Next.js optimize performance?
10. How does code splitting work?
11. How do Server Components reduce bundle size?



### Caching

12. Explain Next.js caching layers.
13. Difference between revalidation and cache invalidation?
14. Explain stale data problems.



### Authentication

15. How would you protect routes?
16. Explain middleware-based authentication.



### Production

17. Common production issues in Next.js?
18. How do you structure enterprise-scale Next.js apps?
19. How would you optimize Core Web Vitals?
20. How do you debug hydration issues?



## Final Senior-level Interview Advice

For senior interviews, focus deeply on:

| High-value Topic | Importance |
|---|---|
| Rendering strategies | Extremely important |
| Server Components | Very important |
| Caching | Critical |
| Hydration | Frequently asked |
| Performance optimization | Critical |
| Architecture decisions | Senior-level expectation |
| App Router | Modern standard |

Most companies expect senior engineers to understand:
- why a rendering strategy is chosen
- performance tradeoffs
- caching architecture
- client/server boundaries
- production scalability