# Next.js

## Topics Covered

- [1. Why Next.js](#1-why-nextjs)
- [2. App Router vs Pages Router](#2-app-router-vs-pages-router)
- [3. Routing & Dynamic Routes](#3-routing--dynamic-routes)
- [4. Layouts](#4-layouts)
- [5. Server vs Client Components](#5-server-vs-client-components)
- [6. SSR, SSG, and ISR](#6-ssr-ssg-and-isr)
- [7. Data Fetching Patterns](#7-data-fetching-patterns)
- [8. Server Actions](#8-server-actions)
- [9. Caching & Revalidation](#9-caching--revalidation)
- [10. Hydration](#10-hydration)
- [11. Performance Optimization](#11-performance-optimization)
- [12. Image Optimization](#12-image-optimization)
- [13. Common Production Issues](#13-common-production-issues)
- [14. Best Practices](#14-best-practices)

---

Next.js is a React framework built by Vercel that provides full-stack React capabilities, server-side rendering, static site generation, routing, API handling, performance optimizations, and production-ready architecture. React itself is only a UI library - Next.js adds routing, rendering strategies, backend capabilities, optimization, and deployment support.

---

## 1. Why Next.js

**Interview Focus:** Understanding the "why" is crucial - commonly asked in architectural discussions.

Traditional React apps had problems: poor SEO, large bundle sizes, slow initial page load, manual routing setup, and complex webpack configuration. Next.js solved these problems.

**Core features:**

| Feature | Purpose |
|---|---|
| File-based routing | Automatic route creation |
| SSR | Better SEO + faster initial load |
| SSG | Pre-render pages at build time |
| ISR | Update static pages incrementally |
| Server Components | Reduced client JS |
| Image Optimization | Faster image loading |
| Streaming | Progressive UI rendering |

**Main reasons companies use Next.js:**

1. **SEO Optimization:** Server rendering makes content crawlable immediately
2. **Better Performance:** Faster initial load, Core Web Vitals, code splitting
3. **Full-stack Capabilities:** Frontend, backend, APIs, auth in one project
4. **Better DX:** Zero config, automatic routing, fast refresh
5. **Production Scalability:** Used by Netflix, TikTok, Twitch

---

## 2. App Router vs Pages Router

**Interview Focus:** Understanding modern Next.js architecture is essential.

**Pages Router (Old):** Located inside `/pages`. Uses `getServerSideProps`, `getStaticProps`, and API routes.

**App Router (Modern):** Located inside `/app`. Introduced in Next.js 13+. Uses React Server Components, layouts, streaming, nested routing, and server actions.

**Comparison:**

| Feature | Pages Router | App Router |
|---|---|---|
| Routing | File-based | File-based |
| Layouts | Manual | Nested layouts |
| Data Fetching | Old APIs | async components |
| Streaming | Limited | Built-in |
| Server Components | No | Yes |
| Performance | Good | Better when optimized |
| Future Support | Legacy | Recommended |

Most new enterprise applications use App Router because of better architecture, lower JS bundle, improved scalability, and React 18 integration.

---

## 3. Routing & Dynamic Routes

**Interview Focus:** File-based routing is fundamental - commonly tested in practical exercises.

Each file becomes a route automatically. Example: `app/about/page.tsx` becomes `/about`.

**Nested routing:**

```txt
app/
 └── dashboard/
      ├── page.tsx           → /dashboard
      └── settings/
           └── page.tsx      → /dashboard/settings
```

**Dynamic routes:**

Dynamic URL segments like `/products/123`. Folder: `app/products/[id]/page.tsx`.

Accessing params:

```tsx
export default function ProductPage({
  params,
}: {
  params: { id: string };
}) {
  return <div>Product {params.id}</div>;
}
```

**Catch-all routes:** `[...slug]` for routes like `/docs/react/hooks/useEffect`.

---

## 4. Layouts

**Interview Focus:** Understanding layouts is key to Next.js architecture.

Layouts are shared UI wrappers like navbar, sidebar, and footer:

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

**Benefits:**
- Persistent UI - doesn't re-render on navigation
- Nested layouts - different sections can have different layouts
- Better UX - smooth transitions

---

## 5. Server vs Client Components

**Interview Focus:** Critical concept - understanding the boundary is essential for senior roles.

**Server Components (default):** Rendered on the server.

```tsx
export default async function Page() {
  const data = await fetchData();
  return <div>{data.title}</div>;
}
```

**Benefits:**

| Benefit | Reason |
|---|---|
| Smaller JS bundle | No client JS sent |
| Better performance | Server execution |
| Direct DB access | Safer |
| Faster initial load | HTML streamed |

**Important rule:** Server Components cannot use `useState`, `useEffect`, or browser APIs.

**Client Components:** Interactive browser-side components. Must use `"use client"`:

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

Use client components for: state, effects, event handlers, browser APIs, animations.

**Interview insight:** Best practice is to default to server and move only interactive parts to client. This minimizes JS bundle size.

---

## 6. SSR, SSG, and ISR

**Interview Focus:** Core rendering strategies - commonly asked in architectural discussions.

**Rendering strategies:**

| Strategy | When Rendering Happens |
|---|---|
| CSR | Browser |
| SSR | Request time |
| SSG | Build time |
| ISR | Build + incremental updates |

**SSR (Server-side Rendering):** Page generated on every request.

```tsx
export default async function Page() {
  const data = await fetch(API, {
    cache: "no-store", // force SSR
  });

  return <div>SSR Page</div>;
}
```

Use cases: personalized dashboards, authenticated pages, frequently changing data. Drawback: higher server cost.

**SSG (Static Site Generation):** Pages generated during build time. Extremely fast, CDN cacheable, cheap hosting. Use cases: blogs, docs, marketing pages.

**ISR (Incremental Static Regeneration):** Allows static pages to update periodically.

```tsx
fetch(API, {
  next: {
    revalidate: 60, // refresh every 60 seconds
  },
});
```

Production advantage: combines static speed with dynamic freshness.

---

## 7. Data Fetching Patterns

**Interview Focus:** Modern data fetching is essential - commonly tested.

**In Server Components:**

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

Advantages: no client fetching, better security, reduced bundle size.

**In Client Components:**

```tsx
"use client";

useEffect(() => {
  fetchUsers();
}, []);
```

**Best practice:** Prefer Server Components for initial data; Client Components for live updates.

**Streaming and Suspense:**

```tsx
<Suspense fallback={<Loading />}>
  <Products />
</Suspense>
```

Benefits: faster perceived performance, better UX, progressive rendering.

---

## 8. Server Actions

**Interview Focus:** Modern pattern - shows understanding of full-stack Next.js.

Functions executed directly on server:

```tsx
"use server";

export async function createUser(formData: FormData) {
  await db.user.create({
    name: formData.get("name"),
  });
}
```

Benefits: no separate API endpoint, simplified architecture, better security.

**Route Handlers** - backend endpoints inside app router:

```tsx
// app/api/users/route.ts
export async function GET() {
  return Response.json({
    users: [],
  });
}
```

**Middleware** — runs code before a request is completed:

```tsx
// middleware.ts (at project root)
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("auth-token");

  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

Middleware runs on every matching request — before the page renders. Use cases: authentication redirects, A/B testing, geolocation-based routing, request logging.

**Key difference from Server Actions:** Middleware runs before the page loads; Server Actions run during user interactions (form submits, button clicks).

---

## 9. Caching & Revalidation

**Interview Focus:** Critical production concept - caching mistakes cause major issues.

**Caching layers:**

| Cache | Purpose |
|---|---|
| Request Memoization | Prevent duplicate fetches |
| Data Cache | Store fetch results |
| Full Route Cache | Cache HTML |
| Router Cache | Client-side cache |

**Revalidation strategies:**

**Time-based:**

```tsx
fetch(API, {
  next: { revalidate: 60 }
});
```

**On-demand:**

```tsx
revalidateTag("products");
```

**Interview insight:** Caching mistakes are major production issues. Understand stale data, invalidation, and revalidation deeply.

---

## 10. Hydration

**Interview Focus:** Understanding hydration issues is common in senior interviews.

**What is hydration?** When Next.js renders a page on the server, it sends HTML to the browser. The user sees content immediately. But that HTML is "static" — no event handlers attached yet. Hydration is the process where React downloads the JavaScript, runs it, and "attaches" interactivity (event listeners, state, etc.) to the existing HTML.

Think of it as: server sends a static picture → React makes it interactive.

```txt
Server HTML → Browser Downloads JS → React Hydrates → Interactive UI
```

**Hydration mismatch problems:** The server rendered HTML and the first client render must produce identical output. If they differ, React throws a hydration error and replaces the server HTML with the client version — causing a flash, or worse, a crash.

**Root cause:** You're running code during render that produces different results on server vs client.

Common causes:

| Cause | Example |
|---|---|
| Date.now() | Different timestamps |
| Math.random() | Random values |
| window object | Browser APIs — doesn't exist on server |
| Conditional rendering | `typeof window !== "undefined"` checks |

**Fixes:**
- Move browser-only logic into `useEffect` (runs client-only, after hydration)
- Use `dynamic(() => import("./Component"), { ssr: false })` for browser-only components
- Avoid random values during initial render

```jsx
// Bad - causes mismatch: server has no window
const isDesktop = window.innerWidth > 768;

// Good - runs only on client, after hydration
const [isDesktop, setIsDesktop] = useState(false);
useEffect(() => {
  setIsDesktop(window.innerWidth > 768);
}, []);
```

---

## 11. Performance Optimization

**Interview Focus:** Performance is a key senior-level topic.

| Area | Optimization |
|---|---|
| Images | next/image |
| Fonts | next/font |
| JS Bundle | Server Components |
| Data | caching |
| Rendering | streaming |
| Navigation | prefetching |

**Code splitting:** Next.js splits bundles automatically by route. Dynamic imports:

```tsx
const Chart = dynamic(() => import("./Chart"));
```

Benefits: smaller initial bundle, faster load times.

**Senior-level insight:** Performance optimization is mostly reducing JS, reducing hydration, and reducing unnecessary client components.

---

## 12. Image Optimization

**Interview Focus:** next/image is fundamental - commonly tested.

Using `next/image`:

```tsx
import Image from "next/image";

<Image
  src="/photo.jpg"
  alt="Photo"
  width={500}
  height={300}
/>
```

**Benefits:**
- Lazy loading - images load as user scrolls
- Responsive images - different sizes for different screens
- Automatic optimization - converts to WebP/AVIF
- Modern formats - better compression

**Interview insight:** Using regular `<img>` in production is usually considered a mistake in Next.js apps.

**Font optimization:**

```tsx
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });
```

Benefits: no layout shift, self-hosted fonts, optimized loading.

---

## 13. Common Production Issues

**Interview Focus:** Real-world experience - demonstrates production knowledge.

| Problem | Cause |
|---|---|
| Hydration mismatch | Different server/client output |
| Large bundle size | Too many client components |
| Slow SSR | Heavy DB queries |
| Cache inconsistency | Incorrect revalidation |
| Memory leaks | Persistent server processes |
| SEO issues | Wrong metadata |

**Real-world senior insight:** Most production problems are caching issues, rendering misunderstandings, and client/server boundary mistakes.

---

## 14. Best Practices

**Interview Focus:** Senior-level expectations - demonstrates production maturity.

**1. Prefer Server Components:**

Move only interactive parts to client. This minimizes JS bundle.

**2. Keep Client Bundle Small:**

Avoid unnecessary libraries, state management, and browser JS.

**3. Use Proper Caching:**

Understand static, dynamic, and revalidation properly.

**4. Use Suspense Boundaries:**

Improves loading UX and perceived performance.

**5. Optimize Images and Fonts:**

Always use next/image and next/font.

**6. Protect Sensitive Logic:**

Never expose secrets to client. Server Actions and Route Handlers keep logic secure.

**7. Separate Layers:**

Maintain UI layer, service layer, and database layer.

**8. Environment Variables:**

```env
DATABASE_URL=abc
NEXT_PUBLIC_API=xyz
```

Only variables prefixed with `NEXT_PUBLIC_` are exposed to browser.

**Common Senior Interview Questions:**

**Core Architecture:**
- Why is Next.js better than traditional React SPA?
- Explain App Router architecture
- Difference between Server and Client Components?
- Why are Server Components important?

**Rendering:**
- Explain SSR vs SSG vs ISR
- When would you choose SSR?
- What causes hydration mismatch?
- Explain streaming in React 18

**Performance:**
- How does Next.js optimize performance?
- How does code splitting work?
- How do Server Components reduce bundle size?

**Caching:**
- Explain Next.js caching layers
- Difference between revalidation and cache invalidation?
- Explain stale data problems

**Production:**
- Common production issues in Next.js?
- How do you structure enterprise-scale Next.js apps?
- How would you optimize Core Web Vitals?
- How do you debug hydration issues?

---

## Final Takeaway

For senior interviews, focus deeply on:

| Topic | Importance |
|---|---|
| Rendering strategies | Extremely important |
| Server Components | Very important |
| Caching | Critical |
| Hydration | Frequently asked |
| Performance optimization | Critical |
| Architecture decisions | Senior-level expectation |

Most companies expect senior engineers to understand why a rendering strategy is chosen, performance tradeoffs, caching architecture, client/server boundaries, and production scalability.
