# Frontend Architecture

## Topics Covered

- [1. Why Architecture Matters](#1-why-architecture-matters)
- [2. Scalable Folder Structure](#2-scalable-folder-structure)
- [3. Feature-based vs Domain-driven](#3-feature-based-vs-domain-driven)
- [4. Component Layering](#4-component-layering)
- [5. API Layer Architecture](#5-api-layer-architecture)
- [6. State Architecture](#6-state-architecture)
- [7. Design Systems](#7-design-systems)
- [8. Authentication & Authorization](#8-authentication--authorization)
- [9. Error Handling](#9-error-handling)
- [10. Performance Architecture](#10-performance-architecture)
- [11. Scalability Challenges](#11-scalability-challenges)
- [12. Security Fundamentals](#12-security-fundamentals)
- [13. Common Patterns](#13-common-patterns)
- [14. Production Best Practices](#14-production-best-practices)

---

Frontend architecture is the process of designing the overall structure of a frontend application so it remains scalable, maintainable, testable, reusable, performant, and easy for teams to collaborate on. Without proper architecture, code becomes tightly coupled, bugs increase, performance degrades, onboarding becomes difficult, and feature development slows down.

---

## 1. Why Architecture Matters

**Interview Focus:** Understanding the "why" is crucial for senior roles - commonly asked in behavioral questions.

Good architecture solves long-term engineering problems.

**Benefits:**

| Benefit | Impact |
|---|---|
| **Scalability** | New features without breaking code |
| **Maintainability** | Easier debugging and modification |
| **Team Collaboration** | Teams work independently |
| **Reusability** | Shared modules reduce duplication |
| **Performance** | Architecture impacts rendering |
| **Testability** | Well-structured = easier to test |
| **Faster Delivery** | Clear structure reduces decisions |

**Senior insight:** As applications grow, builds become slow, state becomes complex, ownership becomes unclear, and bundle size increases. Continuously evolving architecture is essential.

---

## 2. Scalable Folder Structure

**Interview Focus:** Common architectural question - "How would you structure a large React app?"

**Scalable structure:**

```txt
src/
 ├── app/              # app initialization
 ├── features/         # business features
 ├── shared/           # reusable UI/components
 ├── services/         # API logic
 ├── hooks/            # reusable hooks
 ├── store/            # global state
 ├── routes/           # routing
 ├── utils/            # helper functions
 └── assets/           # static files
```

**Responsibilities:**

| Folder | Responsibility |
|---|---|
| app | app initialization, providers |
| features | business features |
| shared | reusable UI/components |
| services | API logic |
| store | global state |
| hooks | reusable hooks |
| utils | helper functions |

---

## 3. Feature-based vs Domain-driven

**Interview Focus:** Architectural decision - demonstrates system design thinking.

**Feature-based:** Organize code by business features.

```txt
features/
 ├── auth/
 │    ├── components/
 │    ├── hooks/
 │    ├── api/
 │    ├── pages/
 │    └── store/
 ├── cart/
 └── dashboard/
```

Instead of separating by file type globally (components/, hooks/, pages/), organize by feature. This improves ownership, discoverability, isolation, and scalability.

**Domain-driven design (DDD):** Align frontend structure with the business domains (the real-world areas the app deals with). For an e-commerce app, domains might be: Authentication, Billing, Orders, Inventory, Analytics. Each domain owns its own UI, state, API logic, validations, and tests.

Think of it like this: feature-based groups by *what you're building*, DDD groups by *what part of the business it belongs to*. For small-to-medium apps, feature-based is simpler. For large apps with multiple teams, DDD makes ownership clearer.

Benefits: business-aligned structure, better team ownership, easier scaling, cleaner boundaries.

**Shared vs Feature-specific:**

One major architectural mistake is over-sharing code.

- **Shared modules:** Reusable across features (`shared/components/Button`)
- **Feature-specific modules:** Used only inside one feature (`features/cart/components/CartSummary`)

**Best practice:** Start feature-specific. Promote to shared only when reused multiple times, abstraction is stable, and API becomes generic enough. Avoid premature abstraction.

---

## 4. Component Layering

**Interview Focus:** Understanding separation of concerns is essential.

Frontend systems should separate components by responsibility:

```txt
Page Components
    ↓
Feature Components
    ↓
Reusable UI Components
```

**Example:**
- UI Component: `<Button />`
- Feature Component: `<CartCheckoutButton />`
- Page Component: `<CheckoutPage />`

Benefits: better separation, improved reusability, easier testing, reduced complexity.

**UI Layer vs Business Logic Layer:**

Senior engineers separate rendering from logic.

- **UI Layer:** Rendering, styles, interactions (`<UserCard user={user} />`)
- **Business Logic Layer:** Validations, transformations, workflows, calculations (`calculateDiscount()`, `validateCheckout()`)

Why separation matters: easier testing, reusable logic, cleaner components, reduced UI complexity.

---

## 5. API Layer Architecture

**Interview Focus:** Senior-level expectation - API logic should never live in components.

**Bad:**

```tsx
useEffect(() => {
  fetch("/api/users")
}, [])
```

**Better architecture:**

```txt
services/
 ├── apiClient.ts
 ├── userService.ts
 └── postService.ts
```

Example:

```ts
export const getUsers = () => {
  return apiClient.get("/users")
}
```

**Advantages:**
- Reusable APIs
- Centralized error handling
- Easier mocking/testing
- Consistent API behavior

**Service layer:** Encapsulates external communication with responsibilities including API calls, data transformation, caching, authentication headers, and retries.

Production insight: Avoid directly coupling UI to backend response structure. Transform data inside service layer.

---

## 6. State Architecture

**Interview Focus:** Critical architectural decision - commonly asked.

Large applications require structured state management.

**State categories:**

| Type | Example |
|---|---|
| Local UI State | modal open |
| Shared UI State | theme |
| Server State | API data |
| Form State | form inputs |
| Session State | auth user |

**Key principle:** Not all state belongs in global stores.

**Global State Organization:**

Global state should contain only truly shared data: authentication, theme, notifications, feature flags, app configuration.

**Avoid:** Putting everything into Redux/Zustand. Over-globalization causes unnecessary re-renders, debugging complexity, and state synchronization issues.

**State Colocation Strategy:**

Keep state as close as possible to where it is used.

Bad: Global modal state for a local component.
Good: Local component state (`const [open, setOpen] = useState(false)`).

Benefits: fewer re-renders, simpler logic, reduced dependencies.

---

## 7. Design Systems

**Interview Focus:** Understanding design systems shows experience with scaled systems.

A design system is a centralized UI foundation including typography, spacing, colors, buttons, forms, and layout primitives.

**Goals:**
- Consistency
- Accessibility
- Scalability
- Faster development

**Shared Component Library:**

```txt
ui/
 ├── Button
 ├── Modal
 ├── Table
 ├── Input
```

**Rules:** Good shared components should be reusable, composable, accessible, and configurable.

**Theme System:**

Themes allow dynamic UI customization using CSS Variables, Tailwind Theme Tokens, or Context-based Theme Switching.

Best practice: Use design tokens instead of hardcoded colors.

```css
/* Good */
color: var(--text-primary);

/* Bad */
color: blue;
```

Production insight: Large companies heavily invest in design systems because UI inconsistency becomes expensive.

---

## 8. Authentication & Authorization

**Interview Focus:** Security and access control - essential for production apps.

**Authentication:** Determines user identity.

Common strategies:

| Strategy | Usage |
|---|---|
| JWT | SPAs |
| Session Cookies | secure web apps |
| OAuth | social login |

Frontend responsibilities: token storage, session handling, route protection, refresh token flow.

Production insight: Avoid storing sensitive tokens in insecure places (localStorage). Prefer secure cookies when possible.

**Authorization:** Controls what users can access.

Common models:
- **RBAC (Role-based):** admin, editor, viewer
- **Permission-based:** canEditUser, canDeletePost

**UI Protection:**

```tsx
{canEdit && <EditButton />}
```

**Important:** Frontend authorization improves UX. Backend authorization enforces security. Never rely on frontend checks alone.

---

## 9. Error Handling

**Interview Focus:** Production maturity - error handling separates junior from senior.

Applications must fail gracefully.

**Error types:**

| Type | Example |
|---|---|
| Network | API failure |
| UI | render crash |
| Validation | invalid form |
| Auth | expired token |

**Strategies:**
- React Error Boundaries - catch rendering errors
- Global API interceptors - catch network errors
- Fallback UIs - graceful degradation
- Centralized logging - track errors

**Logging and Monitoring:**

Production apps require observability.

Common tools: Sentry, Datadog, LogRocket.

Track: crashes, API failures, performance issues, user sessions.

Production insight: Monitoring reduces debugging time drastically.

---

## 10. Performance Architecture

**Interview Focus:** Performance is architectural, not just optimization - commonly tested in senior roles.

Architecture strongly affects performance.

**Key techniques:**
- Lazy loading - defer non-critical components
- Memoization - prevent unnecessary re-renders
- Virtualization - handle large lists efficiently
- Caching - reduce redundant requests
- SSR/SSG - faster initial load
- Code splitting - smaller bundles

**Code Splitting:**

```tsx
const Dashboard = lazy(() => import("./Dashboard"))
```

Types:
- Route-based: split by pages
- Component-based: split heavy components

Benefits: faster initial load, smaller bundles, improved UX.

Production insight: Performance optimization should start during architecture design, not later.

---

## 11. Scalability Challenges

**Interview Focus:** Real-world experience - demonstrates production knowledge.

As systems grow:
- Builds become slow
- State becomes complex
- Ownership becomes unclear
- Bundle size increases
- Testing becomes expensive

**Solutions:**

**Monolithic vs Modular:**

- **Monolithic:** Everything in one tightly connected app
- **Modular:** Split into isolated features with independent ownership

**Micro Frontends:** Split frontend apps into independently deployable pieces.

```txt
Shell App
 ├── Auth App
 ├── Payments App
 └── Analytics App
```

Each mini-app is built and deployed separately. The "Shell App" (host) loads them dynamically at runtime. Teams can use different tech stacks, deploy independently, and own their entire vertical.

**Module Federation** (Webpack 5+) is the main technology that makes micro frontends work. It lets one app ("host") dynamically load code from another app ("remote") at runtime — not at build time:

```js
// webpack.config.js (remote app — exposes its components)
new ModuleFederationPlugin({
  name: "paymentsApp",
  exposes: {
    "./Checkout": "./src/Checkout",
  },
});

// webpack.config.js (host app — consumes remote components)
new ModuleFederationPlugin({
  remotes: {
    paymentsApp: "paymentsApp@https://payments.company.com/remoteEntry.js",
  },
});
```

Advantages: independent deployments, team autonomy, scalable organizations.
Challenges: shared dependencies, styling conflicts, routing complexity, debugging across apps.

**Monorepo:** A single Git repository that contains multiple projects (frontend apps, backend services, shared packages). Instead of separate repos for each project, everything lives in one place.

```txt
monorepo/
 ├── apps/
 │    ├── web/          ← main web app
 │    └── admin/        ← admin panel
 └── packages/
      ├── ui/           ← shared component library
      └── utils/        ← shared helpers
```

Benefits: shared tooling, easier cross-project refactoring, code sharing between apps, atomic commits.

Popular tools:
- **Turborepo** — simple, fast task runner (build/test/lint with caching). Best for most teams.
- **Nx** — more opinionated, has generators, IDE integration, and dependency graph visualization. Better for large orgs with many teams.

| Tool | Best For |
|---|---|
| Turborepo | Simplicity, speed, small-medium monorepos |
| Nx | Large orgs, strict structure, advanced tooling |

---

## 12. Security Fundamentals

**Interview Focus:** Security awareness is essential for senior roles.

Frontend security is critical.

**Common risks:**

| Risk | Example |
|---|---|
| XSS | injected scripts |
| CSRF | unauthorized requests |
| Token leaks | insecure storage |

**Best practices:**
- Sanitize user input
- Use HTTPS
- Secure cookies
- CSP headers
- Avoid dangerous HTML rendering (`dangerouslySetInnerHTML`)

**Environment Configuration:**

Applications use multiple environments: development, staging, production.

```env
VITE_API_URL=
```

Best practices: never hardcode secrets, separate configs, validate environment variables.

---

## 13. Common Patterns

**Interview Focus:** Practical patterns that appear frequently in production.

**Routing Architecture:**

Best practices: nested layouts, lazy loading, protected routes, route-based code splitting.

**Feature Flags:**

Allow controlled releases.

Benefits: gradual rollout, A/B testing, instant rollback, safer deployments.

```tsx
if (featureFlags.newDashboard) {}
```

**Testing Architecture:**

Testing strategy should exist at architecture level.

Types:
- Unit: isolated logic
- Integration: module interactions
- E2E: real user flows

Common tools: Jest, Vitest, React Testing Library, Playwright, Cypress.

Best practice: test behavior, not implementation details.

**CI/CD:**

Automates software delivery.

CI Pipeline: lint, test, build, security checks.
CD Pipeline: staging deployment, production deployment, rollback strategies.

Benefits: fewer manual errors, faster releases, safer deployments.

---

## 14. Production Best Practices

**Interview Focus:** Senior-level expectations - demonstrates production maturity.

**Core principles:**

1. **Separation of Concerns:** UI, logic, API, state clearly separated
2. **Colocate Related Logic:** Keep related files together
3. **Prefer Composition:** More flexible than inheritance
4. **Avoid Premature Optimization:** Build, measure, optimize
5. **Design for Change:** Architecture should adapt
6. **Keep APIs Stable:** Changes are expensive
7. **Optimize Developer Experience:** Happy devs = faster delivery
8. **Monitor Production Continuously:** Observability is critical
9. **Build Incrementally:** Iterative architecture evolution
10. **Simplicity Scales Better:** Complex systems break

**Dependency Management:**

Every dependency increases bundle size, security risk, and maintenance cost.

Best practices: avoid unnecessary libraries, audit dependencies, lock versions, remove unused packages.

**Common Anti-patterns:**

1. Massive components (2000+ lines)
2. Over-globalized state (everything in Redux)
3. Premature abstraction (generic systems too early)
4. Tight coupling (UI directly coupled to APIs)
5. Shared folder chaos (huge unorganized shared directory)

**Architecture Tradeoffs:**

There is no perfect architecture - every decision has tradeoffs.

| Choice | Benefit | Cost |
|---|---|---|
| Redux | centralized state | boilerplate |
| Micro Frontends | team autonomy | complexity |
| Monorepo | code sharing | tooling complexity |

Interview tip: Senior interviews heavily focus on tradeoff reasoning.

**Common Senior Interview Questions:**

**Architecture Design:**
- How would you structure a large-scale React application?
- When would you choose Redux over Context?
- How do you organize shared components?
- How would you design frontend architecture for multiple teams?
- What problems do micro frontends solve?
- How do you handle frontend scalability?

**Performance:**
- How do you optimize large React applications?
- What performance considerations exist at the architecture level?

**State:**
- How do you decide where state should live?
- How do you separate business logic from UI?

**Security:**
- How do you handle authentication architecture?
- What frontend security concerns matter?

**Production:**
- What frontend architecture mistakes have you seen in production?
- How do you ensure code quality and maintainability at scale?

---

## Final Takeaway

Junior developers focus on components.
Mid-level developers focus on features.
Senior engineers focus on systems.

Frontend architecture is about building applications that:
- Scale technically
- Scale organizationally
- Remain maintainable for years
- Allow teams to move fast safely
- Support business growth without collapsing technically

The best architecture is simple, flexible, and evolves with the application's needs.
