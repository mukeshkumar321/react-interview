# Testing in React

## Topics Covered

- [1. Why Testing Matters](#1-why-testing-matters)
- [2. Types of Tests](#2-types-of-tests)
- [3. Jest Basics](#3-jest-basics)
- [4. React Testing Library](#4-react-testing-library)
- [5. Querying Elements](#5-querying-elements)
- [6. User Events](#6-user-events)
- [7. Testing Components](#7-testing-components)
- [8. Testing Async Code](#8-testing-async-code)
- [9. Testing Custom Hooks](#9-testing-custom-hooks)
- [10. Common Patterns & Best Practices](#10-common-patterns--best-practices)

---

Tests give confidence that code works as expected. Without tests, refactoring breaks features silently, bugs reach production, onboarding is risky, and deploys are stressful. With tests, changes are safe, regressions are caught early, code is better designed, and teams move faster long-term.

---

## 1. Why Testing Matters

**Interview Focus:** Understanding the "why" shows maturity - commonly asked in behavioral questions.

**Benefits of testing:**

| Benefit | Impact |
|---|---|
| **Catch bugs early** | Before production |
| **Safer refactoring** | Confidence to change code |
| **Better code design** | Testable code is better code |
| **Documentation** | Tests show how code should work |
| **Team velocity** | Fewer production bugs, faster iteration |

**Senior insight:** Good tests focus on behavior, not implementation. Tests should verify "what" the code does, not "how" it does it.

---

## 2. Types of Tests

**Interview Focus:** Understanding the testing pyramid is essential.

**Unit Tests:** Test a single function or component in isolation. Fast, cheap, and easy to write.

```tsx
test("adds two numbers", () => {
  expect(add(1, 2)).toBe(3);
});
```

**Integration Tests:** Test multiple units working together - for example, a form that calls an API on submit, or a component that reads from context.

**End-to-End (E2E) Tests:** Test the full application in a real browser using tools like Cypress or Playwright. Slow but most realistic.

**Testing Pyramid:**

```txt
        E2E (few)
      Integration (some)
    Unit Tests (many)
```

More unit tests, fewer E2E tests. Unit tests are fast and cheap; E2E tests are slow and expensive.

---

## 3. Jest Basics

**Interview Focus:** Jest is the standard - understanding matchers and setup is fundamental.

Jest is the default test runner for React apps.

```js
describe("Calculator", () => {
  test("adds numbers", () => {
    expect(add(1, 2)).toBe(3);
  });
});
```

**Common matchers:**

```js
expect(x).toBe(1);           // strict equality
expect(x).toEqual({ a: 1 }); // deep equality
expect(x).toBeTruthy();
expect(x).toBeNull();
expect(x).toContain("text");
expect(fn).toHaveBeenCalled();
expect(fn).toHaveBeenCalledWith(arg);
```

**Setup and teardown:**

```js
beforeEach(() => { /* runs before each test */ });
afterEach(() => { /* runs after each test */ });
```

**Mocking:**

```tsx
const mockFn = jest.fn();
mockFn("hello");
expect(mockFn).toHaveBeenCalledWith("hello");

// Mock return values
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: [] }); // for async
mockFn.mockRejectedValue(new Error("fail")); // for errors
```

---

## 4. React Testing Library

**Interview Focus:** RTL is the industry standard - philosophy and approach are commonly tested.

React Testing Library (RTL) tests components the way users use them.

**Core philosophy:** "The more your tests resemble the way your software is used, the more confidence they give you."

RTL encourages:
- Querying by visible text
- Querying by role
- Interacting like a real user
- Avoiding implementation details

Basic setup:

```tsx
import { render, screen } from "@testing-library/react";

test("renders heading", () => {
  render(<App />);
  expect(screen.getByText("Hello")).toBeInTheDocument();
});
```

**Why RTL over Enzyme:** RTL tests user behavior, not component internals. Enzyme tests component state/props which makes tests fragile. RTL tests are more resilient to refactoring.

---

## 5. Querying Elements

**Interview Focus:** Understanding query priorities is critical - commonly tested in practical exercises.

**Priority order (best to worst):**

| Query | Use When |
|---|---|
| `getByRole` | Most accessible elements |
| `getByLabelText` | Form inputs |
| `getByPlaceholderText` | Inputs with placeholder |
| `getByText` | Non-interactive text |
| `getByTestId` | Last resort only |

**Query variants:**

| Variant | Throws if missing | Returns | Use case |
|---|---|---|---|
| `getBy` | Yes | Element | Element must exist |
| `queryBy` | No (returns null) | Element or null | Check absence |
| `findBy` | Yes (async) | Promise | Async elements |

**Examples:**

```tsx
screen.getByRole("button", { name: "Submit" });
screen.getByLabelText("Email");
screen.getByText("Welcome");
screen.queryByText("Error"); // returns null if not found
await screen.findByText("Loaded"); // waits for element
```

**Important:** Prefer `getByRole` - it encourages accessibility and semantic HTML.

---

## 6. User Events

**Interview Focus:** Understanding realistic interactions - commonly used in practical tests.

Use `@testing-library/user-event` to simulate real user interactions:

```tsx
import userEvent from "@testing-library/user-event";

test("types in input", async () => {
  const user = userEvent.setup();
  render(<Input />);
  await user.type(screen.getByRole("textbox"), "hello");
  expect(screen.getByRole("textbox")).toHaveValue("hello");
});
```

**Common interactions:**

```tsx
await user.click(button);
await user.type(input, "text");
await user.clear(input);
await user.selectOptions(select, "option");
await user.keyboard("{Enter}");
```

**Why not fireEvent?** `fireEvent` dispatches synthetic events; `userEvent` simulates real browser interactions including focus, hover, and keyboard sequences. Prefer `userEvent` for realistic tests.

---

## 7. Testing Components

**Interview Focus:** Most common practical interview question - testing component behavior.

**Basic component test:**

```tsx
function Greeting({ name }: { name: string }) {
  return <h1>Hello {name}</h1>;
}

test("renders greeting", () => {
  render(<Greeting name="Mukesh" />);
  expect(screen.getByText("Hello Mukesh")).toBeInTheDocument();
});
```

**Testing state:**

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

**Testing conditional rendering:**

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

**Testing Context:** Wrap component with provider:

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

**Custom render utility** for all providers:

```tsx
function customRender(ui: React.ReactElement) {
  return render(ui, { 
    wrapper: ({ children }) => (
      <ThemeProvider>
        <AuthProvider>
          {children}
        </AuthProvider>
      </ThemeProvider>
    )
  });
}
```

---

## 8. Testing Async Code

**Interview Focus:** Essential for real-world apps - commonly tested in practical exercises.

**waitFor:** Waits until assertion passes.

```tsx
test("loads data", async () => {
  render(<UserList />);
  expect(screen.getByText("Loading...")).toBeInTheDocument();
  await waitFor(() => {
    expect(screen.getByText("John")).toBeInTheDocument();
  });
});
```

**findBy (shorthand):**

```tsx
test("shows loaded user", async () => {
  render(<UserList />);
  const name = await screen.findByText("John");
  expect(name).toBeInTheDocument();
});
```

**Mocking fetch:**

```tsx
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve([{ id: 1, name: "John" }])
  })
) as jest.Mock;

test("fetches and displays users", async () => {
  render(<UserList />);
  expect(await screen.findByText("John")).toBeInTheDocument();
});
```

**Testing error states:**

```tsx
test("shows error on failed fetch", async () => {
  global.fetch = jest.fn(() => Promise.reject(new Error("Failed")));
  
  render(<UserList />);
  
  expect(await screen.findByText("Error loading users")).toBeInTheDocument();
});
```

**MSW (Mock Service Worker)** is preferred for realistic network mocking:

```tsx
server.use(
  rest.get("/api/users", (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: "John" }]));
  })
);
```

---

## 9. Testing Custom Hooks

**Interview Focus:** Advanced pattern - shows deeper testing knowledge.

Use `renderHook` from RTL:

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

**Why act()?** `act()` ensures all state updates and effects are processed before assertions. Without it, you get warnings and unpredictable behavior.

**Testing hooks with context:**

```tsx
test("uses theme from context", () => {
  const wrapper = ({ children }) => (
    <ThemeContext.Provider value="dark">
      {children}
    </ThemeContext.Provider>
  );
  
  const { result } = renderHook(() => useTheme(), { wrapper });
  
  expect(result.current.theme).toBe("dark");
});
```

---

## 10. Common Patterns & Best Practices

**Interview Focus:** Senior-level expectations - demonstrates testing maturity.

**Arrange, Act, Assert pattern:**

```tsx
// Arrange
render(<Counter />);

// Act
await user.click(screen.getByRole("button"));

// Assert
expect(screen.getByText("Count: 1")).toBeInTheDocument();
```

**Testing forms:**

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

**Anti-patterns to avoid:**

**1. Testing implementation details:**

```tsx
// Bad
expect(component.state.count).toBe(1);

// Good
expect(screen.getByText("Count: 1")).toBeInTheDocument();
```

**2. Using getByTestId everywhere:**

```tsx
// Bad
screen.getByTestId("submit-btn");

// Good
screen.getByRole("button", { name: "Submit" });
```

**3. Not cleaning up mocks:**

```tsx
afterEach(() => {
  jest.clearAllMocks();
});
```

**4. Overusing snapshot tests:** Snapshot tests capture the full rendered output of a component. Every time you change any text, className, or structure — even intentionally — the snapshot breaks and you have to update it. This creates noise and makes developers blindly run `jest --updateSnapshot` without actually reviewing what changed. **Use snapshots sparingly** — only for stable, pure UI components that you explicitly don't want to change accidentally.

**5. Not using screen.debug():** When a test fails and you can't figure out what's in the DOM, use `screen.debug()` to print the current rendered HTML:

```tsx
test("shows user list", async () => {
  render(<UserList />);
  screen.debug(); // prints the current DOM — very useful for debugging
  await screen.findByText("John");
});
```

**Accessibility testing with jest-axe:**

`jest-axe` runs automated accessibility checks against your rendered components. It catches common issues like missing ARIA labels, improper heading hierarchy, and low contrast:

```tsx
import { axe, toHaveNoViolations } from "jest-axe";
expect.extend(toHaveNoViolations);

test("has no accessibility violations", async () => {
  const { container } = render(<LoginForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

This doesn't replace manual accessibility testing, but it catches ~30–40% of common violations automatically.

**Production best practices:**

1. **Test behavior, not implementation** - tests should survive refactoring
2. **Prefer getByRole** - encourages accessibility
3. **Use realistic interactions** - userEvent over fireEvent
4. **Test error states** - don't just test happy paths
5. **Mock at the network boundary** - use MSW for API mocking
6. **Keep tests focused** - one behavior per test
7. **Use descriptive test names** - should explain what's being tested
8. **Test critical paths** - not every detail needs tests

**Common Senior Interview Questions:**

**Q: Difference between unit and integration tests?**
Unit tests isolate a single function or component. Integration tests verify multiple units working together.

**Q: Why use React Testing Library over Enzyme?**
RTL tests user behavior, not implementation. Enzyme tests component internals which makes tests fragile. RTL encourages accessibility.

**Q: What is the difference between getBy, queryBy, findBy?**
`getBy` throws if not found; `queryBy` returns null; `findBy` is async and waits.

**Q: When should you mock?**
Mock external API calls, timers, and modules you don't control. Don't mock internal state, Redux store (use real store), or implementation details.

**Q: What is act() and why is it needed?**
`act()` ensures React processes all state updates and effects before assertions. Required when testing hooks or triggering updates manually.

**Q: What is MSW?**
Mock Service Worker intercepts real network requests in tests. More realistic than mocking fetch directly.

**Q: What is test coverage and is 100% coverage the goal?**
Coverage measures how much code is tested. 100% is not the goal. Goal: critical paths tested, edge cases covered, confidence to refactor. High coverage with bad tests gives false confidence.

---

## Final Takeaway

Good testing focuses on:
- User behavior over implementation
- Critical paths over 100% coverage
- Realistic interactions over shortcuts
- Accessibility over test IDs
- Maintainable tests that survive refactoring

Tests should give you confidence to ship, not slow you down. Test what matters, skip what doesn't, and always prioritize behavior over implementation details.
