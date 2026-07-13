# React Interview Questions (3–5 Years Experience)

These questions cover React fundamentals, internals, hooks, performance optimization, state management, TypeScript, Next.js, and machine coding discussions commonly asked in frontend interviews.

---

# React Internals & Rendering

## Fundamental

1. What happens when a React component re-renders?
2. What is the Virtual DOM?
3. Why is React faster than direct DOM manipulation?
4. How does React know which DOM nodes changed?
5. What is Reconciliation?
6. Explain React's Diffing Algorithm.
7. What assumptions does React make during reconciliation?
8. Why should list items have unique keys?
9. What happens if keys are unstable?
10. Why should we avoid array indexes as keys?

## Advanced

11. Explain React Fiber Architecture.
12. Why was Fiber introduced?
13. What is the difference between Stack Reconciler and Fiber Reconciler?
14. Explain Render Phase vs Commit Phase.
15. Which phase can be interrupted?
16. Which phase is synchronous?
17. What is Concurrent Rendering?
18. What is Time Slicing?
19. How does React prioritize updates?
20. What is Batching?

## Senior Level

21. Why does React sometimes re-render child components unnecessarily?
22. How would you debug excessive re-renders?
23. What causes React to discard and recreate a component?
24. How does React preserve state between renders?
25. What actually happens when `setState()` is called?

---

# Strict Mode

26. What does `React.StrictMode` do?
27. Why does React render components twice in development with Strict Mode?
28. What issues does Strict Mode help detect?
29. Does Strict Mode affect production builds?

---

# Hooks Deep Dive

## useState

30. Why doesn't `useState` update immediately?
31. Why can multiple state updates be batched?
32. What is a Functional State Update?
33. When should you use Functional Updates?
34. How does React store Hook state internally?

## useEffect

35. Why does `useEffect` exist?
36. Difference between `useEffect` and `useLayoutEffect`?
37. When does `useEffect` run?
38. When does Cleanup execute?
39. Why do we return a Cleanup Function?
40. What problems can occur without Cleanup?
41. Why should async functions not be passed directly to `useEffect`?

## Dependency Array

42. How does React compare dependencies?
43. Why does `useEffect` sometimes run unexpectedly?
44. What happens if the dependency array is omitted?
45. What happens if the dependency array is empty?
46. Why does React recommend the `exhaustive-deps` ESLint rule?

## Stale Closures

47. What is a Stale Closure?
48. Why do Stale Closures happen?
49. How do you fix Stale Closures?
50. Why do intervals commonly suffer from Stale Closures?
51. When should `useRef` be used to solve Stale Closure issues?

## useRef

52. Why does `useRef` not trigger re-renders?
53. How is `useRef` different from State?
54. How does React keep Ref values persistent?
55. When should State be preferred over Ref?
56. Explain the implementation of `usePrevious`.

## useTransition & useDeferredValue

57. What is `useTransition`?
58. What is `useDeferredValue`?
59. Difference between `useTransition` and `useDeferredValue`?
60. When would you use `useTransition` over `useMemo`?
61. What is `isPending` in `useTransition` and how do you use it?

## forwardRef & useImperativeHandle

62. What is `forwardRef` and why is it needed?
63. Why can't you pass `ref` as a regular prop to a component?
64. What is `useImperativeHandle`?
65. When should `useImperativeHandle` be used?
66. How do you type `forwardRef` with TypeScript?

---

# Custom Hooks

## Design

67. What is a Custom Hook?
68. Why create Custom Hooks?
69. What rules must Custom Hooks follow?
70. What should Custom Hooks return?
71. When should a Hook return an Object vs an Array?

## Implementations

72. Implement `useToggle`.
73. Implement `usePrevious`.
74. Implement `useDebounce`.
75. Implement `useThrottle`.
76. Implement `useLocalStorage`.
77. Implement `useTimeout`.
78. Implement `useInterval`.
79. Implement `useClickOutside`.
80. Implement `usePagination`.
81. Implement `useFetch`.

## Follow-up Questions

82. How would you make `useFetch` production-ready?
83. How would you cancel previous API requests?
84. How would you prevent Race Conditions?

---

# Component Patterns

## Higher Order Components (HOC)

85. What is a Higher Order Component (HOC)?
86. When should you use an HOC vs a Custom Hook?
87. What are the downsides of HOCs?
88. Give an example of a common HOC pattern (e.g., `withAuth`).

## Render Props

89. What is the Render Props pattern?
90. Why was Render Props popular before Hooks?
91. When would you still use Render Props today?

## Compound Components

92. What is the Compound Component pattern?
93. How does Context enable Compound Components?
94. Design a Compound Component (e.g., Accordion or Tabs).

---

# Error Boundaries & Portals

## Error Boundaries

95. What is an Error Boundary?
96. Why must Error Boundaries be Class Components?
97. What lifecycle methods do Error Boundaries use?
98. How would you handle async errors that Error Boundaries can't catch?

## Portals

99. What is a React Portal?
100. When would you use a Portal?
101. Does an event inside a Portal bubble through the React tree or the DOM tree?

---

# Performance Optimization

102. What causes a React component to re-render?
103. What does `React.memo` do?
104. When should `React.memo` be avoided?
105. Difference between `React.memo` and `useMemo`?
106. Difference between `useMemo` and `useCallback`?
107. When does `useMemo` become harmful?
108. How do you identify Performance Bottlenecks?

## Practical Scenarios

109. A Parent renders 1000 Child Components. How would you optimize it?
110. Why is inline object creation problematic?
111. Why are inline functions problematic?
112. How would you optimize a large Table?
113. What is Virtualization?
114. When would you use Windowing?

---

# Code Splitting & Lazy Loading

115. What is Code Splitting and why does it matter?
116. How does `React.lazy` work?
117. What is `Suspense` and how does it work with lazy loading?
118. What is a dynamic `import()` and how does it differ from a static import?
119. How would you lazy load a route in React Router?
120. What happens if a lazy-loaded component fails to load?
121. How do you prefetch a lazy component before the user navigates to it?

---

# State Management

## Context API

122. How does Context API work internally?
123. Why can Context cause performance issues?
124. How would you optimize Context?
125. What is Context Splitting?

## useReducer

126. When should you use `useReducer` instead of `useState`?
127. What is the difference between `useReducer` and Redux?
128. How do you handle async operations with `useReducer`?
129. How would you share a reducer's dispatch across components without Redux?
130. What is the `init` (initializer) argument in `useReducer`?

## Redux

131. Why Redux?
132. Explain Redux Data Flow.
133. What is a Reducer?
134. Why must Reducers be Pure Functions?
135. What is Middleware?
136. What problems does Redux Toolkit solve?
137. How does Immer work?
138. Why can we mutate state inside RTK Reducers?

## RTK Query

139. What problems does RTK Query solve?
140. How does RTK Query Caching work?
141. What are Tags?
142. How does Cache Invalidation work?
143. Difference between Query and Mutation?

## React Query (TanStack Query)

144. What problems does React Query solve?
145. How does React Query caching differ from RTK Query?
146. What is `staleTime` vs `gcTime` (formerly `cacheTime`) in React Query?
147. How does React Query handle background refetching?
148. When would you choose React Query over RTK Query?

---

# React Event System

149. What is a Synthetic Event?
150. Why did React create Synthetic Events?
151. What is Event Delegation?
152. How does React implement Event Delegation?
153. Difference between Native and Synthetic Events?
154. What is Event Bubbling?
155. What is Event Capturing?
156. Difference between `stopPropagation()` and `preventDefault()`?

---

# Async React & Data Fetching

157. How do you fetch data in React?
158. How do you handle Loading States?
159. How do you handle Error States?
160. What is a Race Condition?
161. How does `AbortController` work?
162. Why should requests be cancelled?
163. How would you implement Debounced Search?
164. Infinite Scroll vs Pagination?

---

# Forms

165. What is a Controlled Component?
166. What is an Uncontrolled Component?
167. When would you choose Uncontrolled over Controlled?
168. What problems do libraries like React Hook Form solve?
169. How does React Hook Form avoid unnecessary re-renders?
170. What is `register`, `watch`, and `handleSubmit` in React Hook Form?
171. How do you handle validation in React Hook Form?
172. How do you handle a dynamic field array (e.g., add/remove rows)?

---

# React + TypeScript

173. How do you type Component Props?
174. How do you type `useState`?
175. How do you type `useRef`?
176. How do you type `useReducer`?
177. What are Generics?
178. Why are Generics useful in Custom Hooks?
179. Type a generic `useFetch<T>()` Hook.
180. Type a generic `useLocalStorage<T>()` Hook.

---

# Next.js

181. Difference between CSR, SSR, SSG, and ISR?
182. When would you use SSR?
183. When would you use SSG?
184. What is Hydration?
185. What is a Hydration Mismatch?
186. Server Components vs Client Components?
187. Why are Server Components faster?
188. What can and cannot be used inside Server Components?
189. Explain the App Router Architecture.

---

# Accessibility (a11y)

190. How do you make a custom button component accessible?
191. What is the difference between `aria-label`, `aria-labelledby`, and `aria-describedby`?
192. How do you manage focus after opening a Modal?
193. How do you trap focus inside a Modal?
194. What is the `role` attribute and when should you use it?
195. How do you support keyboard navigation in a Dropdown?

---

# Styling Approaches

196. What are the tradeoffs between CSS Modules, CSS-in-JS, and Tailwind CSS?
197. Why can CSS-in-JS (e.g., styled-components) hurt performance?
198. How does styled-components handle dynamic styles?
199. What is the difference between a global style and a scoped style in React?
200. How would you implement a dark mode toggle in React?

---

# Testing

201. What is React Testing Library and why is it preferred over Enzyme?
202. Difference between `getBy`, `queryBy`, and `findBy`?
203. How do you test a component that fetches data?
204. How do you mock a Custom Hook in tests?
205. What is `userEvent` and how does it differ from `fireEvent`?
206. How do you test async state updates?
207. What is the `act()` wrapper and when is it needed?

---

# Machine Coding Discussion Questions

208. Design a reusable Modal Component.
209. Design a reusable Table Component.
210. Design an OTP Input Component.
211. Design an Infinite Scroll Component.
212. Design a Multi-Step Form.
213. Design a Debounced Search / Autocomplete Component.
214. Design a File Explorer.
215. Design a Kanban Board.
216. Design a Virtualized List.
217. Design an Accordion Component.
218. Design a Toast / Notification System.
219. Design a Tabs Component using the Compound Component pattern.

---

# Senior React Questions (Most Asked)

220. Explain the complete React Render Lifecycle.
221. How does React internally store Hooks?
222. Why do Hooks rely on Call Order?
223. How does React know which Hook belongs to which Component?
224. Explain Stale Closures with a real-world example.
225. Explain React Fiber as if teaching a Junior Developer.
226. A page is slow. Walk me through your debugging process.
227. When would you choose Context over Redux?
228. When would `React.memo` not work?
229. Explain Reconciliation using Keys.
230. What happens internally from `setState()` to DOM update?

---

# Interview Readiness Checklist

If you can confidently answer:

- ✅ 90+ Questions → Good for Mid-Level React Interviews
- ✅ 150+ Questions → Strong for 4–5 Years Experience
- ✅ 200+ Questions → Senior Frontend Interview Ready
- ✅ 230+ Questions → React Internals & Architecture Level Understanding
