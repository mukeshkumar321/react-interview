# Testing in React

## 📚 Topics Covered

- [1. Why Testing Matters](#1-why-testing-matters)
- [2. Types of Tests](#2-types-of-tests)
- [3. Testing Pyramid](#3-testing-pyramid)
- [4. Jest Basics](#4-jest-basics)
- [5. React Testing Library](#5-react-testing-library)
- [6. Querying Elements](#6-querying-elements)
- [7. User Events](#7-user-events)
- [8. Testing Components](#8-testing-components)
- [9. Testing Async Code](#9-testing-async-code)
- [10. Testing Custom Hooks](#10-testing-custom-hooks)
- [11. Mocking](#11-mocking)
- [12. Testing Context](#12-testing-context)
- [13. Testing with Redux](#13-testing-with-redux)
- [14. Common Testing Patterns](#14-common-testing-patterns)
- [15. Testing Anti-patterns](#15-testing-anti-patterns)
- [16. Common Senior-level Interview Questions](#16-common-senior-level-interview-questions)

---

## 1. Why Testing Matters

Tests give confidence that code works as expected.

Without tests:

- refactoring breaks features silently
- bugs reach production
- onboarding new developers is risky
- deploys become stressful

With tests:

- changes are safe
- regressions are caught early
- code is better designed
- teams move faster long-term

---

## 2. Types of Tests

### Unit Tests

Test a single function or component in isolation.

Fast, cheap, easy to write.

Example:

```tsx
test("adds two numbers", () => {
  expect(add(1, 2)).toBe(3);
});
```

---

### Integration Tests

Test multiple units working together.

Example:

- a form that calls an API on submit
- a component that reads from context

---

### End-to-End (E2E) Tests

Test the full application in a real browser.

Tools:

- Cypress
- Playwright

Slow but most realistic.

---

## 3. Testing Pyramid

```txt
        E2E (few)
      Integration (some)
    Unit Tests (many)
```

More unit tests, fewer E2E tests.

Unit tests are fast and cheap.

E2E tests are slow and expensive.

---

## 4. Jest Basics

Jest is the default test runner for React apps.

### Key Concepts

```js
describe("group name", () => {
  test("test name", () => {
    expect(value).toBe(expected);
  });
});
```

### Common Matchers

```js
expect(x).toBe(1);           // strict equality
expect(x).toEqual({ a: 1 }); // deep equality
expect(x).toBeTruthy();
expect(x).toBeNull();
expect(x).toContain("text");
expect(fn).toHaveBeenCalled();
expect(fn).toHaveBeenCalledWith(arg);
```

### Setup and Teardown

```js
beforeEach(() => { /* runs before each test */ });
afterEach(() => { /* runs after each test */ });
beforeAll(() => { /* runs once before all */ });
afterAll(() => { /* runs once after all */ });
```

---

## 5. React Testing Library

React Testing Library (RTL) tests components the way users use them.

Core philosophy:

> "The more your tests resemble the way your software is used, the more confidence they give you."

RTL encourages:

- querying by visible text
- querying by role
- interacting like a real user
- avoiding implementation details

---

### Basic Setup

```tsx
import { render, screen } from "@testing-library/react";

test("renders heading", () => {
  render(<App />);
  expect(screen.getByText("Hello")).toBeInTheDocument();
});
```

---

## 6. Querying Elements

RTL provides multiple query types.

### Priority Order (best to worst)

| Query | Use When |
|---|---|
| `getByRole` | Most accessible elements |
| `getByLabelText` | Form inputs |
| `getByPlaceholderText` | Inputs with placeholder |
| `getByText` | Non-interactive text |
| `getByTestId` | Last resort only |

---

### Query Variants

| Variant | Throws if missing | Returns |
|---|---|---|
| `getBy` | Yes | Element |
| `queryBy` | No (returns null) | Element or null |
| `findBy` | Yes (async) | Promise |

---

### Examples

```tsx
screen.getByRole("button", { name: "Submit" });
screen.getByLabelText("Email");
screen.getByText("Welcome");
screen.queryByText("Error"); // returns null if not found
await screen.findByText("Loaded"); // waits for element
```

---

## 7. User Events

Use `@testing-library/user-event` to simulate real user interactions.

```tsx
import userEvent from "@testing-library/user-event";

test("types in input", async () => {
  const user = userEvent.setup();

  render(<Input />);

  await user.type(screen.getByRole("textbox"), "hello");

  expect(screen.getByRole("textbox")).toHaveValue("hello");
});
```

---

### Common Interactions

```tsx
await user.click(button);
await user.type(input, "text");
await user.clear(input);
await user.selectOptions(select, "option");
await user.keyboard("{Enter}");
```

---

### Why Not fireEvent?

`fireEvent` dispatches synthetic events.

`userEvent` simulates real browser interactions including focus, hover, keyboard sequences.

Prefer `userEvent` for realistic tests.

---

## 8. Testing Components

### Basic Component Test

```tsx
function Greeting({ name }: { name: string }) {
  return <h1>Hello {name}</h1>;
}

test("renders greeting", () => {
  render(<Greeting name="Mukesh" />);
  expect(screen.getByText("Hello Mukesh")).toBeInTheDocument();
});
```

---

### Testing State

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}

test("increments on click", async () => {
  const user = userEvent.setup();
  render(<Counter />);

  await user.click(screen.getByRole("button"));

  expect(screen.getByText("Count: 1")).toBeInTheDocument();
});
```

---

### Testing Conditional Rendering

```tsx
test("shows error message", () => {
  render(<Form error="Required" />);
  expect(screen.getByText("Required")).toBeInTheDocument();
});

test("hides error when none", () => {
  render(<Form />);
  expect(screen.queryByText("Required")).not.toBeInTheDocument();
});
```

---

## 9. Testing Async Code

### waitFor

Waits until assertion passes.

```tsx
test("loads data", async () => {
  render(<UserList />);

  expect(screen.getByText("Loading...")).toBeInTheDocument();

  await waitFor(() => {
    expect(screen.getByText("John")).toBeInTheDocument();
  });
});
```

---

### findBy (shorthand)

```tsx
test("shows loaded user", async () => {
  render(<UserList />);

  const name = await screen.findByText("John");

  expect(name).toBeInTheDocument();
});
```

---

### Mocking fetch

```tsx
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve([{ id: 1, name: "John" }])
  })
) as jest.Mock;
```

---

## 10. Testing Custom Hooks

Use `renderHook` from RTL.

```tsx
import { renderHook, act } from "@testing-library/react";

function useCounter() {
  const [count, setCount] = useState(0);
  return { count, increment: () => setCount(c => c + 1) };
}

test("increments counter", () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

---

### Why act()?

`act()` ensures all state updates and effects are processed before assertions.

---

## 11. Mocking

### Mock Functions

```tsx
const mockFn = jest.fn();

mockFn("hello");

expect(mockFn).toHaveBeenCalledWith("hello");
```

---

### Mock Modules

```tsx
jest.mock("./api", () => ({
  fetchUsers: jest.fn(() => Promise.resolve([]))
}));
```

---

### Mock Return Values

```tsx
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: [] }); // for async
mockFn.mockRejectedValue(new Error("fail")); // for errors
```

---

### Spying

```tsx
const spy = jest.spyOn(console, "error").mockImplementation(() => {});

// after test
spy.mockRestore();
```

---

## 12. Testing Context

Wrap component with provider in tests.

```tsx
function renderWithTheme(ui: React.ReactElement) {
  return render(
    <ThemeContext.Provider value="dark">
      {ui}
    </ThemeContext.Provider>
  );
}

test("uses theme from context", () => {
  renderWithTheme(<Button />);
  expect(screen.getByRole("button")).toHaveClass("dark");
});
```

---

### Custom Render Utility

Create a shared wrapper for all providers.

```tsx
function AllProviders({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        {children}
      </AuthProvider>
    </ThemeProvider>
  );
}

function customRender(ui: React.ReactElement) {
  return render(ui, { wrapper: AllProviders });
}
```

Reuse across all tests.

---

## 13. Testing with Redux

Wrap with real or mock store.

```tsx
import { Provider } from "react-redux";
import { configureStore } from "@reduxjs/toolkit";

function renderWithStore(ui: React.ReactElement, preloadedState = {}) {
  const store = configureStore({
    reducer: rootReducer,
    preloadedState
  });

  return render(<Provider store={store}>{ui}</Provider>);
}

test("shows user from store", () => {
  renderWithStore(<UserProfile />, {
    user: { name: "John" }
  });

  expect(screen.getByText("John")).toBeInTheDocument();
});
```

---

### Why Real Store?

Using a real store catches integration bugs that mocks miss.

Avoid mocking Redux in most cases.

---

## 14. Common Testing Patterns

### Arrange, Act, Assert

```tsx
// Arrange
render(<Counter />);

// Act
await user.click(screen.getByRole("button"));

// Assert
expect(screen.getByText("Count: 1")).toBeInTheDocument();
```

---

### Testing Error States

```tsx
test("shows error on failed fetch", async () => {
  server.use(
    rest.get("/api/users", (req, res, ctx) => res(ctx.status(500)))
  );

  render(<UserList />);

  await screen.findByText("Something went wrong");
});
```

---

### Testing Loading States

```tsx
test("shows loader while fetching", () => {
  render(<UserList />);
  expect(screen.getByRole("progressbar")).toBeInTheDocument();
});
```

---

### Testing Forms

```tsx
test("submits form", async () => {
  const onSubmit = jest.fn();
  const user = userEvent.setup();

  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText("Email"), "test@example.com");
  await user.type(screen.getByLabelText("Password"), "secret");
  await user.click(screen.getByRole("button", { name: "Login" }));

  expect(onSubmit).toHaveBeenCalledWith({
    email: "test@example.com",
    password: "secret"
  });
});
```

---

## 15. Testing Anti-patterns

### Testing Implementation Details

Bad:

```tsx
expect(component.state.count).toBe(1);
```

This breaks when implementation changes.

Good:

```tsx
expect(screen.getByText("Count: 1")).toBeInTheDocument();
```

---

### Using getByTestId Everywhere

Bad:

```tsx
screen.getByTestId("submit-btn");
```

Adds noise to production code.

Better:

```tsx
screen.getByRole("button", { name: "Submit" });
```

---

### Not Cleaning Up Mocks

```tsx
afterEach(() => {
  jest.clearAllMocks();
});
```

Leftover mocks cause flaky tests.

---

### Snapshot Testing Everything

Snapshot tests are brittle.

Every UI change breaks them.

Use sparingly — only for stable, pure UI components.

---

### Testing Too Many Details

Tests should verify behavior, not internals.

If a test breaks every time you refactor without changing behavior — it is a bad test.

---

## 16. Common Senior-level Interview Questions

---

### Q1. What is the difference between unit and integration tests?

Unit tests isolate a single function or component.

Integration tests verify multiple units working together.

---

### Q2. Why use React Testing Library over Enzyme?

RTL tests user behavior, not implementation.

Enzyme tests component internals which makes tests fragile.

---

### Q3. What is the difference between getBy, queryBy, findBy?

| Query | Behavior |
|---|---|
| `getBy` | Throws if not found |
| `queryBy` | Returns null if not found |
| `findBy` | Async, waits for element |

---

### Q4. When should you mock?

Mock when:

- external API calls
- timers
- modules you do not control

Do not mock:

- internal state
- Redux store (use real store)
- implementation details

---

### Q5. What is act() and why is it needed?

`act()` ensures React processes all state updates and effects before assertions run.

Required when testing hooks or triggering updates manually.

---

### Q6. How do you test a component that uses Context?

Wrap the component with the provider in the test.

Create a custom render utility with all providers for reuse.

---

### Q7. How do you test a component with API calls?

Mock the fetch/axios call using jest.fn() or MSW (Mock Service Worker).

MSW is preferred for realistic network mocking.

---

### Q8. What is MSW?

Mock Service Worker intercepts real network requests in tests.

More realistic than mocking fetch directly.

```tsx
server.use(
  rest.get("/api/users", (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: "John" }]));
  })
);
```

---

### Q9. How do you test a custom hook?

Use `renderHook` from React Testing Library.

Wrap state updates in `act()`.

---

### Q10. What is test coverage and is 100% coverage the goal?

Coverage measures how much code is tested.

100% coverage is not the goal.

Goal is:

- critical paths are tested
- edge cases are covered
- tests give confidence to refactor

High coverage with bad tests gives false confidence.
