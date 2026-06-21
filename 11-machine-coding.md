# Frontend Machine Coding Interview Preparation

> Machine coding interviews test your ability to build scalable, production-ready UI — not just working code.

## Table of Contents

- [1. How to Approach Machine Coding](#1-how-to-approach-machine-coding)
- [2. Before You Start Coding](#2-before-you-start-coding)
- [3. Beginner Questions](#3-beginner-questions)
- [4. Intermediate Questions](#4-intermediate-questions)
- [5. Advanced Questions](#5-advanced-questions)
- [6. Common Mistakes](#6-common-mistakes)
- [7. Preparation Plan](#7-preparation-plan)

---

## 1. How to Approach Machine Coding

For a 4 YOE developer, interviewers don't just check if the UI works. They evaluate:

- **Architecture thinking** — how you break the problem into components
- **State design** — what lives locally vs globally, derived vs stored state
- **Performance awareness** — do you think about re-renders, memoization, virtualization
- **Edge case handling** — loading, error, empty states
- **Code quality** — readable, reusable, maintainable
- **Accessibility** — semantic HTML, keyboard nav, ARIA labels

The mindset shift: think like a **product engineer** building for production, not a developer completing a task.

---

## 2. Before You Start Coding

**The first 5 minutes matter most.** Don't start coding immediately — this alone separates experienced developers.

**Step 1: Clarify requirements**

Ask these questions before writing a single line:
- What features are mandatory vs optional?
- Does it need API integration or mock data?
- Does it need persistence (localStorage)?
- Mobile responsive?
- Accessibility requirements?
- What happens on error/empty/loading states?

**Step 2: Break into components**

```txt
Example: Kanban Board
Board
├── Column
│   ├── Card
│   ├── AddCard
│   └── CardMenu
├── Filters
└── Modal
```

**Step 3: Plan your state**
- What state is local vs shared?
- Use Context or prop drilling?
- Any derived state that shouldn't be stored?
- Any async/server state?

**Step 4: Talk through performance** (even if not asked)
- Will any list need virtualization?
- Any heavy calculations needing `useMemo`?
- Any callbacks passed to memoized children?

**Step 5: Time management — plan your 30/45 minutes**

| Time | What to do |
|---|---|
| 0-5 min | Clarify, plan architecture, announce your approach |
| 5-15 min | Core structure — components, state, routing skeleton |
| 15-30 min | Core features working end-to-end |
| 30-40 min | Edge cases (loading, error, empty states), accessibility |
| 40-45 min | Polish, performance mentions, cleanup |

**If time runs out:** finish the core flow first. A working Todo CRUD beats a half-finished Trello clone. State this to the interviewer: "I'll implement the core flow first, then add error handling and pagination."

**Step 6: Start with structure, then features**
Build the skeleton first (components + state), then fill in logic. Don't jump to edge cases before core flow works.

---

## 3. Beginner Questions

> Goal: Component thinking, basic state, event handling.

### 1. Counter App
**Concepts:** useState, event handling  
**Add:** keyboard shortcuts (+/- keys), localStorage persistence, min/max limits

### 2. Theme Toggle (Dark/Light Mode)
**Concepts:** Context API, localStorage, CSS variables  
**Add:** System preference detection (`prefers-color-scheme`)

### 3. Todo App
**Concepts:** CRUD, list rendering, immutable updates, filters  
**Add:** localStorage persistence, search, priority sorting

### 4. Accordion Component
**Concepts:** Reusable UI, controlled/uncontrolled, state isolation  
**Add:** Single-open vs multi-open mode, smooth animation

### 5. Tabs Component
**Concepts:** Controlled component, accessibility  
**Add:** Keyboard navigation (arrow keys), lazy loading tab content

### 6. Modal Component
**Concepts:** Portals, event bubbling, focus trap  
**Add:** ESC to close, click outside to close, scroll lock on body

### 7. OTP Input Component
**Concepts:** useRef, controlled inputs, keyboard events  
**Add:** Auto-focus next input, paste handling, backspace to go back

### 8. Star Rating Component
**Concepts:** Controlled vs uncontrolled, hover state  
**Add:** Half-star support, read-only mode, accessible with keyboard

### 9. Progress Bar
**Concepts:** CSS transitions, dynamic state  
**Add:** Animated fill, step-based progress, color thresholds

### 10. Infinite Scroll
**Concepts:** Intersection Observer, pagination, loading states  
**Add:** Debounced scroll, skeleton loader, error retry

---

## 4. Intermediate Questions

> Goal: Real interview-level — architecture, async, scalable state.

### 1. File Explorer (Nested Folders)
**Concepts:** Recursive components, tree rendering, expand/collapse  
**Add:** Search/filter, rename, delete, drag-and-drop reorder

### 2. Kanban Board
**Concepts:** Complex state management, drag-and-drop, optimistic updates  
**Add:** Column-level limits, card filtering, localStorage persistence

### 3. Autocomplete Search
**Concepts:** Debouncing, async API, caching, race condition handling  
**Add:** Keyboard navigation in dropdown, highlight matched text, loading state

### 4. Data Table with Sort/Filter/Pagination
**Concepts:** Derived state, optimized rendering, controlled inputs  
**Add:** Multi-column sort, column visibility toggle, row selection, export

### 5. Multi-Step Form
**Concepts:** Form state architecture, validation, step navigation  
**Add:** Draft saving (localStorage), navigation guards ("Are you sure?"), progress indicator

### 6. Nested Comments System
**Concepts:** Recursive rendering, reply threading, collapse/expand  
**Add:** Like/dislike, edit/delete, infinite nesting limit

### 7. Chat Application UI
**Concepts:** Real-time UI thinking, scroll management, optimistic messages  
**Add:** Typing indicator, unread count, grouped messages by date

### 8. E-commerce Product Page
**Concepts:** State architecture, cart management, variant selection  
**Add:** Image gallery zoom, stock management, add-to-cart animation

### 9. Image Carousel
**Concepts:** useRef for DOM, touch/swipe, autoplay with cleanup  
**Add:** Lazy load images, thumbnail preview, keyboard navigation

### 10. Video Player (Custom Controls)
**Concepts:** Media APIs, refs, event handling  
**Add:** Fullscreen, playback speed, volume, time scrubber, keyboard shortcuts

---

## 5. Advanced Questions

> Goal: Senior-level — scalable architecture, performance engineering, production thinking.

### 1. Trello Clone
**Concepts:** Drag-and-drop between lists, optimistic updates, complex state  
**Add:** Real API integration, undo/redo, card labels/due dates, filters

### 2. Dynamic Form Builder
**Concepts:** Schema-driven rendering, dynamic validation, drag-and-drop fields  
**Add:** JSON export/import, conditional logic (show field B if A = X), preview mode

### 3. Virtualized Data Grid (10k+ rows)
**Concepts:** Virtualization (react-window), sticky headers, column resize  
**Add:** Virtual scrolling both axes, cell editing, row grouping

### 4. Real-Time Dashboard
**Concepts:** WebSocket updates, live charts, efficient re-renders  
**Add:** Pause/resume updates, historical view, alert thresholds

### 5. Rich Text Editor
**Concepts:** contenteditable challenges, toolbar state, serialization  
**Add:** Markdown support, image upload, keyboard shortcuts, export HTML

### 6. Design System / Component Library
**Concepts:** Scalable component architecture, theming, polymorphic components  
**Add:** Token-based theming, accessibility first, documentation (Storybook)

### 7. Frontend Architecture Challenge
**Concepts:** Auth flow, role-based access, routing, caching, feature modules  
**Add:** Lazy-loaded routes, refresh token handling, permission gates

---

## 6. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Start coding immediately | Always spend 5 mins on architecture first |
| One giant component | Break into small, focused components |
| No loading/error/empty states | Always handle all 3 async states |
| `key={index}` in dynamic lists | Always use stable unique IDs |
| State stored that can be derived | Compute it instead of storing |
| No cleanup in useEffect | Always clean up subscriptions/timers |
| Ignoring accessibility | Add semantic HTML + ARIA from the start |
| Prop drilling 3+ levels | Use Context or lift state appropriately |
| No performance mention | Always proactively mention memoization/virtualization |

---

## 7. Preparation Plan

**Week 1-2 — Beginner problems**
Build all 10 beginner problems. Focus on clean component structure and state design.

**Week 3-5 — Intermediate problems**
Build 6-7 intermediate problems. Focus on async handling, scalable state, accessibility.

**Week 6-8 — Advanced problems**
Pick 3-4 advanced problems. Focus on production-ready architecture and performance.

**For every problem, always add:**
- ✅ Loading, error, and empty states
- ✅ localStorage persistence where it makes sense
- ✅ Keyboard navigation and accessibility
- ✅ At least one performance optimization
- ✅ Clean folder structure

> **Remember:** Interviewers want to see how you think, not just what you build. Talking through decisions out loud is as important as the code itself.

---

## 8. Complete Worked Example — Todo App

> This shows how to think through a "simple" problem the senior way.

**Problem:** Build a Todo app with add, toggle complete, and delete.

---

### Step 1: Clarify (2 min)

Before coding, ask:
- Can todos be edited after creation?
- Should todos persist after page refresh?
- Any categories or priority levels?
- Keyboard accessible?

Assume: basic CRUD, localStorage persistence, no categories.

---

### Step 2: Component Breakdown (2 min)

```txt
TodoApp
├── TodoInput       ← controlled input + add button
├── TodoFilters     ← All / Active / Completed
├── TodoList
│   └── TodoItem    ← checkbox + text + delete
└── TodoStats       ← "3 of 5 completed"
```

---

### Step 3: State Design (1 min)

```js
// What to store
const [todos, setTodos] = useState([]);
const [filter, setFilter] = useState("all"); // "all" | "active" | "completed"

// What NOT to store — derive it
const filteredTodos = todos.filter(todo => {
  if (filter === "active") return !todo.completed;
  if (filter === "completed") return todo.completed;
  return true;
});
```

Key insight: `filteredTodos` is derived, not stored. Storing it separately creates sync bugs.

---

### Step 4: Core Implementation

```jsx
// types
// { id: string, text: string, completed: boolean }

function TodoApp() {
  const [todos, setTodos] = useState(() => {
    // lazy init — reads localStorage only once on mount
    const saved = localStorage.getItem("todos");
    return saved ? JSON.parse(saved) : [];
  });
  const [filter, setFilter] = useState("all");
  const [input, setInput] = useState("");

  // Sync to localStorage on every todos change
  useEffect(() => {
    localStorage.setItem("todos", JSON.stringify(todos));
  }, [todos]);

  const addTodo = () => {
    if (!input.trim()) return;
    setTodos(prev => [
      ...prev,
      { id: crypto.randomUUID(), text: input.trim(), completed: false },
    ]);
    setInput("");
  };

  const toggleTodo = (id) => {
    setTodos(prev =>
      prev.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
    );
  };

  const deleteTodo = (id) => {
    setTodos(prev => prev.filter(t => t.id !== id));
  };

  const filteredTodos = todos.filter(t => {
    if (filter === "active") return !t.completed;
    if (filter === "completed") return t.completed;
    return true;
  });

  return (
    <div>
      <TodoInput value={input} onChange={setInput} onAdd={addTodo} />
      <TodoFilters current={filter} onChange={setFilter} />
      <TodoList todos={filteredTodos} onToggle={toggleTodo} onDelete={deleteTodo} />
      <TodoStats total={todos.length} completed={todos.filter(t => t.completed).length} />
    </div>
  );
}
```

---

### Step 5: What to mention to the interviewer

- **No `key={index}`** — using stable IDs from `crypto.randomUUID()`
- **Lazy `useState` init** — avoids reading localStorage on every render
- **Immutable updates** — using `.map()` and `.filter()` instead of mutating
- **Derived state** — `filteredTodos` computed, not stored
- **Performance:** for 10k+ todos, would add virtualization with react-window
- **Accessibility:** TodoInput uses `onKeyDown` for Enter key submission, `<button>` for delete has aria-label

This is what separates a 4 YOE answer from a junior answer — not just making it work, but explaining the decisions.
