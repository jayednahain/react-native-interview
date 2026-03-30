# Hooks & Modern React Patterns - Complete Learning Hub

---

## 🎯 Overview

This comprehensive guide covers all hooks in modern React development, from essential to advanced custom hooks. Each section includes:

- 📌 Clear definitions
- 🎯 Purpose and use cases
- 🔄 Class component comparisons
- 💡 Real-world examples
- ❓ Interview questions
- 📝 Practice exercises

---

## 📚 Learning Structure

### Phase 1: Essential Hooks (Must Master First)

These 8 hooks form the foundation of modern React development. Master these completely before moving to advanced hooks.

#### 1. [useState](./essential-hooks/01-useState.md)

- **Purpose:** Manage component-level state
- **When to use:** Form inputs, toggles, counters, UI state
- **Key concept:** Triggers re-render on state change
- **Interview weight:** ⭐⭐⭐ (Asked in almost every interview)

#### 2. [useEffect](./essential-hooks/02-useEffect.md)

- **Purpose:** Run side effects and manage lifecycle
- **When to use:** API calls, subscriptions, cleanup
- **Key concept:** Dependency array controls execution
- **Interview weight:** ⭐⭐⭐ (Critical for data fetching)

#### 3. [useContext](./essential-hooks/03-useContext.md)

- **Purpose:** Share data without prop drilling
- **When to use:** Global state like theme, auth, preferences
- **Key concept:** Avoids intermediate component props
- **Interview weight:** ⭐⭐ (Often asked in state management section)

#### 4. [useReducer](./essential-hooks/04-useReducer.md)

- **Purpose:** Manage complex state logic with actions
- **When to use:** Multiple related state values, complex updates
- **Key concept:** Pure reducer function pattern
- **Interview weight:** ⭐⭐⭐ (Shows understanding of Redux patterns)

#### 5. [useCallback](./essential-hooks/05-useCallback.md)

- **Purpose:** Memoize functions for optimization
- **When to use:** Passing callbacks to memoized children
- **Key concept:** Prevents function recreation
- **Interview weight:** ⭐⭐ (Performance optimization topic)

#### 6. [useMemo](./essential-hooks/06-useMemo.md)

- **Purpose:** Cache computed values
- **When to use:** Expensive calculations, object creation for props
- **Key concept:** Prevents recalculation
- **Interview weight:** ⭐⭐ (Performance optimization)

#### 7. [useRef](./essential-hooks/07-useRef.md)

- **Purpose:** Access DOM or store mutable values
- **When to use:** DOM focus, timers, previous values
- **Key concept:** Doesn't trigger re-render
- **Interview weight:** ⭐⭐ (DOM access patterns)

#### 8. [useLayoutEffect](./essential-hooks/08-useLayoutEffect.md)

- **Purpose:** Synchronous effects before paint
- **When to use:** DOM measurements, preventing flashing
- **Key concept:** Blocks rendering (use rarely)
- **Interview weight:** ⭐ (Rarely asked, but good to know)

**[→ Essential Hooks Overview](./essential-hooks/README.md)**

---

### Phase 2: Advanced & Custom Hooks (Building Blocks)

Coming next! These hooks are built on essential hooks and solve specific problems.

#### Custom Hooks (Next)

- **useAsync** - Handle async operations
- **useDebounce** - Debounce input values
- **useFetch** - Data fetching with loading/error
- **useForm** - Form handling and validation
- **usePrevious** - Track previous values
- **useToggle** - Boolean state toggling
- **useWindowSize** - Track window dimensions
- **useLocalStorage** - Persist to localStorage

#### React Query Hooks (Next)

- **useQuery** - Server state management
- **useMutation** - Optimistic updates
- **useInfiniteQuery** - Infinite scrolling

#### Specialized Hooks (Next)

- **useKeyboard** - Keyboard management
- **useOrientation** - Device orientation
- **useNetInfo** - Network connectivity
- **useAppState** - App lifecycle
- **useDimensions** - Screen dimensions

**[Advanced & Custom Hooks Coming Soon](./advanced-custom-hooks/README.md)**

---

### Phase 3: State Management Solutions

After mastering hooks, learn modern state management:

- Zustand (recommended for most projects)
- TanStack Query (React Query)
- Redux Toolkit + RTK Query
- Context API patterns
- Custom state management

---

## 🎓 Learning Recommendations

### For Interviews

**1-2 Weeks Focus:**

- Master useState and useEffect deeply
- Understand dependency arrays
- Practice data fetching patterns
- Know useContext vs prop drilling
- Be ready to explain useCallback vs useMemo

**Common Interview Questions:**

```
1. "Explain the React hook rules"
2. "What's the difference between useState and useReducer?"
3. "Why do we need the dependency array in useEffect?"
4. "When would you use useCallback?"
5. "What's prop drilling and how does Context solve it?"
6. "How would you implement a custom hook?"
7. "Explain useRef and when you'd use it"
8. "What causes memory leaks in useEffect?"
```

### For Building Real Apps

**Priority Order:**

1. ✅ useState & useEffect (fundamentals)
2. ✅ useContext (global state)
3. ✅ useCallback & useMemo (optimization)
4. ✅ Custom hooks (code reusability)
5. ✅ useReducer (complex state)
6. ✅ State management library (Zustand/Redux)

---

## 🛠️ Hands-On Learning Path

### Week 1: Fundamentals

```
Day 1-2: useState
Day 3-4: useEffect with dependency arrays
Day 5-6: useContext
Day 7: Practice project - Todo App
```

### Week 2: Intermediate

```
Day 1-2: useReducer
Day 3-4: useCallback & useMemo
Day 5-6: useRef & useLayoutEffect
Day 7: Practice project - Shopping Cart
```

### Week 3: Advanced & Custom

```
Day 1-2: Custom hooks
Day 3-4: React Query patterns
Day 5-6: Complex state management
Day 7: Practice project - Real-world App
```

---

## 💡 Common Patterns You'll Use

### Pattern 1: Data Fetching

```javascript
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}
```

### Pattern 2: Form Handling

```javascript
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);

  const handleChange = useCallback((name, value) => {
    setValues((v) => ({ ...v, [name]: value }));
  }, []);

  const handleSubmit = useCallback(
    async (e) => {
      e.preventDefault();
      await onSubmit(values);
    },
    [values, onSubmit],
  );

  return { values, handleChange, handleSubmit };
}
```

### Pattern 3: Global State

```javascript
const StoreContext = createContext();

function Provider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StoreContext.Provider value={{ state, dispatch }}>
      {children}
    </StoreContext.Provider>
  );
}

function useStore() {
  return useContext(StoreContext);
}
```

---

## 🚀 Practice Projects

### Beginner

1. **Counter App** - useState, useCallback
2. **Todo List** - useState, useEffect, hooks list
3. **Weather App** - useEffect, useState, API calls

### Intermediate

4. **Shopping Cart** - useReducer, useContext
5. **Blog Platform** - Multiple hooks, data fetching
6. **User Dashboard** - Complex state, API integration

### Advanced

7. **Social Media Feed** - Infinite scroll, real-time
8. **E-commerce Site** - Full state management
9. **Chat Application** - WebSocket, complex state

---

## 📊 Hook Decision Tree

```
Does it affect rendering?
  ├─ YES
  │  ├─ Simple state? → useState
  │  ├─ Complex state? → useReducer
  │  └─ Share across tree? → useContext
  └─ NO
     ├─ Need DOM access? → useRef
     └─ Side effect? → useEffect
          └─ Before paint? → useLayoutEffect
          └─ After paint? → useEffect

Need to optimize?
  ├─ Memoize function? → useCallback
  ├─ Memoize value? → useMemo
  └─ Memoize component? → React.memo()
```

---

## ❓ FAQ

**Q: How many hooks can I use in one component?**
A: No limit! Use as many as needed, but consider breaking into smaller components if it gets unwieldy.

**Q: Can I call hooks conditionally?**
A: No! Always call hooks at the top level of components. Conditional hooks violate the Rules of Hooks.

**Q: When should I use useReducer vs useState?**
A: useState for independent state, useReducer for related state that updates together.

**Q: Should I use Context for everything?**
A: No! Use Context for truly global state. For local component state, use useState/useReducer.

**Q: Is useCallback always needed for performance?**
A: No! Only use when passing to memoized children. For regular callbacks, it's overhead.

---

## 🎯 Interview Checklist

Before your interview, make sure you can:

- [ ] Explain all 8 essential hooks clearly
- [ ] Compare class components with hooks
- [ ] Describe the Rules of Hooks
- [ ] Handle the dependency array correctly
- [ ] Implement custom hooks
- [ ] Solve infinite loop problems
- [ ] Understand closure and stale values
- [ ] Choose between hooks correctly
- [ ] Explain hook performance optimization
- [ ] Handle API calls and cleanup

---

## 📚 Additional Resources

### Official Documentation

- [React Hooks Official Docs](https://react.dev/reference/react/hooks)
- [Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [Custom Hooks Guide](https://react.dev/learn/reusing-logic-with-custom-hooks)

### Recommended Learning

- [Epic React - Kent C. Dodds](https://epicreact.dev/)
- [React Query Documentation](https://tanstack.com/query/latest)
- [Advanced React Patterns](https://www.patterns.dev/posts/custom-hooks/)

---

## 🎓 Next Steps

1. **Master Essential Hooks** → Spend 2 weeks on Phase 1
2. **Learn Custom Hooks** → Build reusable hook patterns
3. **Study State Management** → Learn Zustand/Redux
4. **Build Real Projects** → Apply knowledge in practice
5. **Interview Prep** → Mock interviews and Q&A

---

**Ready to start? Begin with [Essential Hooks](./essential-hooks/README.md)** 🚀

---

**Last Updated:** March 2026  
**Hook Coverage:** All modern hooks + patterns  
**Interview Ready:** Yes ✅
