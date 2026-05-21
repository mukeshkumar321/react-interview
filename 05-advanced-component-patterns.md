# Advanced Component Patterns

## 📚 Topics Covered

- [1. Introduction to Component Patterns](#1-introduction-to-component-patterns)
- [2. Why Component Patterns Matter](#2-why-component-patterns-matter)
- [3. Reusable Component Architecture](#3-reusable-component-architecture)
- [4. Component API Design Principles](#4-component-api-design-principles)
- [5. Smart vs Presentational Components](#5-smart-vs-presentational-components)
- [6. Controlled Components Pattern](#6-controlled-components-pattern)
- [7. Uncontrolled Components Pattern](#7-uncontrolled-components-pattern)
- [8. Compound Components Pattern](#8-compound-components-pattern)
- [9. Flexible Compound Components](#9-flexible-compound-components)
- [10. Provider Pattern](#10-provider-pattern)
- [11. Higher Order Components (HOC)](#11-higher-order-components-hoc)
- [12. HOC Composition Patterns](#12-hoc-composition-patterns)
- [13. Render Props Pattern](#13-render-props-pattern)
- [14. Custom Hooks vs Render Props vs HOC](#14-custom-hooks-vs-render-props-vs-hoc)
- [15. Headless Component Architecture](#15-headless-component-architecture)
- [16. Slot-based Component APIs](#16-slot-based-component-apis)
- [17. Polymorphic Components](#17-polymorphic-components)
- [18. Forwarding Refs in Component Libraries](#18-forwarding-refs-in-component-libraries)
- [19. Imperative APIs with Components](#19-imperative-apis-with-components)
- [20. Controlled vs Uncontrolled API Tradeoffs](#20-controlled-vs-uncontrolled-api-tradeoffs)
- [21. Building Reusable Form Components](#21-building-reusable-form-components)
- [22. Building Scalable Modal Systems](#22-building-scalable-modal-systems)
- [23. Shared Component Libraries](#23-shared-component-libraries)
- [24. Design System Fundamentals](#24-design-system-fundamentals)
- [25. Themeable Component Architecture](#25-themeable-component-architecture)
- [26. Accessibility in Component Design](#26-accessibility-in-component-design)
- [27. Component Composition Strategies](#27-component-composition-strategies)
- [28. Component Scalability Challenges](#28-component-scalability-challenges)
- [29. Avoiding Prop Explosion](#29-avoiding-prop-explosion)
- [30. Component Pattern Anti-patterns](#30-component-pattern-anti-patterns)
- [31. Production-level Component Best Practices](#31-production-level-component-best-practices)
- [32. Common Senior-level Interview Questions](#32-common-senior-level-interview-questions)

---


## 1. Introduction to Component Patterns

Component patterns are reusable solutions to common UI and architecture problems in React applications.

Instead of writing components randomly, senior developers follow proven structures that make components:

- reusable
- scalable
- maintainable
- testable
- flexible
- easy to understand

A component pattern is essentially:

> “A standardized way of organizing component behavior and communication.”

Patterns become extremely important in:

- large applications
- design systems
- reusable UI libraries
- enterprise applications
- teams with multiple developers

Without proper patterns:

- components become tightly coupled
- props become messy
- logic duplication increases
- reusability decreases
- scaling becomes painful



## 2. Why Component Patterns Matter

As applications grow, component complexity grows exponentially.

Small apps can survive with basic components.

Large applications cannot.

Patterns help solve:

| Problem | Pattern Solution |
|---|---|
| Prop drilling | Context / Provider Pattern |
| Reusable logic | Custom Hooks / HOC |
| Flexible APIs | Compound Components |
| UI abstraction | Headless Components |
| Shared state | Controlled Components |
| Design consistency | Design Systems |



### Real Production Example

Without patterns:

```jsx
<UserCard
  showAvatar
  showEmail
  showRole
  showActions
  actionType="dropdown"
  avatarSize="large"
  roleColor="blue"
/>
```

Over time:

- too many props
- confusing API
- hard maintenance
- difficult testing

With patterns:

```jsx
<UserCard>
  <UserCard.Avatar />
  <UserCard.Info />
  <UserCard.Actions />
</UserCard>
```

This becomes:

- cleaner
- scalable
- composable
- easier to extend



## 3. Reusable Component Architecture

Reusable components should:

- solve one problem
- avoid business-specific logic
- support composition
- expose clean APIs
- remain flexible



### Bad Reusable Component

```jsx
function DashboardAdminSalesButton() {
  return <button>Export Sales</button>;
}
```

This component is:

- too specific
- impossible to reuse



### Better

```jsx
function Button({ children, variant }) {
  return (
    <button className={`btn-${variant}`}>
      {children}
    </button>
  );
}
```

Reusable because:

- generic
- configurable
- composable



### Senior-Level Thinking

Good architecture separates:

| Layer | Responsibility |
|---|---|
| UI Components | Rendering |
| Hooks | Logic |
| Services | API |
| State Layer | Shared state |
| Utilities | Helper functions |



## 4. Component API Design Principles

A component API is how developers interact with components.

Good APIs feel intuitive.



### Principles

#### 1. Keep APIs Small

Bad:

```jsx
<Button
  primary
  secondary
  danger
  success
/>
```

Good:

```jsx
<Button variant="primary" />
```



#### 2. Use Composition

Prefer:

```jsx
<Card>
  <Card.Header />
  <Card.Body />
</Card>
```

Instead of:

```jsx
<Card
  headerTitle=""
  bodyContent=""
/>
```



#### 3. Predictable Naming

Bad:

```jsx
<Button isBlue />
```

Good:

```jsx
<Button variant="primary" />
```



#### 4. Sensible Defaults

```jsx
<Button />
```

Should work without excessive configuration.



## 5. Smart vs Presentational Components

This is one of the oldest React architecture patterns.



### Smart Components

Responsible for:

- business logic
- API calls
- state management

Example:

```jsx
function UserContainer() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetchUsers().then(setUsers);
  }, []);

  return <UserList users={users} />;
}
```



### Presentational Components

Responsible only for UI.

```jsx
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```



### Benefits

| Smart | Presentational |
|---|---|
| Handles logic | Handles UI |
| Stateful | Usually stateless |
| Harder to reuse | Highly reusable |



### Modern React

Today:

- Custom Hooks often replace Smart Components
- Presentational pattern still matters

Example:

```jsx
function UserPage() {
  const users = useUsers();

  return <UserList users={users} />;
}
```



## 6. Controlled Components Pattern

In controlled components:

> React controls the state.



### Example

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



### Advantages

- full control
- validation
- synchronization
- predictable behavior



### Common Use Cases

- forms
- filters
- search inputs
- validation systems



### Interview Insight

Libraries like:

- React Hook Form
- Formik

optimize controlled component limitations.



## 7. Uncontrolled Components Pattern

In uncontrolled components:

> DOM controls the state.



### Example

```jsx
function Input() {
  const inputRef = useRef();

  function handleSubmit() {
    console.log(inputRef.current.value);
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>
        Submit
      </button>
    </>
  );
}
```



### Advantages

- less re-rendering
- simpler for basic forms
- better performance in huge forms



### Downsides

- harder validation
- less predictable
- difficult synchronization



## 8. Compound Components Pattern

Compound components allow multiple components to work together with shared state.

Popular examples:

- Accordion
- Tabs
- Select
- Modal



### Example

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



### Benefits

- expressive APIs
- natural composition
- flexible structure



### Internal Implementation

Usually uses:

- Context API
- shared parent state



## 9. Flexible Compound Components

Normal compound components can become rigid.

Flexible compound components allow:

- custom layout
- arbitrary nesting
- better extensibility



### Example

```jsx
<Modal>
  <CustomHeader />

  <Modal.Close />

  <SomeOtherComponent />

  <Modal.Body />
</Modal>
```

This is common in:

- Radix UI
- Headless UI
- Reach UI



## 10. Provider Pattern

Provider Pattern shares data across component trees.

Built using Context API.



### Example

```jsx
<AuthProvider>
  <App />
</AuthProvider>
```



### Common Use Cases

- authentication
- theme
- localization
- global settings



### Problem It Solves

Avoids:

- prop drilling
- deeply nested props



### Production Insight

Too many providers can create:

- provider hell
- debugging complexity

Solution:

- provider composition
- context splitting



## 11. Higher Order Components (HOC)

A Higher Order Component:

> A function that takes a component and returns a new component.



### Example

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
```

Usage:

```jsx
const ProtectedDashboard = withAuth(Dashboard);
```



### Common HOC Use Cases

- authentication
- analytics
- permissions
- feature flags



### Downsides

- wrapper hell
- debugging issues
- prop collisions



## 12. HOC Composition Patterns

Multiple HOCs can be composed together.



### Example

```jsx
export default withAuth(
  withAnalytics(
    withTheme(Dashboard)
  )
);
```

Problem:

- deeply nested wrappers



### Better

```jsx
compose(
  withAuth,
  withAnalytics,
  withTheme
)(Dashboard);
```



### Modern Trend

Custom hooks are replacing many HOC use cases.



## 13. Render Props Pattern

Render props share logic using functions.



### Example

```jsx
<DataFetcher
  render={(data) => (
    <UserList users={data} />
  )}
/>
```



### How It Works

Component exposes state through a function.



### Benefits

- reusable logic
- flexible rendering



### Downsides

- deeply nested JSX
- harder readability



## 14. Custom Hooks vs Render Props vs HOC

This is a very common senior interview question.



### Comparison

| Pattern | Best For | Downsides |
|---|---|---|
| HOC | Cross-cutting concerns | Wrapper nesting |
| Render Props | Flexible rendering | Nested JSX |
| Custom Hooks | Shared logic | Hooks-only usage |



### Modern Recommendation

Today:

- Prefer Custom Hooks
- Use HOC mainly in legacy code
- Render props used in special cases



## 15. Headless Component Architecture

Headless components separate:

- logic
- accessibility
- behavior

from UI styling.



### Example

```jsx
const {
  open,
  toggle
} = useDropdown();
```

You build your own UI.



### Popular Libraries

- Radix UI
- Headless UI
- Reach UI



### Benefits

- complete design freedom
- reusable behavior
- accessibility included



## 16. Slot-based Component APIs

Slots allow flexible placement of UI pieces.



### Example

```jsx
<Card>
  <Card.Header />
  <Card.Content />
  <Card.Footer />
</Card>
```

Each section acts like a slot.



### Benefits

- flexible layouts
- clean APIs
- composability



## 17. Polymorphic Components

Polymorphic components render different HTML elements dynamically.



### Example

```jsx
function Button({ as: Component = "button", ...props }) {
  return <Component {...props} />;
}
```

Usage:

```jsx
<Button as="a" href="/about">
  About
</Button>
```



### Why Important

Used heavily in:

- design systems
- UI libraries



## 18. Forwarding Refs in Component Libraries

Libraries often need direct DOM access.

React provides:

```jsx
forwardRef()
```



### Example

```jsx
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});
```



### Why Important

Needed for:

- focus management
- accessibility
- animations
- integrations



### Interview Insight

Not forwarding refs in reusable libraries is considered poor design.



## 19. Imperative APIs with Components

Sometimes declarative APIs are insufficient.

Imperative APIs expose methods manually.



### Example

```jsx
useImperativeHandle(ref, () => ({
  focus() {
    inputRef.current.focus();
  }
}));
```



### Common Use Cases

- modals
- focus control
- video players
- animations



## 20. Controlled vs Uncontrolled API Tradeoffs

Senior developers should know when to use each approach.



### Controlled

Pros:

- predictable
- easier debugging
- validation support

Cons:

- more re-renders
- more code



### Uncontrolled

Pros:

- performance
- simpler

Cons:

- less control
- harder synchronization



### Production Strategy

Large libraries often support BOTH.

Example:

```jsx
<Select
  value={value}
  defaultValue="react"
/>
```



## 21. Building Reusable Form Components

Good form systems require:

- validation
- accessibility
- composability
- scalability



### Example Structure

```jsx
<FormField>
  <Label />
  <Input />
  <ErrorMessage />
</FormField>
```



### Best Practices

- support refs
- expose controlled APIs
- accessibility-first
- support validation libraries



## 22. Building Scalable Modal Systems

Bad modal systems become impossible to manage.



### Problems

- nested state
- z-index conflicts
- scroll locking
- accessibility issues



### Better Architecture

Use:

- portal rendering
- context-based modal manager
- centralized modal system



### Example

```jsx
<ModalProvider>
  <App />
</ModalProvider>
```



## 23. Shared Component Libraries

Large companies maintain shared component libraries.



### Benefits

- UI consistency
- faster development
- reduced duplication



### Common Structure

```txt
components/
tokens/
hooks/
utils/
themes/
```



### Real Production Examples

- Material UI
- Chakra UI
- Ant Design



## 24. Design System Fundamentals

A design system is more than components.

It includes:

- typography
- spacing
- colors
- accessibility
- motion
- tokens
- guidelines



### Goals

- consistency
- scalability
- predictable UI



### Core Parts

| Part | Description |
|---|---|
| Tokens | Design variables |
| Components | Reusable UI |
| Guidelines | Usage standards |
| Accessibility | Inclusive UI |



## 25. Themeable Component Architecture

Themeable systems support:

- light mode
- dark mode
- custom branding



### Common Techniques

- CSS variables
- context providers
- design tokens



### Example

```css
:root {
  --primary: blue;
}
```



### Production Insight

Avoid hardcoded colors inside components.

Use:

- tokens
- semantic naming

Instead of:

```css
color: red;
```

Prefer:

```css
color: var(--error-color);
```



## 26. Accessibility in Component Design

Accessibility is a senior-level expectation.



### Important Areas

- keyboard navigation
- focus management
- ARIA attributes
- semantic HTML
- screen readers



### Example

Bad:

```jsx
<div onClick={handleClick}>
```

Good:

```jsx
<button onClick={handleClick}>
```



### Production Insight

Accessibility should be built into component APIs from day one.

Not added later.



## 27. Component Composition Strategies

Composition means:

> combining smaller components to create larger features



### Example

```jsx
<Page>
  <Sidebar />
  <Content />
</Page>
```



### Benefits

- flexibility
- reusability
- scalability



### React Philosophy

React strongly prefers:

- composition over inheritance



## 28. Component Scalability Challenges

As systems grow:

- prop drilling increases
- APIs become bloated
- re-renders increase
- dependencies grow



### Common Problems

| Problem | Cause |
|---|---|
| Prop explosion | Too many props |
| Tight coupling | Shared assumptions |
| Large components | Mixed responsibilities |
| Poor reusability | Business-specific logic |



## 29. Avoiding Prop Explosion

Prop explosion happens when components receive excessive props.



### Bad

```jsx
<Table
  sortable
  pagination
  filtering
  selectable
  exportable
  searchable
/>
```



### Better Approaches

#### Composition

```jsx
<Table>
  <Table.Search />
  <Table.Pagination />
</Table>
```



#### Context

Share internal state via context.



#### Configuration Objects

```jsx
<Table features={{
  sorting: true,
  filtering: true
}} />
```



## 30. Component Pattern Anti-patterns



### 1. Massive Components

Bad:

```jsx
DashboardPage.jsx
```

containing:

- API calls
- UI
- business logic
- state
- validation



### 2. Prop Drilling Everywhere

Use:

- context
- composition
- hooks



### 3. Premature Abstraction

Do not abstract too early.

Bad:

```jsx
SuperMegaUniversalButton
```



### 4. Overusing Context

Context updates can trigger unnecessary re-renders.



### 5. Deep Component Nesting

Too much nesting hurts readability.



## 31. Production-level Component Best Practices



### 1. Keep Components Focused

Single responsibility principle.



### 2. Prefer Composition

Composition scales better than inheritance.



### 3. Build Accessibility First

Never treat accessibility as optional.



### 4. Design Stable APIs

Changing component APIs later is expensive.



### 5. Support Flexibility

Reusable components should adapt to multiple use cases.



### 6. Separate Logic from UI

Use:

- custom hooks
- services
- utility functions



### 7. Avoid Unnecessary Re-renders

Use:

- memoization
- state colocation
- stable references



### 8. Document Components Properly

Important for:

- teams
- onboarding
- design systems



## 32. Common Senior-level Interview Questions



### Conceptual Questions

#### Q1. Why are component patterns important in React?

**Answer:**

Component patterns improve:

- scalability
- maintainability
- reusability
- developer experience

They help teams build consistent and predictable architectures.



#### Q2. Difference between HOC and Custom Hooks?

**Answer:**

HOCs wrap components.

Custom hooks share logic directly using hooks.

Modern React prefers custom hooks because they are simpler and avoid wrapper nesting.



#### Q3. What are compound components?

**Answer:**

Compound components are multiple related components sharing internal state through context.

Example:

- Tabs
- Accordion
- Select

They create highly flexible APIs.



#### Q4. Why is composition preferred over inheritance in React?

**Answer:**

Composition is:

- more flexible
- easier to extend
- simpler to maintain

Inheritance creates tight coupling and rigid hierarchies.



#### Q5. What makes a good reusable component?

**Answer:**

Good reusable components are:

- focused
- composable
- accessible
- configurable
- predictable
- not business-specific



#### Q6. Explain headless UI architecture.

**Answer:**

Headless UI separates logic and accessibility from styling.

Developers control the visual layer while reusing behavior logic.



#### Q7. When would you use controlled vs uncontrolled components?

**Answer:**

Use controlled components when:

- validation
- synchronization
- predictability

matter.

Use uncontrolled components for:

- simple forms
- high-performance forms
- minimal state requirements



## Final Senior-Level Summary

Advanced component patterns are the foundation of scalable React architecture.

Senior developers focus on:

- flexibility
- API design
- composition
- accessibility
- maintainability
- scalability

The biggest difference between junior and senior React developers is often:

> “How they design reusable systems.”