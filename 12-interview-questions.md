# 🎯 React Interview Questions (3–5 Years Experience)

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

# Hooks Deep Dive

## useState

26. Why doesn't `useState` update immediately?
27. Why can multiple state updates be batched?
28. What is a Functional State Update?
29. When should you use Functional Updates?
30. How does React store Hook state internally?

## useEffect

31. Why does `useEffect` exist?
32. Difference between `useEffect` and `useLayoutEffect`?
33. When does `useEffect` run?
34. When does Cleanup execute?
35. Why do we return a Cleanup Function?
36. What problems can occur without Cleanup?
37. Why should async functions not be passed directly to `useEffect`?

## Dependency Array

38. How does React compare dependencies?
39. Why does `useEffect` sometimes run unexpectedly?
40. What happens if the dependency array is omitted?
41. What happens if the dependency array is empty?
42. Why does React recommend the `exhaustive-deps` ESLint rule?

## Stale Closures

43. What is a Stale Closure?
44. Why do Stale Closures happen?
45. How do you fix Stale Closures?
46. Why do intervals commonly suffer from Stale Closures?
47. When should `useRef` be used to solve Stale Closure issues?

## useRef

48. Why does `useRef` not trigger re-renders?
49. How is `useRef` different from State?
50. How does React keep Ref values persistent?
51. When should State be preferred over Ref?
52. Explain the implementation of `usePrevious`.

---

# Custom Hooks

## Design

53. What is a Custom Hook?
54. Why create Custom Hooks?
55. What rules must Custom Hooks follow?
56. What should Custom Hooks return?
57. When should a Hook return an Object vs an Array?

## Implementations

58. Implement `useToggle`.
59. Implement `usePrevious`.
60. Implement `useDebounce`.
61. Implement `useThrottle`.
62. Implement `useLocalStorage`.
63. Implement `useTimeout`.
64. Implement `useInterval`.
65. Implement `useClickOutside`.
66. Implement `usePagination`.
67. Implement `useFetch`.

## Follow-up Questions

68. How would you make `useFetch` production-ready?
69. How would you cancel previous API requests?
70. How would you prevent Race Conditions?

---

# Performance Optimization

71. Why does React re-render?
72. What triggers a Component Re-render?
73. What does `React.memo` do?
74. When should `React.memo` be avoided?
75. Difference between `React.memo` and `useMemo`?
76. Difference between `useMemo` and `useCallback`?
77. When does `useMemo` become harmful?
78. How do you identify Performance Bottlenecks?

## Practical Scenarios

79. A Parent renders 1000 Child Components. How would you optimize it?
80. Why is inline object creation problematic?
81. Why are inline functions problematic?
82. How would you optimize a large Table?
83. What is Virtualization?
84. When would you use Windowing?

---

# State Management

## Context API

85. How does Context API work internally?
86. Why can Context cause performance issues?
87. How would you optimize Context?
88. What is Context Splitting?

## Redux

89. Why Redux?
90. Explain Redux Data Flow.
91. What is a Reducer?
92. Why must Reducers be Pure Functions?
93. What is Middleware?
94. What problems does Redux Toolkit solve?
95. How does Immer work?
96. Why can we mutate state inside RTK Reducers?

## RTK Query

97. What problems does RTK Query solve?
98. How does RTK Query Caching work?
99. What are Tags?
100. How does Cache Invalidation work?
101. Difference between Query and Mutation?

---

# React Event System

102. What is a Synthetic Event?
103. Why did React create Synthetic Events?
104. What is Event Delegation?
105. How does React implement Event Delegation?
106. Difference between Native and Synthetic Events?
107. What is Event Bubbling?
108. What is Event Capturing?
109. Difference between `stopPropagation()` and `preventDefault()`?

---

# Async React & Data Fetching

110. How do you fetch data in React?
111. How do you handle Loading States?
112. How do you handle Error States?
113. What is a Race Condition?
114. How does `AbortController` work?
115. Why should requests be cancelled?
116. How would you implement Debounced Search?
117. Infinite Scroll vs Pagination?

---

# React + TypeScript

118. How do you type Component Props?
119. How do you type `useState`?
120. How do you type `useRef`?
121. How do you type `useReducer`?
122. What are Generics?
123. Why are Generics useful in Custom Hooks?
124. Type a generic `useFetch<T>()` Hook.
125. Type a generic `useLocalStorage<T>()` Hook.

---

# Next.js

126. Difference between CSR, SSR, SSG, and ISR?
127. When would you use SSR?
128. When would you use SSG?
129. What is Hydration?
130. What is a Hydration Mismatch?
131. Server Components vs Client Components?
132. Why are Server Components faster?
133. What can and cannot be used inside Server Components?
134. Explain the App Router Architecture.

---

# Machine Coding Discussion Questions

135. Design a reusable Modal Component.
136. Design a reusable Table Component.
137. Design an OTP Input Component.
138. Design an Infinite Scroll Component.
139. Design a Multi-Step Form.
140. Design a Debounced Search Component.
141. Design a File Explorer.
142. Design a Kanban Board.
143. Design a Virtualized List.

---

# Senior React Questions (Most Asked)

144. Explain the complete React Render Lifecycle.
145. How does React internally store Hooks?
146. Why do Hooks rely on Call Order?
147. How does React know which Hook belongs to which Component?
148. Explain Stale Closures with a real-world example.
149. Explain React Fiber as if teaching a Junior Developer.
150. A page is slow. Walk me through your debugging process.
151. When would you choose Context over Redux?
152. When would `React.memo` not work?
153. Explain Reconciliation using Keys.
154. What happens internally from `setState()` to DOM update?

---

# Interview Readiness Checklist

If you can confidently answer:

- ✅ 80+ Questions → Good for Mid-Level React Interviews
- ✅ 120+ Questions → Strong for 4–5 Years Experience
- ✅ 140+ Questions → Senior Frontend Interview Ready
- ✅ 150+ Questions → React Internals & Architecture Level Understanding
