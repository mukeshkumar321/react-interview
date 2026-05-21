# React + TypeScript



## 1. Introduction to TypeScript in React

### What is TypeScript?

TypeScript is a superset of JavaScript that adds:

- Static typing
- Better tooling
- Compile-time error checking
- Improved IDE support
- Safer large-scale applications

React + TypeScript is the industry standard for modern frontend applications.



### Why React Developers Use TypeScript

Without TypeScript:

```jsx
function UserCard({ user }) {
  return <h1>{user.name}</h1>;
}
```

Problems:

- Unknown prop structure
- Runtime crashes
- Harder refactoring
- Poor autocomplete

With TypeScript:

```tsx
type User = {
  id: number;
  name: string;
};

type Props = {
  user: User;
};

function UserCard({ user }: Props) {
  return <h1>{user.name}</h1>;
}
```

Benefits:

- Compile-time safety
- Better editor intelligence
- Safer refactoring
- Self-documenting code



### How TypeScript Works in React

TypeScript checks types during development.

Flow:

```text
TSX Code → TypeScript Compiler → JavaScript
```

Type errors are caught before production.



### TSX Files

React TypeScript files use:

```text
.ts   → TypeScript
.tsx  → TypeScript + JSX
```

Example:

```tsx
const element = <h1>Hello</h1>;
```



### Key TypeScript Features Used in React

| Feature | Usage |
|---|---|
| Interfaces | Object shapes |
| Types | Flexible typing |
| Generics | Reusable logic |
| Utility Types | Transforming types |
| Unions | Multiple possible values |
| Enums | Constants |
| Type Inference | Auto-detected types |



### Interview Insight

Senior engineers use TypeScript not just for safety but for:

- Better architecture
- Safer scaling
- Team collaboration
- Refactoring confidence
- API contract enforcement



## 2. Benefits of TypeScript in Large React Applications

### Better Maintainability

Large applications contain:

- Hundreds of components
- Shared APIs
- Complex state
- Multiple developers

TypeScript prevents accidental breakage.



### Safer Refactoring

Without TypeScript:

```jsx
user.fullName
```

If renamed to:

```jsx
user.name
```

Many components may silently break.

TypeScript catches all invalid references instantly.



### Improved Developer Experience

Benefits:

- Autocomplete
- Smart suggestions
- Inline documentation
- Jump-to-definition
- Safer imports



### API Contract Enforcement

Example:

```tsx
type Product = {
  id: number;
  title: string;
  price: number;
};
```

Incorrect API usage gets caught immediately.



### Fewer Runtime Errors

TypeScript prevents:

- Undefined access
- Invalid function arguments
- Wrong prop usage
- Invalid state updates



### Better Team Collaboration

Types act as documentation.

Example:

```tsx
type ButtonProps = {
  variant: "primary" | "secondary";
  disabled?: boolean;
};
```

Developers instantly understand component usage.



### Scalability

TypeScript helps when applications grow because:

- Shared types reduce duplication
- Contracts stay consistent
- Components become predictable



### Interview Insight

TypeScript’s biggest advantage in enterprise React apps is:

> Refactoring confidence at scale.



## 3. Type Inference in React

### What is Type Inference?

TypeScript automatically detects types.

Example:

```tsx
const count = 10;
```

TypeScript infers:

```tsx
number
```



### Inference in React State

```tsx
const [count, setCount] = useState(0);
```

Type inferred:

```tsx
number
```



### Inference in Arrays

```tsx
const users = ["John", "Jane"];
```

Inferred:

```tsx
string[]
```



### Inference in Objects

```tsx
const user = {
  id: 1,
  name: "John",
};
```

Type inferred automatically.



### When Inference Fails

Example:

```tsx
const [user, setUser] = useState(null);
```

Type becomes:

```tsx
null
```

This causes issues later.

Correct:

```tsx
type User = {
  id: number;
  name: string;
};

const [user, setUser] = useState<User | null>(null);
```



### Avoid Over-typing

Bad:

```tsx
const count: number = 0;
```

Good:

```tsx
const count = 0;
```

Let TypeScript infer simple types.



### Interview Insight

Senior engineers:

- Trust inference for simple cases
- Explicitly type complex logic
- Avoid unnecessary annotations



## 4. Typing Functional Components

### Basic Functional Component Typing

```tsx
type Props = {
  title: string;
};

function Header({ title }: Props) {
  return <h1>{title}</h1>;
}
```



### Arrow Function Components

```tsx
type Props = {
  name: string;
};

const UserCard = ({ name }: Props) => {
  return <div>{name}</div>;
};
```



### React.FC

Possible approach:

```tsx
const Header: React.FC<Props> = ({ title }) => {
  return <h1>{title}</h1>;
};
```



### Why Many Teams Avoid React.FC

Problems:

- Automatically includes `children`
- Less explicit
- Can complicate generics

Preferred:

```tsx
function Component(props: Props) {}
```

or

```tsx
const Component = (props: Props) => {};
```



### Return Type Typing

Usually inferred automatically.

Explicit typing:

```tsx
function Header(): JSX.Element {
  return <h1>Hello</h1>;
}
```

Rarely needed.



### Interview Insight

Senior React teams usually avoid `React.FC` for cleaner and more explicit typing.



## 5. Typing Component Props

### Basic Props Typing

```tsx
type ButtonProps = {
  text: string;
  disabled: boolean;
};

function Button({ text, disabled }: ButtonProps) {
  return <button disabled={disabled}>{text}</button>;
}
```



### Optional Props

```tsx
type Props = {
  title: string;
  subtitle?: string;
};
```

`?` means optional.



### Union Types

```tsx
type Props = {
  status: "loading" | "success" | "error";
};
```

Safer than plain strings.



### Function Props

```tsx
type Props = {
  onClick: () => void;
};
```

With arguments:

```tsx
type Props = {
  onSelect: (id: number) => void;
};
```



### Nested Object Props

```tsx
type User = {
  id: number;
  name: string;
};

type Props = {
  user: User;
};
```



### Reusable Shared Types

```tsx
export type User = {
  id: number;
  name: string;
};
```

Used across components.



### Interview Insight

Well-typed props create:

- Predictable components
- Better reusability
- Stronger contracts



## 6. Optional Props and Default Values

### Optional Props

```tsx
type Props = {
  size?: "sm" | "md" | "lg";
};
```



### Default Values with Destructuring

```tsx
function Button({ size = "md" }: Props) {
  return <button>{size}</button>;
}
```

Preferred modern approach.



### Avoid defaultProps in Function Components

Old approach:

```tsx
Button.defaultProps = {
  size: "md",
};
```

Rarely used now.



### Optional Callback Example

```tsx
type Props = {
  onClose?: () => void;
};
```

Usage:

```tsx
onClose?.();
```



### Interview Insight

Prefer:

- Optional props
- Default destructuring
- Optional chaining

For cleaner modern React code.



## 7. Typing Children Props

### Using React.ReactNode

```tsx
type Props = {
  children: React.ReactNode;
};
```

Most common approach.



### Example

```tsx
function Card({ children }: Props) {
  return <div>{children}</div>;
}
```



### What ReactNode Includes

`ReactNode` supports:

- JSX
- Strings
- Numbers
- Fragments
- Arrays
- Null



### Restricting Children

Sometimes specific children are required.

Example:

```tsx
type Props = {
  children: JSX.Element;
};
```



### Multiple Children

```tsx
type Props = {
  children: React.ReactNode[];
};
```

Rarely needed explicitly.



### Interview Insight

`React.ReactNode` is the standard type for flexible children APIs.



## 8. Typing Event Handlers

### Button Click Events

```tsx
const handleClick = (
  e: React.MouseEvent<HTMLButtonElement>
) => {
  console.log(e.currentTarget);
};
```



### Input Change Events

```tsx
const handleChange = (
  e: React.ChangeEvent<HTMLInputElement>
) => {
  console.log(e.target.value);
};
```



### Form Submit Events

```tsx
const handleSubmit = (
  e: React.FormEvent<HTMLFormElement>
) => {
  e.preventDefault();
};
```



### Keyboard Events

```tsx
const handleKeyDown = (
  e: React.KeyboardEvent<HTMLInputElement>
) => {};
```



### Common Interview Question

#### Difference Between `target` and `currentTarget`

| Property | Meaning |
|---|---|
| target | Actual clicked element |
| currentTarget | Element with handler attached |



### Interview Insight

Strong event typing prevents:

- DOM misuse
- Invalid properties
- Runtime bugs



## 9. Typing useState

### Basic State Typing

```tsx
const [count, setCount] = useState<number>(0);
```



### String State

```tsx
const [name, setName] = useState<string>("");
```



### Object State

```tsx
type User = {
  id: number;
  name: string;
};

const [user, setUser] = useState<User | null>(null);
```



### Array State

```tsx
const [users, setUsers] = useState<User[]>([]);
```



### Union State

```tsx
const [status, setStatus] = useState<
  "idle" | "loading" | "success"
>("idle");
```



### Common Mistake

Bad:

```tsx
const [data, setData] = useState([]);
```

Inferred as:

```tsx
never[]
```

Correct:

```tsx
const [data, setData] = useState<User[]>([]);
```



### Interview Insight

Always explicitly type:

- Null states
- Empty arrays
- Complex objects
- Union states



## 10. Typing useRef

### DOM Refs

```tsx
const inputRef = useRef<HTMLInputElement>(null);
```



### Usage

```tsx
inputRef.current?.focus();
```



### Mutable Values

```tsx
const timerRef = useRef<number | null>(null);
```



### Common Mistake

Bad:

```tsx
const ref = useRef(null);
```

Leads to weak typing.



### Ref Safety

Always use optional chaining:

```tsx
ref.current?.scrollIntoView();
```



### Interview Insight

`useRef` is commonly used for:

- DOM access
- Mutable storage
- Previous values
- Timers



## 11. Typing useReducer

### State and Action Types

```tsx
type State = {
  count: number;
};

type Action =
  | { type: "increment" }
  | { type: "decrement" };
```



### Reducer

```tsx
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };

    case "decrement":
      return { count: state.count - 1 };

    default:
      return state;
  }
}
```



### Usage

```tsx
const [state, dispatch] = useReducer(reducer, {
  count: 0,
});
```



### Benefits

- Fully type-safe actions
- Autocomplete for action types
- Impossible invalid dispatches



### Interview Insight

Discriminated unions make reducers extremely safe and scalable.



## 12. Typing useContext

### Creating Context

```tsx
type ThemeContextType = {
  theme: string;
  toggleTheme: () => void;
};
```



### Context Creation

```tsx
const ThemeContext =
  createContext<ThemeContextType | null>(null);
```



### Provider Usage

```tsx
<ThemeContext.Provider value={value}>
  {children}
</ThemeContext.Provider>
```



### Consuming Context

```tsx
const context = useContext(ThemeContext);

if (!context) {
  throw new Error("Context missing");
}
```



### Common Problem

Avoid:

```tsx
createContext({} as ThemeContextType);
```

Unsafe.



### Interview Insight

Null-safe contexts are preferred in production systems.



## 13. Creating Type-safe Context APIs

### Custom Hook Pattern

```tsx
function useTheme() {
  const context = useContext(ThemeContext);

  if (!context) {
    throw new Error("Must be inside provider");
  }

  return context;
}
```



### Benefits

- Removes repetitive null checks
- Safer API
- Cleaner consumers



### Production Pattern

```tsx
export {
  ThemeProvider,
  useTheme,
};
```

Encapsulates internal context implementation.



### Interview Insight

Senior engineers usually expose:

- Provider
- Custom hook

Instead of exporting raw context directly.



## 14. Typing Custom Hooks

### Basic Custom Hook

```tsx
function useCounter(initialValue: number) {
  const [count, setCount] = useState(initialValue);

  return {
    count,
    increment: () => setCount(c => c + 1),
  };
}
```



### Explicit Return Types

Useful for complex hooks.

```tsx
type UseCounterReturn = {
  count: number;
  increment: () => void;
};
```



### Async Hook Example

```tsx
type User = {
  id: number;
  name: string;
};

function useUser(): User | null {
  return null;
}
```



### Interview Insight

Well-typed hooks become reusable application APIs.



## 15. Generic Components

### What Are Generics?

Generics allow reusable typed components.



### Generic List Component

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
};

function List<T>({
  items,
  renderItem,
}: ListProps<T>) {
  return (
    <>
      {items.map(renderItem)}
    </>
  );
}
```



### Usage

```tsx
<List
  items={[1, 2, 3]}
  renderItem={(item) => <div>{item}</div>}
/>
```



### Benefits

- Reusability
- Strong type inference
- Flexible APIs



### Interview Insight

Generics are heavily used in:

- UI libraries
- Form libraries
- Data tables
- Hooks



## 16. Generic Custom Hooks

### Example

```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);

  return data;
}
```



### Usage

```tsx
type User = {
  id: number;
};

const user = useFetch<User>("/api/user");
```



### Benefits

- Reusable fetching logic
- Strong API contracts
- Flexible hook architecture



### Interview Insight

Generics are essential for scalable shared hooks.



## 17. Utility Types in React

### Partial

```tsx
type User = {
  id: number;
  name: string;
};

type PartialUser = Partial<User>;
```

All properties become optional.



### Pick

```tsx
type UserPreview = Pick<User, "id" | "name">;
```



### Omit

```tsx
type UserWithoutId = Omit<User, "id">;
```



### Record

```tsx
type Users = Record<string, User>;
```



### Required

```tsx
type CompleteUser = Required<User>;
```



### Interview Insight

Utility types reduce duplication significantly in enterprise applications.



## 18. Discriminated Unions

### What Are They?

A safe way to model multiple states.



### Example

```tsx
type State =
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; error: string };
```



### Usage

```tsx
if (state.status === "success") {
  console.log(state.data);
}
```

TypeScript automatically narrows the type.



### Benefits

- Impossible invalid states
- Safer UI rendering
- Better reducer patterns



### Interview Insight

Discriminated unions are a major senior-level TypeScript concept.



## 19. Type-safe Reducer Patterns

### Strongly Typed Actions

```tsx
type Action =
  | { type: "ADD"; payload: string }
  | { type: "REMOVE"; payload: number };
```



### Exhaustive Checking

```tsx
default:
  const exhaustiveCheck: never = action;
  return exhaustiveCheck;
```

Ensures all action types are handled.



### Benefits

- Safer reducers
- Easier maintenance
- Better autocomplete



### Interview Insight

Exhaustive reducer checking is a strong senior-level practice.



## 20. API Response Typing

### API Model Example

```tsx
type ApiResponse<T> = {
  data: T;
  success: boolean;
};
```



### Usage

```tsx
type User = {
  id: number;
};

const response: ApiResponse<User> = data;
```



### Error Handling

```tsx
type ApiError = {
  message: string;
  code: number;
};
```



### Production Insight

Never trust API responses blindly.

Validate:

- Nullable fields
- Missing properties
- Backend inconsistencies



## 21. Form Typing Strategies

### Controlled Inputs

```tsx
const [email, setEmail] = useState("");
```



### Form Event Typing

```tsx
const handleSubmit = (
  e: React.FormEvent<HTMLFormElement>
) => {};
```



### Dynamic Form State

```tsx
type FormValues = {
  email: string;
  password: string;
};
```



### Form Libraries

Common enterprise libraries:

- React Hook Form
- Formik
- Zod validation



### Interview Insight

Type-safe forms prevent many production bugs.



## 22. Typing Async Functions

### Async Function Typing

```tsx
async function fetchUsers(): Promise<User[]> {
  return [];
}
```



### Async Event Handlers

```tsx
const handleSubmit = async (): Promise<void> => {};
```



### Error Typing

```tsx
try {
} catch (error: unknown) {
}
```

Avoid:

```tsx
catch (error: any)
```



### Interview Insight

Use `unknown` instead of `any` for safer error handling.



## 23. Typing Forward Refs

### Basic Example

```tsx
type Props = {
  label: string;
};

const Input = forwardRef<
  HTMLInputElement,
  Props
>(({ label }, ref) => {
  return <input ref={ref} />;
});
```



### Why It Matters

Important for:

- UI libraries
- Accessibility
- Focus management



### Interview Insight

Forward refs are heavily used in reusable component systems.



## 24. Polymorphic Component Typing

### What is a Polymorphic Component?

A component that can render different HTML elements.

Example:

```tsx
<Button as="a" href="/home" />
```



### Basic Pattern

```tsx
type Props<T extends React.ElementType> = {
  as?: T;
};
```



### Why It’s Advanced

Requires:

- Generics
- Prop merging
- Element inference



### Production Usage

Used heavily in:

- Design systems
- Component libraries
- Headless UI frameworks



## 25. Component Library Typing Patterns

### Common Patterns

#### Variant Props

```tsx
type Variant =
  | "primary"
  | "secondary";
```



#### Shared Base Props

```tsx
type BaseProps = {
  className?: string;
};
```



#### Extending Native HTML Props

```tsx
type ButtonProps =
  React.ButtonHTMLAttributes<HTMLButtonElement>;
```



### Interview Insight

Component library typing is a common senior frontend interview topic.



## 26. Strict TypeScript Configuration

### Important Strict Options

```json
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true
}
```



### Why Strict Mode Matters

Prevents:

- Unsafe null access
- Weak typing
- Hidden runtime bugs



### Production Insight

Enterprise applications should always use strict mode.



## 27. Common React TypeScript Errors

### Property Does Not Exist

```tsx
Property 'name' does not exist
```

Cause:

- Wrong object shape
- Missing types



### Type 'null' is not assignable

Common with refs and state.

Fix:

```tsx
User | null
```



### Never[] Problems

```tsx
useState([])
```

Fix with explicit typing.



### Event Type Errors

Incorrect:

```tsx
e.target.value
```

Without proper event typing.



## 28. Avoiding any and Unsafe Types

### Why `any` is Dangerous

`any` disables TypeScript safety completely.



### Bad Example

```tsx
const data: any = response;
```



### Better Alternatives

#### unknown

```tsx
const data: unknown = response;
```

#### Generics

```tsx
function identity<T>(value: T): T {
  return value;
}
```



### Interview Insight

Using excessive `any` usually signals weak TypeScript architecture.



## 29. Refactoring JavaScript React Apps to TypeScript

### Recommended Migration Strategy

#### Step 1

Rename:

```text
.js → .tsx
```



#### Step 2

Enable loose TypeScript initially.



#### Step 3

Type components gradually.

Start with:

- Props
- API models
- Shared utilities



#### Step 4

Enable stricter rules incrementally.



### Production Insight

Large migrations should be incremental, not full rewrites.



## 30. Scalable Type Organization

### Common Structure

```text
types/
├── api/
├── components/
├── forms/
├── shared/
```



### Shared Domain Models

```tsx
types/user.ts
types/product.ts
```



### Avoid Type Duplication

Centralize shared contracts.



### Production Insight

Poor type organization creates massive maintenance issues in enterprise apps.



## 31. TypeScript Performance Considerations

### Type Complexity Can Slow Builds

Heavy usage of:

- Deep generics
- Massive unions
- Recursive types

Can impact IDE performance.



### Optimize Shared Types

Avoid extremely complex inferred types.



### Prefer Simpler Abstractions

Over-engineered types reduce readability.



### Production Insight

Type safety should improve developer productivity, not harm it.



## 32. React TypeScript Anti-patterns

### Overusing React.FC

Often unnecessary.



### Excessive Generics

Bad:

```tsx
<T extends unknown>
```

Without need.



### Using any Everywhere

Removes TypeScript benefits entirely.



### Massive Shared Types

Huge global types become hard to maintain.



### Type Assertions Abuse

Bad:

```tsx
data as User
```

Without validation.



### Interview Insight

Senior engineers prioritize maintainability over “clever” typing.



## 33. Production-level TypeScript Best Practices

### Prefer Type Inference

Avoid unnecessary annotations.



### Type API Boundaries Carefully

Critical areas:

- APIs
- Forms
- Shared hooks
- Context
- State



### Use Strict Mode

Always.



### Keep Types Close to Features

Avoid giant global type files.



### Use Shared Domain Models

Example:

```text
User
Product
Order
```

Shared across app layers.



### Validate External Data

TypeScript does not validate runtime data automatically.

Use:

- Zod
- Yup
- Runtime validation



### Interview Insight

Production TypeScript is about:

- Scalability
- Predictability
- Refactoring safety
- Developer productivity



## 34. Common Senior-level Interview Questions

### Conceptual Questions

#### Why use TypeScript with React?

#### Difference between `type` and `interface`?

#### Why avoid excessive `any`?

#### What are discriminated unions?

#### What are generics?



### Practical Questions

#### Type a reusable table component

#### Build a typed reducer

#### Type a custom hook

#### Type a context API

#### Type API responses safely



### Architecture Questions

#### How do you organize types in large applications?

#### How do you migrate JS apps to TS?

#### How do you type component libraries?

#### How do you enforce API contracts?



### Senior-level Expectations

Interviewers expect:

- Strong practical typing knowledge
- Scalable architecture thinking
- Clean abstraction design
- Type-safe API handling
- Real production experience