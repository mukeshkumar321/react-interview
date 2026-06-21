# Advanced Component Patterns

## Topics Covered

- [1. Why Component Patterns Matter](#1-why-component-patterns-matter)
- [2. Controlled vs Uncontrolled Components](#2-controlled-vs-uncontrolled-components)
- [3. Compound Components Pattern](#3-compound-components-pattern)
- [4. Higher Order Components (HOC)](#4-higher-order-components-hoc)
- [5. Render Props Pattern](#5-render-props-pattern)
- [6. Custom Hooks Pattern](#6-custom-hooks-pattern)
- [7. Headless Component Architecture](#7-headless-component-architecture)
- [8. Common Component Patterns](#8-common-component-patterns)
- [9. forwardRef Pattern](#9-forwardref-pattern)
- [10. When to Use Which Pattern](#10-when-to-use-which-pattern)
- [11. Component Pattern Anti-patterns](#11-component-pattern-anti-patterns)
- [12. Production Best Practices](#12-production-best-practices)

---

Component patterns are reusable solutions to common UI and architecture problems in React applications. Senior developers follow proven structures that make components reusable, scalable, maintainable, testable, and flexible.

---

## 1. Why Component Patterns Matter

**Interview Focus:** Understanding the "why" behind patterns is crucial for senior interviews.

As applications grow, component complexity grows exponentially. Small apps can survive with basic components, but large applications cannot.

| Problem | Pattern Solution |
|---|---|
| Prop drilling | Context / Provider Pattern |
| Reusable logic | Custom Hooks / HOC |
| Flexible APIs | Compound Components |
| UI abstraction | Headless Components |
| Shared state | Controlled Components |

Without patterns, you end up with:

```jsx
<UserCard
  showAvatar
  showEmail
  showRole
  showActions
  actionType="dropdown"
  avatarSize="large"
/>
```

Over time this becomes unmaintainable - too many props, confusing API, hard maintenance. With patterns:

```jsx
<UserCard>
  <UserCard.Avatar />
  <UserCard.Info />
  <UserCard.Actions />
</UserCard>
```

This is cleaner, scalable, composable, and easier to extend.

**Senior-level thinking:** Good architecture separates UI components (rendering), hooks (logic), services (API), state layer (shared state), and utilities (helper functions).

---

## 2. Controlled vs Uncontrolled Components

**Interview Focus:** This is a fundamental pattern - often asked in both conceptual and practical questions.

**Controlled Components:** React controls the state.

```jsx
function Input() {
  const [value, setValue] = useState("");

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

Advantages: full control, validation, synchronization, and predictable behavior. Use for forms, filters, search inputs, and validation systems.

**Uncontrolled Components:** The DOM controls the state.

```jsx
function Input() {
  const inputRef = useRef();

  function handleSubmit() {
    console.log(inputRef.current.value);
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

Advantages: less re-rendering, simpler for basic forms, and better performance in huge forms. Downsides: harder validation, less predictable, and difficult synchronization.

**Interview insight:** Most production libraries support both:

```jsx
<Select value={value} defaultValue="react" />
```

Controlled gives full control; uncontrolled gives better performance.

---

## 3. Compound Components Pattern

**Interview Focus:** Very common in senior interviews - often asked to implement or explain.

Compound components allow multiple components to work together with shared state. Popular examples include Accordion, Tabs, Select, and Modal.

```jsx
<Tabs>
  <Tabs.List>
    <Tabs.Tab>Profile</Tabs.Tab>
    <Tabs.Tab>Settings</Tabs.Tab>
  </Tabs.List>

  <Tabs.Panels>
    <Tabs.Panel>Profile Content</Tabs.Panel>
    <Tabs.Panel>Settings Content</Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

This creates expressive APIs, natural composition, and flexible structure. Internally, compound components usually use Context API and shared parent state.

**Flexible compound components** allow custom layout:

```jsx
<Modal>
  <CustomHeader />
  <Modal.Close />
  <Modal.Body />
</Modal>
```

This pattern is common in Radix UI, Headless UI, and Reach UI.

**Benefits:**
- Highly composable and flexible
- Clean API - no massive prop lists
- Shared state without prop drilling
- Easy to extend and customize

---

## 4. Higher Order Components (HOC)

**Interview Focus:** Understanding HOC vs hooks is a common comparison question.

A Higher Order Component is a function that takes a component and returns a new component:

```jsx
function withAuth(Component) {
  return function ProtectedComponent(props) {
    const isLoggedIn = true;

    if (!isLoggedIn) {
      return <Login />;
    }

    return <Component {...props} />;
  };
}

const ProtectedDashboard = withAuth(Dashboard);
```

Common HOC use cases: authentication, analytics, permissions, and feature flags.

**Downsides:** wrapper hell, debugging issues, and prop collisions.

**Multiple HOCs can be composed:**

```jsx
compose(
  withAuth,
  withAnalytics,
  withTheme
)(Dashboard);
```

**Modern trend:** Custom hooks are replacing many HOC use cases because they're simpler and avoid wrapper nesting.

---

## 5. Render Props Pattern

**Interview Focus:** Less common now, but still appears in interviews for understanding component composition.

Render props share logic using functions:

```jsx
<DataFetcher
  render={(data) => (
    <UserList users={data} />
  )}
/>
```

The component exposes state through a function, enabling reusable logic and flexible rendering.

**Downsides:** deeply nested JSX and harder readability.

**Why render props fell out of favor:** Before hooks (React 16.8), render props were the best way to share stateful logic between components. Once hooks arrived, you could extract the same logic into a `useX` custom hook and call it directly — no wrapper component needed, no JSX nesting, much easier to read and compose. Today, most render-prop patterns have been replaced by custom hooks.

---

## 6. Custom Hooks Pattern

**Interview Focus:** Modern standard - understanding how to extract logic into hooks is essential.

Custom hooks are the modern way to share stateful logic:

```jsx
function useUser() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    fetchUser()
      .then(setUser)
      .finally(() => setLoading(false));
  }, []);
  
  return { user, loading };
}
```

**Benefits:**
- Clean logic separation
- Easy to test
- No wrapper components
- Composable

**Headless hooks** - hooks that encapsulate all logic but no UI:

```jsx
function useModal() {
  const [isOpen, setIsOpen] = useState(false);
  return { 
    isOpen, 
    open: () => setIsOpen(true), 
    close: () => setIsOpen(false) 
  };
}
```

The UI is completely separate. This maximizes reusability across different visual implementations.

---

## 7. Headless Component Architecture

**Interview Focus:** Increasingly important in modern React - shows understanding of separation of concerns.

Headless components separate logic, accessibility, and behavior from UI styling:

```jsx
const { open, toggle } = useDropdown();
```

You build your own UI on top. Popular libraries include Radix UI, Headless UI, and Reach UI.

**Benefits:**
- Complete design freedom
- Reusable behavior
- Accessibility included
- Easier to theme

**Example pattern:**

```jsx
function useAccordion() {
  const [expanded, setExpanded] = useState(null);
  
  return {
    expanded,
    toggle: (id) => setExpanded(expanded === id ? null : id),
    isExpanded: (id) => expanded === id
  };
}

// Developers build their own UI
const { expanded, toggle, isExpanded } = useAccordion();
```

---

## 8. Common Component Patterns

**Interview Focus:** Practical patterns that appear frequently in production code.

**Provider Pattern:** Shares data across component trees using Context API:

```jsx
<AuthProvider>
  <App />
</AuthProvider>
```

Common use cases: authentication, theme, localization, global settings.

**Slot-based APIs:** Flexible placement of UI pieces:

```jsx
<Card>
  <Card.Header />
  <Card.Content />
  <Card.Footer />
</Card>
```

**Polymorphic Components:** Render different HTML elements dynamically:

```jsx
function Button({ as: Component = "button", ...props }) {
  return <Component {...props} />;
}

<Button as="a" href="/about">About</Button>
```

**Smart vs Presentational:** Smart components handle business logic and data fetching. Presentational components only handle UI. In modern React, custom hooks often replace smart components.

---

## 9. forwardRef Pattern

**Interview Focus:** Essential for reusable component libraries - commonly tested.

Libraries often need direct DOM access. React provides `forwardRef()`:

```jsx
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// Parent can access the input DOM node
const inputRef = useRef();
<Input ref={inputRef} />
```

This is needed for focus management, accessibility, animations, and integrations. Not forwarding refs in reusable libraries is considered poor design.

**useImperativeHandle** - customizes what the parent receives through the ref:

```jsx
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  // Instead of exposing the raw DOM node, expose specific methods only
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ""; },
    getValue: () => inputRef.current.value,
  }));

  return <input ref={inputRef} {...props} />;
});

// Parent usage
const inputRef = useRef();
<FancyInput ref={inputRef} />

// Parent can call these methods, but cannot access the raw DOM node
inputRef.current.focus();   // works
inputRef.current.clear();   // works
inputRef.current.value;     // undefined — we didn't expose this
```

**Why this matters:** It's good API design. Instead of giving the parent full control over a raw DOM node (which breaks encapsulation), you expose only the operations that make sense for consumers of your component.

---

## 10. When to Use Which Pattern

**Interview Focus:** Senior-level question - demonstrating decision-making ability.

| Pattern | Best For | Avoid When |
|---|---|---|
| **Custom Hooks** | Shared stateful logic | Need wrapper component |
| **HOC** | Cross-cutting concerns (legacy) | Building new code |
| **Render Props** | Dynamic rendering control | Modern codebases |
| **Compound Components** | Related components with shared state | Simple one-off components |
| **Headless Components** | Design system flexibility | Tightly coupled UI/logic |
| **Controlled** | Form validation, synchronization | Large forms (performance) |
| **Uncontrolled** | Simple forms, performance-critical | Need validation |

**Modern preference:**
1. Custom Hooks (most common)
2. Compound Components (for complex UI)
3. Headless Components (for libraries)
4. HOC (mainly legacy code)
5. Render Props (rarely used now)

---

## 11. Component Pattern Anti-patterns

**Interview Focus:** Knowing what NOT to do is important for senior roles.

**1. Massive components:** 2000+ line components containing API calls, UI, business logic, state, and validation. Split into smaller focused components.

**2. Prop explosion:** Components with 20+ props. Use composition, configuration objects, or compound components.

**3. Premature abstraction:** Creating "SuperMegaUniversalButton" without proven need. Wait until you have 3+ use cases before abstracting.

**4. Overusing context:** Context updates trigger all consumers to re-render. Split contexts or use external state libraries with selectors.

**5. Deep component nesting:** Too much nesting (10+ levels) hurts readability. Flatten with composition.

**6. Mixing concerns:** Business logic inside UI components. Separate logic into hooks, services, and utilities.

---

## 12. Production Best Practices

**Interview Focus:** Demonstrating production experience and best practices.

**1. Keep components focused:** Single responsibility principle. Each component should do one thing well.

**2. Prefer composition over configuration:** Instead of endless props, use composition:

```jsx
// Bad
<Table sortable paginated filterable searchable />

// Good
<Table>
  <Table.Search />
  <Table.Filters />
  <Table.Pagination />
</Table>
```

**3. Build accessibility first:** Never treat accessibility as optional. Use semantic HTML, ARIA attributes, keyboard navigation, and focus management.

**4. Design stable APIs:** Changing component APIs later is expensive. Design carefully upfront.

**5. Support flexibility:** Reusable components should adapt to multiple use cases without excessive configuration.

**6. Separate logic from UI:** Use custom hooks for logic, components for UI.

**7. Avoid unnecessary re-renders:** Use memoization, state colocation, and stable references.

**8. Document components:** Important for teams, onboarding, and design systems. Include props, examples, and usage guidelines.

**Common Senior Questions:**

**Q: Why are component patterns important in React?**
Component patterns improve scalability, maintainability, reusability, and developer experience. They help teams build consistent and predictable architectures.

**Q: Difference between HOC and Custom Hooks?**
HOCs wrap components. Custom hooks share logic directly using hooks. Modern React prefers custom hooks because they're simpler and avoid wrapper nesting.

**Q: What are compound components?**
Compound components are multiple related components sharing internal state through context (e.g., Tabs, Accordion, Select). They create highly flexible APIs.

**Q: Why is composition preferred over inheritance in React?**
Composition is more flexible, easier to extend, and simpler to maintain. Inheritance creates tight coupling and rigid hierarchies. React strongly favors composition.

**Q: What makes a good reusable component?**
Good reusable components are focused, composable, accessible, configurable, predictable, and not business-specific.

**Q: Explain headless UI architecture.**
Headless UI separates logic and accessibility from styling. Developers control the visual layer while reusing behavior logic.

---

## Final Takeaway

Advanced component patterns are the foundation of scalable React architecture. Senior developers focus on flexibility, API design, composition, accessibility, maintainability, and scalability. The biggest difference between junior and senior React developers is often how they design reusable systems.
