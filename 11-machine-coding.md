# Machine Coding

## 📚 Topics Covered

- [1. Introduction to React Machine Coding](#1-introduction-to-react-machine-coding)
- [2. Machine Coding Interview Strategy](#2-machine-coding-interview-strategy)
- [3. Requirement Analysis Techniques](#3-requirement-analysis-techniques)
- [4. Component Breakdown Strategy](#4-component-breakdown-strategy)
- [5. State Design for Machine Coding](#5-state-design-for-machine-coding)
- [6. Folder Structure During Interviews](#6-folder-structure-during-interviews)
- [7. Reusable Component Design](#7-reusable-component-design)
- [8. Scalable State Management in Machine Coding](#8-scalable-state-management-in-machine-coding)
- [9. API Integration Patterns](#9-api-integration-patterns)
- [10. Loading and Error Handling](#10-loading-and-error-handling)
- [11. Search Functionality Implementation](#11-search-functionality-implementation)
- [12. Debounced Search Implementation](#12-debounced-search-implementation)
- [13. Pagination Implementation](#13-pagination-implementation)
- [14. Infinite Scroll Implementation](#14-infinite-scroll-implementation)
- [15. Data Table Implementation](#15-data-table-implementation)
- [16. Sorting and Filtering Systems](#16-sorting-and-filtering-systems)
- [17. Modal System Implementation](#17-modal-system-implementation)
- [18. Toast Notification System](#18-toast-notification-system)
- [19. Accordion Component](#19-accordion-component)
- [20. Tabs Component](#20-tabs-component)
- [21. Dropdown Component](#21-dropdown-component)
- [22. Multi-select Component](#22-multi-select-component)
- [23. Autocomplete Component](#23-autocomplete-component)
- [24. Theme Toggle System](#24-theme-toggle-system)
- [25. Multi-step Form Implementation](#25-multi-step-form-implementation)
- [26. Dynamic Form Builder](#26-dynamic-form-builder)
- [27. File Upload System](#27-file-upload-system)
- [28. Drag and Drop Implementation](#28-drag-and-drop-implementation)
- [29. Kanban Board Implementation](#29-kanban-board-implementation)
- [30. Chat UI Architecture](#30-chat-ui-architecture)
- [31. Real-time UI Concepts](#31-real-time-ui-concepts)
- [32. Optimistic UI Implementation](#32-optimistic-ui-implementation)
- [33. Virtualized List Implementation](#33-virtualized-list-implementation)
- [34. Performance Optimization in Machine Coding](#34-performance-optimization-in-machine-coding)
- [35. Accessibility in Machine Coding](#35-accessibility-in-machine-coding)
- [36. Error Boundary Usage](#36-error-boundary-usage)
- [37. Clean Code Practices During Interviews](#37-clean-code-practices-during-interviews)
- [38. Common Machine Coding Mistakes](#38-common-machine-coding-mistakes)
- [39. Production-level Thinking in Interviews](#39-production-level-thinking-in-interviews)
- [40. Senior-level Machine Coding Discussions](#40-senior-level-machine-coding-discussions)

---


## 1. Introduction to React Machine Coding

### What is a Machine Coding Round?

A machine coding round is a practical frontend interview where you build a small application or feature within a limited time.

#### Typical Duration
- 60–120 minutes

#### Typical Expectations
- Clean UI
- Working functionality
- Proper state management
- Good architecture
- Reusable components
- Edge-case handling
- Performance awareness



### Why Companies Conduct Machine Coding Rounds

Companies evaluate:

| Area | What They Check |
|---|---|
| React knowledge | Hooks, rendering, state |
| Problem solving | Breaking large problems into smaller parts |
| Architecture thinking | Folder structure, scalability |
| Code quality | Naming, readability |
| Performance | Re-renders, optimization |
| UX understanding | Loading states, error handling |
| Seniority | Tradeoffs and decision-making |



### Common Machine Coding Questions

#### Beginner-Level
- Counter App
- Todo App
- Accordion
- Tabs
- Modal
- Theme Toggle

#### Mid-Level
- Data Table
- Pagination
- Search + Filter
- Infinite Scroll
- Kanban Board
- Chat UI

#### Senior-Level
- Dynamic Form Builder
- Dashboard Architecture
- Real-time UI
- Virtualized List
- Optimistic Updates
- Complex State Systems



### What Interviewers Observe Carefully

Interviewers care more about:
- Structure
- Thinking process
- Maintainability

than perfect UI design.

They observe:
- How you start
- How you organize files
- How you manage state
- How reusable your code is
- How you communicate decisions



### Ideal Development Flow During Interviews

#### Step 1 — Requirement Clarification

Ask:
- Is pagination server-side or client-side?
- Should search be debounced?
- Should data persist on refresh?
- Are we handling mobile responsiveness?

#### Step 2 — Component Planning

Break UI into:
- Container components
- Reusable UI components
- Utility functions

#### Step 3 — State Planning

Identify:
- Global state
- Local state
- Derived state

#### Step 4 — Build MVP First

Focus on:
- Functionality
- Architecture
- Correctness

before:
- Animations
- Styling polish

#### Step 5 — Optimize

Then improve:
- Performance
- Accessibility
- Code cleanup



## 2. Machine Coding Interview Strategy

### Most Important Rule

Do NOT start coding immediately.

Senior candidates first:
1. Understand requirements
2. Discuss architecture
3. Plan components
4. Identify state
5. Then start coding

This differentiates senior engineers from beginners.



### Recommended Interview Timeline

| Time | Task |
|---|---|
| First 5–10 min | Requirement analysis |
| Next 10 min | Component planning |
| Main coding time | Build MVP |
| Final 10–15 min | Optimization + cleanup |



### How Senior Developers Approach Problems

#### Bad Approach
Start writing code instantly.

#### Good Approach

Think first:
- What are the entities?
- What components exist?
- Where should state live?
- Which parts are reusable?
- Which APIs are needed?



### Think in Layers

#### UI Layer
- Buttons
- Inputs
- Cards
- Tables

#### Business Logic Layer
- Search
- Filtering
- Pagination

#### Data Layer
- API handling
- Caching
- State synchronization



### Prioritize Features Properly

#### High Priority
- Working functionality
- Stable state
- Error handling

#### Medium Priority
- Reusability
- Performance

#### Low Priority
- Fancy UI
- Complex animations



### Communication Strategy

Interviewers evaluate communication heavily.

Continuously explain:
- Why you chose something
- Alternatives considered
- Tradeoffs

#### Example
> "I'm keeping pagination state in parent component because multiple child components depend on it."



## 3. Requirement Analysis Techniques

### Why Requirement Analysis Matters

Many candidates fail because they assume requirements incorrectly.

Senior engineers clarify ambiguity early.



### Important Questions to Ask

#### Data Questions
- Is data static or API-driven?
- Should data persist?
- Is caching needed?

#### UI Questions
- Responsive design needed?
- Accessibility expectations?
- Dark mode support?

#### Feature Questions
- Sorting?
- Filtering?
- Multi-select?
- Infinite scroll?

#### Performance Questions
- Large datasets?
- API rate limits?
- Virtualization needed?



### Example Analysis

#### Problem
Build a product listing page.

#### Senior-Level Clarifications
- Client-side or server-side pagination?
- Search debounce needed?
- Should filters sync with URL?
- API failure handling?
- Loading skeleton required?



### Convert Requirements into Technical Tasks

#### Requirement
"Search products"

#### Technical Tasks
- Search input
- Debounce logic
- API integration
- Loading state
- Empty state
- Error handling



### Avoid Overengineering

Do not build:
- Redux
- Complex abstractions
- Unnecessary hooks

unless problem scale demands it.



## 4. Component Breakdown Strategy

### Why Component Breakdown Matters

Poor component design creates:
- Prop drilling
- Duplicated logic
- Hard maintenance



### Ideal Breakdown Principles

A component should:
- Have one responsibility
- Be reusable
- Be independently testable



### Example: Data Table

#### Parent Component
`ProductsPage`
- API calls
- Pagination state
- Filter state

#### Child Components
- SearchBar
- FilterPanel
- ProductTable
- Pagination
- EmptyState
- Loader



### Smart vs Dumb Components

#### Smart Components
Contain:
- State
- API calls
- Business logic

#### Dumb Components
Contain:
- Presentation logic only

#### Example

```jsx
<SearchBar value={search} onChange={setSearch} />
```



### Common Mistake

#### Bad
Huge 1000-line component.

#### Better
Split by:
- Responsibility
- Feature
- Reuse potential



## 5. State Design for Machine Coding

### Most Important React Skill

Senior frontend interviews heavily evaluate:
- State modeling
- State ownership
- State flow



### Types of State

| State Type | Example |
|---|---|
| Local UI State | Modal open |
| Shared State | User data |
| Server State | API response |
| Derived State | Filtered results |



### State Design Principles

#### Keep State Minimal

Avoid:

```js
const [filteredUsers, setFilteredUsers]
```

Prefer:

```js
const filteredUsers = users.filter(...)
```

Derived data should usually not be stored.



### Lift State Carefully

State should live:
- At the nearest common parent

Avoid unnecessary global state.



### Avoid Duplicate State

#### Bad

```js
const [users, setUsers]
const [filteredUsers, setFilteredUsers]
```

#### Better

```js
const filteredUsers = useMemo(...)
```



### Common Interview Discussion

#### Why not Redux?

Because:
- Local state sufficient
- Small application
- Avoid overengineering

This answer shows maturity.



## 6. Folder Structure During Interviews

### Why Folder Structure Matters

Interviewers evaluate:
- Scalability thinking
- Organization
- Maintainability



### Recommended Structure

```txt
src/
├── components/
├── pages/
├── hooks/
├── services/
├── utils/
├── context/
├── constants/
├── types/
└── App.jsx
```



### Feature-Based Structure

For larger apps:

```txt
features/
├── products/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   └── utils/
```



### Senior-Level Discussion

Explain:

> "I'm separating feature logic to improve scalability and maintainability."



### Avoid Deep Nesting

Bad:

```txt
components/ui/shared/common/base/
```

Keep structure practical.



## 7. Reusable Component Design

### Reusability Principles

Reusable components should:
- Be configurable
- Avoid business logic coupling
- Support composition



### Example Button Component

```jsx
<Button
  variant="primary"
  size="md"
  disabled={loading}
>
  Save
</Button>
```



### Avoid Hardcoded Logic

#### Bad

```jsx
<LoginButton />
```

#### Better

```jsx
<Button />
```



### Composition Over Configuration

#### Better

```jsx
<Card>
  <Card.Header />
  <Card.Body />
</Card>
```

instead of huge prop-based APIs.



### Senior-Level Insight

Too much abstraction early is harmful.

Build reusable systems gradually.



## 8. Scalable State Management in Machine Coding

### Common Options

| Tool | Use Case |
|---|---|
| useState | Local state |
| useReducer | Complex logic |
| Context API | Shared UI state |
| Zustand | Lightweight global state |
| Redux Toolkit | Enterprise-scale apps |



### When to Use useReducer

Good for:
- Form logic
- Multiple related state updates
- Complex transitions

#### Example

```js
dispatch({ type: "ADD_TODO" })
```



### Avoid Context Overuse

Context re-renders entire consumers.

Bad for:
- Rapidly changing state
- Large applications



### Senior-Level Discussion

Mention:
- State normalization
- Memoization
- Selector-based optimization



## 9. API Integration Patterns

### Standard Flow

```txt
UI → Service Layer → API → State Update
```



### Service Layer Pattern

#### Bad
API calls directly inside components everywhere.

#### Better

```js
// services/products.js
export const fetchProducts = () => axios.get(...)
```



### Benefits
- Reusable
- Testable
- Centralized error handling



### API States

Always handle:
- Loading
- Success
- Error
- Empty state



### Production-Level Insight

Discuss:
- Retries
- Cancellation
- Race conditions

#### Example

```js
AbortController
```



## 10. Loading and Error Handling

### Why It Matters

Many candidates ignore UX states.

Senior engineers never do.



### Loading State Types

| Type | Usage |
|---|---|
| Spinner | Small actions |
| Skeleton | Page loading |
| Shimmer | Content placeholders |



### Error Handling

Always show:
- Retry button
- Meaningful message

#### Example

```jsx
if (error) {
  return <ErrorState onRetry={fetchData} />
}
```



### Empty State

Never show blank screens.

#### Example

```jsx
"No products found"
```



## 11. Search Functionality Implementation

### Core Flow

```txt
Input → State Update → Filter/API → Render
```



### Client-Side Search

```js
const filtered = users.filter(user =>
  user.name.toLowerCase().includes(search.toLowerCase())
)
```



### Server-Side Search

```txt
/search?q=react
```

Better for:
- Huge datasets
- Scalability



### Interview Insight

Discuss:
- Debounce
- API cancellation
- Search indexing



## 12. Debounced Search Implementation

### Why Debouncing Matters

Without debounce:
- Excessive API calls
- Poor performance



### Example

```js
useEffect(() => {
  const timer = setTimeout(() => {
    fetchResults(search)
  }, 500)

  return () => clearTimeout(timer)
}, [search])
```



### Production-Level Improvements

Use:
- AbortController
- Latest-request-only logic



## 13. Pagination Implementation

### Types

| Type | Usage |
|---|---|
| Client-side | Small datasets |
| Server-side | Scalable systems |



### Basic State

```js
const [page, setPage] = useState(1)
```



### Server Pagination API

```txt
/products?page=1&limit=10
```



### Important Discussion

Explain:
- Total pages
- Caching
- Resetting page after search



## 14. Infinite Scroll Implementation

### Core Idea

Load more data while scrolling.



### Common Technique

Use:

```js
IntersectionObserver
```

instead of scroll listeners.



### Important Challenges

- Duplicate requests
- Race conditions
- Memory growth



### Senior-Level Discussion

Mention:
- Virtualization
- Cursor-based pagination



## 15. Data Table Implementation

### Important Features

- Sorting
- Filtering
- Pagination
- Row selection



### Architecture

#### Parent
- Manages state

#### Table Component
- Purely presentational



### Performance Concerns

Large tables require:
- Memoization
- Virtualization



### Accessibility

Use:

```html
<table>
```

instead of div-based fake tables.



## 16. Sorting and Filtering Systems

### Sorting Example

```js
data.sort((a, b) => a.price - b.price)
```



### Filtering Example

```js
products.filter(p => p.category === selected)
```



### Production-Level Concerns

- Stable sorting
- Multi-filter support
- URL synchronization



## 17. Modal System Implementation

### Key Challenges

- Portal rendering
- Keyboard accessibility
- Focus trapping



### Best Practice

Use:

```jsx
createPortal()
```



### Accessibility

Support:
- ESC close
- Tab navigation
- ARIA attributes



## 18. Toast Notification System

### Purpose

Provide non-blocking feedback.



### Common Features

- Auto-dismiss
- Variants
- Stacking



### Example Structure

```txt
ToastProvider
  └── ToastContainer
```



### Senior-Level Insight

Use centralized notification management.



## 19. Accordion Component

### Core Concepts

- Controlled vs uncontrolled
- Single-open vs multi-open



### Example State

```js
const [openId, setOpenId]
```



### Accessibility

Use:
- Keyboard navigation
- aria-expanded



## 20. Tabs Component

### Important Considerations

- Active tab state
- Keyboard navigation
- Lazy rendering



### Example

```js
const [activeTab, setActiveTab]
```



### Accessibility

Support:
- Arrow key navigation
- ARIA roles



## 21. Dropdown Component

### Common Challenges

- Outside click handling
- Keyboard navigation
- Positioning



### Example Logic

```js
useEffect(() => {
  document.addEventListener(...)
})
```



### Senior-Level Discussion

Mention:
- Portals
- Floating UI libraries



## 22. Multi-select Component

### Important Features

- Chip rendering
- Search
- Keyboard support



### State Example

```js
const [selectedItems, setSelectedItems]
```



### Performance Concerns

Large option lists may require virtualization.



## 23. Autocomplete Component

### Core Flow

```txt
Input → Debounce → API → Suggestions
```



### Important Features

- Highlighted selection
- Keyboard navigation
- Loading states



### Common Problems

- Stale requests
- Race conditions



## 24. Theme Toggle System

### Typical Approach

Use:
- Context API
- CSS variables



### Persistence

```js
localStorage.setItem("theme", "dark")
```



### Production-Level Discussion

Mention:
- prefers-color-scheme
- SSR hydration issues



## 25. Multi-step Form Implementation

### Challenges

- Validation
- Navigation
- Persistence



### Architecture

```txt
Stepper
├── Step1
├── Step2
└── Step3
```



### Senior-Level Insight

Keep form state centralized.



## 26. Dynamic Form Builder

### Core Idea

Render forms from configuration.



### Example

```js
fields.map(field => ...)
```



### Benefits

- Scalability
- Configurability



### Senior-Level Discussion

Mention:
- Schema-driven forms
- Validation abstraction



## 27. File Upload System

### Important Features

- Progress tracking
- Validation
- Previews



### Example APIs

```js
FormData
```



### Production-Level Concerns

- Chunk uploads
- Retry mechanisms
- File size limits



## 28. Drag and Drop Implementation

### Common Libraries

- dnd-kit
- react-beautiful-dnd



### Challenges

- Accessibility
- Performance
- Mobile support



### Senior-Level Discussion

Mention:
- Collision detection
- Virtualization compatibility



## 29. Kanban Board Implementation

### Core Features

- Drag-drop columns
- Task movement
- Ordering



### State Design

Normalize data:

```js
{
  columns: {},
  tasks: {}
}
```



### Performance

Avoid rerendering entire board.



## 30. Chat UI Architecture

### Important Features

- Message grouping
- Scrolling
- Optimistic messages



### State Challenges

- Real-time updates
- Pagination
- Unread counts



### Performance Concerns

Use virtualization for large chats.



## 31. Real-time UI Concepts

### Common Technologies

- WebSockets
- SSE
- Polling



### Challenges

- Reconnection
- Synchronization
- Stale state



### Interview Insight

Explain:
- Eventual consistency
- Optimistic updates



## 32. Optimistic UI Implementation

### Core Idea

Update UI before server confirmation.



### Example

```js
setTodos(prev => [...prev, newTodo])
```

before API success.



### Rollback Strategy

Revert state if request fails.



### Senior-Level Discussion

Mention:
- Temporary IDs
- Conflict resolution



## 33. Virtualized List Implementation

### Why Virtualization Matters

Rendering 10,000 items kills performance.



### Libraries

- react-window
- react-virtualized



### Core Idea

Render only visible rows.



### Interview Insight

Discuss:
- Dynamic row heights
- Windowing tradeoffs



## 34. Performance Optimization in Machine Coding

### Common Optimizations

| Technique | Purpose |
|---|---|
| React.memo | Prevent rerenders |
| useMemo | Memoize values |
| useCallback | Stable functions |
| Code splitting | Smaller bundles |
| Virtualization | Large lists |



### Most Important Rule

Do NOT optimize blindly.

Measure first.



### Common Causes of Slow Apps

- Unnecessary rerenders
- Large DOM trees
- Expensive calculations



## 35. Accessibility in Machine Coding

### Why Accessibility Matters

Senior frontend engineers must build inclusive UIs.



### Important Areas

- Semantic HTML
- Keyboard support
- Screen readers
- Focus management



### Common Accessibility Attributes

```html
aria-label
aria-expanded
role="dialog"
```



### Interview Insight

Accessibility awareness strongly differentiates senior candidates.



## 36. Error Boundary Usage

### Purpose

Prevent entire app crashes.



### Example

```jsx
<ErrorBoundary>
  <Dashboard />
</ErrorBoundary>
```



### What Error Boundaries Catch

- Rendering errors
- Lifecycle errors



### What They Don't Catch

- Async errors
- Event handler errors



## 37. Clean Code Practices During Interviews

### Important Rules

#### Use Good Naming

Bad:

```js
x
```

Better:

```js
filteredProducts
```



#### Keep Functions Small

Bad:
- Giant handlers

Better:
- Split responsibilities



#### Avoid Repetition

Extract reusable utilities.



### Interviewer Expectations

Readable code is more valuable than clever code.



## 38. Common Machine Coding Mistakes

### Most Common Failures

#### Starting Without Planning

Creates architecture problems later.



#### Overengineering

Avoid:
- Unnecessary Redux
- Excessive abstractions



#### Ignoring Edge Cases

Examples:
- Loading
- Empty states
- API failures



#### Huge Components

Split responsibilities properly.



#### Poor Communication

Explain tradeoffs continuously.



## 39. Production-level Thinking in Interviews

### What Senior Engineers Discuss

#### Scalability
- Growing datasets
- Modular architecture

#### Reliability
- Retries
- Caching
- Error handling

#### Performance
- Rerender optimization
- Virtualization

#### User Experience
- Skeleton loaders
- Optimistic UI
- Accessibility



### Example Senior Discussion

> "If dataset grows significantly, I'd move filtering to server-side and introduce virtualization."

This shows engineering maturity.



## 40. Senior-level Machine Coding Discussions

### What Separates Senior Candidates

Not:
- Fancy UI

But:
- Architecture decisions
- Scalability thinking
- Tradeoff analysis
- Maintainability



### Senior-Level Topics to Discuss

#### Tradeoffs
- Context vs Zustand
- Pagination vs infinite scroll
- Client filtering vs server filtering



#### Scalability
- Feature-based architecture
- Code splitting
- Caching strategies



#### Reliability
- Optimistic rollback
- Retries
- Offline handling



#### Performance
- Memoization
- Virtualization
- Render optimization



### Final Interview Advice

During machine coding rounds:
- Communicate continuously
- Build MVP first
- Prioritize functionality
- Keep architecture clean
- Explain tradeoffs
- Avoid overengineering

A clean, maintainable solution usually beats a flashy but unstable implementation.