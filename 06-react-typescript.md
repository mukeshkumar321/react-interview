# React + TypeScript

## Topics Covered

- [1. Why TypeScript in React](#1-why-typescript-in-react)
- [2. Typing Components & Props](#2-typing-components--props)
- [3. Typing Children](#3-typing-children)
- [4. Typing Events](#4-typing-events)
- [5. Typing useState](#5-typing-usestate)
- [6. Typing useRef](#6-typing-useref)
- [7. Typing useReducer](#7-typing-usereducer)
- [8. Typing useContext](#8-typing-usecontext)
- [9. Typing Custom Hooks](#9-typing-custom-hooks)
- [10. Generic Components](#10-generic-components)
- [11. Common Patterns](#11-common-patterns)
- [12. Production Best Practices](#12-production-best-practices)

---

TypeScript is a superset of JavaScript that adds static typing, better tooling, compile-time error checking, improved IDE support, and safer large-scale applications. React + TypeScript is the industry standard for modern frontend applications.

---

## 1. Why TypeScript in React

**Interview Focus:** Understanding the "why" is crucial - commonly asked in architectural discussions.

Without TypeScript, you get unknown prop structures, runtime crashes, harder refactoring, and poor autocomplete. With TypeScript, you get compile-time safety, better editor intelligence, safer refactoring, and self-documenting code.

**Benefits in large applications:**

| Benefit | Impact |
|---|---|
| **Safer refactoring** | Catch all breaking changes instantly |
| **Better DX** | Autocomplete, smart suggestions, inline docs |
| **API contracts** | Components become self-documenting |
| **Fewer bugs** | Catch errors before production |
| **Team collaboration** | Clear interfaces reduce confusion |

Senior engineers use TypeScript not just for safety but for better architecture, scaling confidence, and team productivity.

---

## 2. Typing Components & Props

**Interview Focus:** Most fundamental TypeScript pattern in React - almost always tested.

Basic functional component:

```tsx
type Props = {
  title: string;
};

function Header({ title }: Props) {
  return <h1>{title}</h1>;
}
```

**Optional props:**

```tsx
type Props = {
  title: string;
  subtitle?: string;
};
```

**Union types** (safer than plain strings):

```tsx
type Props = {
  status: "loading" | "success" | "error";
};
```

**Function props:**

```tsx
type Props = {
  onClick: () => void;
  onSelect: (id: number) => void;
};
```

**Optional props with defaults:**

```tsx
type Props = {
  size?: "sm" | "md" | "lg";
};

function Button({ size = "md" }: Props) {
  return <button>{size}</button>;
}
```

**Why avoid React.FC:** Many teams avoid `React.FC` because it automatically includes `children`, is less explicit, and can complicate generics. Prefer direct function typing.

---

## 3. Typing Children

**Interview Focus:** Common practical question - understanding React.ReactNode is important.

Use `React.ReactNode` for flexible children APIs:

```tsx
type Props = {
  children: React.ReactNode;
};

function Card({ children }: Props) {
  return <div>{children}</div>;
}
```

`ReactNode` supports JSX, strings, numbers, fragments, arrays, and null.

For more restrictive children, use `JSX.Element`:

```tsx
type Props = {
  children: JSX.Element;
};
```

This only accepts a single React element, not strings or numbers.

---

## 4. Typing Events

**Interview Focus:** Frequently needed in practical exercises - essential for forms.

**Button click:**

```tsx
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log(e.currentTarget);
};
```

**Input change:**

```tsx
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};
```

**Form submit:**

```tsx
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};
```

**Important difference:** `target` is the actual clicked element; `currentTarget` is the element with the handler attached.

Strong event typing prevents DOM misuse, invalid properties, and runtime bugs.

---

## 5. Typing useState

**Interview Focus:** Very common in coding exercises - especially with complex state.

**Basic state:**

```tsx
const [count, setCount] = useState<number>(0);
```

**Object state:**

```tsx
type User = {
  id: number;
  name: string;
};

const [user, setUser] = useState<User | null>(null);
```

**Array state:**

```tsx
const [users, setUsers] = useState<User[]>([]);
```

**Union state:**

```tsx
const [status, setStatus] = useState<"idle" | "loading" | "success">("idle");
```

**Common mistake:** `const [data, setData] = useState([]);` infers `never[]` - useless type.

Always explicitly type: null states, empty arrays, complex objects, and union states.

---

## 6. Typing useRef

**Interview Focus:** Understanding ref types is important for DOM manipulation.

**DOM refs:**

```tsx
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();
```

**Mutable values:**

```tsx
const timerRef = useRef<number | null>(null);
```

Always use optional chaining (`ref.current?.`) for ref safety since refs can be null.

Common uses: DOM access, mutable storage, previous values, and timers.

---

## 7. Typing useReducer

**Interview Focus:** Senior-level pattern - discriminated unions are powerful.

**State and action types:**

```tsx
type State = {
  count: number;
};

type Action =
  | { type: "increment" }
  | { type: "decrement" }
  | { type: "set"; payload: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "set":
      return { count: action.payload };
    default:
      return state;
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0 });
```

Discriminated unions make reducers extremely safe - fully type-safe actions, autocomplete for action types, and impossible invalid dispatches.

---

## 8. Typing useContext

**Interview Focus:** Type-safe context is essential for production apps.

**Creating context:**

```tsx
type ThemeContextType = {
  theme: string;
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType | null>(null);

<ThemeContext.Provider value={value}>
  {children}
</ThemeContext.Provider>
```

**Consuming context:**

```tsx
const context = useContext(ThemeContext);

if (!context) {
  throw new Error("Must be used inside provider");
}
```

**Custom hook pattern (safer):**

```tsx
function useTheme() {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error("Must be inside provider");
  }
  
  return context;
}
```

This removes repetitive null checks and creates a safer API.

Avoid `createContext({} as ThemeContextType)` - it's unsafe. Null-safe contexts are preferred in production.

---

## 9. Typing Custom Hooks

**Interview Focus:** Essential skill for reusable logic - commonly tested.

**Basic custom hook:**

```tsx
function useCounter(initialValue: number) {
  const [count, setCount] = useState(initialValue);
  
  return {
    count,
    increment: () => setCount(c => c + 1),
    decrement: () => setCount(c => c - 1),
  };
}
```

**Generic hooks:**

```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then((data: T) => setData(data))
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading };
}

type User = { id: number; name: string };
const { data } = useFetch<User>("/api/user");
```

Generics are essential for scalable shared hooks, enabling reusable fetching logic and strong API contracts.

---

## 10. Generic Components

**Interview Focus:** Advanced pattern - shows deep TypeScript understanding.

**What are generics?** Think of generics like function parameters, but for types. Instead of hardcoding a type, you let the caller decide what type to use. The angle-bracket `<T>` is a type placeholder — `T` can be any type, and TypeScript fills it in based on how you call the function/component.

```ts
// Regular function — works only with numbers
function first(arr: number[]) { return arr[0]; }

// Generic function — works with any type
function first<T>(arr: T[]): T { return arr[0]; }

first([1, 2, 3]);         // T = number
first(["a", "b"]);        // T = string
```

Generics allow reusable typed components:

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
};

function List<T>({ items, renderItem }: ListProps<T>) {
  return <>{items.map(renderItem)}</>;
}

<List items={[1, 2, 3]} renderItem={(item) => <div>{item}</div>} />
```

**Benefits:** reusability, strong type inference, and flexible APIs. Generics are heavily used in UI libraries, form libraries, data tables, and hooks.

---

## 11. Common Patterns

**Interview Focus:** Practical patterns that appear frequently in production code.

**Utility Types:**

```tsx
type User = {
  id: number;
  name: string;
  email: string;
};

// Make all properties optional
type PartialUser = Partial<User>;

// Pick specific properties
type UserPreview = Pick<User, "id" | "name">;

// Exclude specific properties
type UserWithoutId = Omit<User, "id">;

// Create object types
type Users = Record<string, User>;
```

**ComponentProps — extend native HTML elements:**

```tsx
import { ComponentProps } from "react";

// Get all props that a native <button> accepts
type ButtonProps = ComponentProps<"button"> & {
  variant: "primary" | "secondary";
};

function Button({ variant, ...props }: ButtonProps) {
  return <button className={`btn-${variant}`} {...props} />;
}

// Now Button accepts all standard HTML button props + variant
<Button variant="primary" onClick={fn} disabled={true} type="submit" />
```

This is cleaner than manually writing `React.ButtonHTMLAttributes<HTMLButtonElement>`. Use `ComponentProps<typeof SomeComponent>` to extend your own components too.

**Discriminated Unions (critical for state machines):**

A "discriminated union" is a union type where each variant has a unique property that TypeScript can use to tell them apart. When you check that property, TypeScript automatically knows which variant you have and what other properties exist:

```tsx
type State =
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; error: string };

if (state.status === "success") {
  console.log(state.data); // TypeScript knows data exists here
}
if (state.status === "error") {
  console.log(state.error); // TypeScript knows error exists here
}
// state.data outside of success check → TypeScript error
```

Benefits: impossible invalid states, safer UI rendering, and better reducer patterns.

**Forward Refs:**

```tsx
type Props = {
  label: string;
};

const Input = forwardRef<HTMLInputElement, Props>(({ label }, ref) => {
  return <input ref={ref} />;
});
```

Forward refs are heavily used in reusable component systems for UI libraries, accessibility, and focus management.

**Extending Native Props:**

```tsx
type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  variant: "primary" | "secondary";
};

function Button({ variant, ...props }: ButtonProps) {
  return <button className={`btn-${variant}`} {...props} />;
}
```

This pattern allows your components to accept all standard HTML attributes.

---

## 12. Production Best Practices

**Interview Focus:** Senior-level expectations - demonstrating production experience.

**1. Prefer type inference:**

```tsx
// Unnecessary
const count: number = 0;

// Better - TypeScript infers
const count = 0;
```

**2. Type API boundaries carefully:**

Critical areas: APIs, forms, shared hooks, context, and state. These are where type safety matters most.

**3. Use strict mode:**

```json
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true
}
```

Strict mode prevents unsafe null access, weak typing, and hidden runtime bugs. Enterprise applications should always use strict mode.

**4. Keep types close to features:**

```txt
features/
  auth/
    types.ts
    components/
    hooks/
```

Avoid giant global type files. Colocate types with the features that use them.

**5. Use shared domain models:**

```txt
types/
  user.ts
  product.ts
  order.ts
```

Shared models like User, Product, Order should be centralized and reused across layers.

**6. Validate external data:**

TypeScript does NOT validate runtime data. Use Zod, Yup, or runtime validation for API responses:

```tsx
const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
});

const user = UserSchema.parse(apiResponse);
```

**7. Avoid excessive `any`:**

`any` disables TypeScript safety completely. Use `unknown` when you don't know the type:

```tsx
// Bad
const data: any = response;

// Better
const data: unknown = response;

if (typeof data === "object" && data !== null) {
  // Type guard
}
```

**8. Organize types scalably:**

```txt
types/
  api/
  components/
  forms/
  shared/
```

Poor type organization creates massive maintenance issues in enterprise apps.

**Common Senior Interview Questions:**

**Q: Why use TypeScript with React?**
Type safety, better tooling, safer refactoring, self-documenting code, and team collaboration.

**Q: Difference between `type` and `interface`?**
Mostly interchangeable. `type` is more flexible (unions, primitives); `interface` is better for extending. Modern preference: use `type` for most cases.

**Q: Why avoid excessive `any`?**
`any` removes all type safety. Use `unknown` or proper types instead.

**Q: What are discriminated unions?**
A union type where each variant has a unique literal property (like `status`). TypeScript can narrow the type based on that property.

**Q: What are generics?**
Generics allow components/functions to work with multiple types while maintaining type safety. Like function parameters, but for types.

**Q: How do you type API responses?**
Define the expected shape as a type, then validate the runtime data with a library like Zod. TypeScript only checks at compile time, not runtime.

**Q: When should you use `unknown` vs `any`?**
Always prefer `unknown` when you don't know the type. `unknown` forces you to check the type before using it; `any` allows anything unsafely.

---

## Final Takeaway

Production TypeScript is about: scalability, predictability, refactoring safety, and developer productivity. Senior engineers focus on practical typing (not over-engineering), safe API boundaries, shared domain models, and maintainable architecture.

TypeScript should improve developer productivity, not harm it. Keep types simple, focus on high-value areas, and avoid over-abstraction.
