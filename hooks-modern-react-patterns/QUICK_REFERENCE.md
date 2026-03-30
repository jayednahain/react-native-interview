# Quick Reference: All Essential Hooks

A one-page reference guide for all 8 essential React hooks.

---

## 1️⃣ useState - State Management

**Syntax:**

```javascript
const [state, setState] = useState(initialValue);
```

**When to use:** Form inputs, toggles, counters, UI state

**Example:**

```javascript
const [count, setCount] = useState(0);
setCount(count + 1); // or use functional update
setCount((prev) => prev + 1); // Recommended
```

**Interview tip:** Explain that it triggers re-render and is the most basic hook.

---

## 2️⃣ useEffect - Side Effects & Lifecycle

**Syntax:**

```javascript
useEffect(() => {
  // Code here
  return () => {
    // Cleanup here
  };
}, [dependencies]);
```

**When to use:** Data fetching, subscriptions, event listeners, cleanup

**Dependency array rules:**

- `[]` = Run once on mount
- `[dep1, dep2]` = Run when deps change
- No array = Run after every render

**Example:**

```javascript
useEffect(() => {
  const handleResize = () => console.log("resized");
  window.addEventListener("resize", handleResize);
  return () => window.removeEventListener("resize", handleResize);
}, []);
```

**Interview tip:** Be ready to explain memory leaks and cleanup functions.

---

## 3️⃣ useContext - Avoid Prop Drilling

**Syntax:**

```javascript
const value = useContext(MyContext);
```

**When to use:** Global state (theme, auth, language)

**Pattern:**

```javascript
const MyContext = createContext();

function Provider({ children }) {
  return <MyContext.Provider value={data}>{children}</MyContext.Provider>;
}

function useMyContext() {
  return useContext(MyContext);
}
```

**Interview tip:** Explain prop drilling problem and context solution clearly.

---

## 4️⃣ useReducer - Complex State

**Syntax:**

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

**When to use:** Multiple related state values, complex updates

**Pattern:**

```javascript
function reducer(state, action) {
  switch (action.type) {
    case "ACTION":
      return { ...state, value: action.payload };
    default:
      return state;
  }
}

const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: "ACTION", payload: value });
```

**Interview tip:** Show understanding of reducer pattern (similar to Redux).

---

## 5️⃣ useCallback - Memoize Functions

**Syntax:**

```javascript
const memoizedFn = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

**When to use:** Passing callbacks to memoized children, effect dependencies

**Example:**

```javascript
const handleClick = useCallback(() => {
  setCount((prev) => prev + 1);
}, []);

<MemoButton onClick={handleClick} />; // Won't re-render unless onClick changes
```

**Interview tip:** Explain when it helps vs when it's unnecessary overhead.

---

## 6️⃣ useMemo - Cache Values

**Syntax:**

```javascript
const memoizedValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

**When to use:** Expensive calculations, prevent child re-renders

**Example:**

```javascript
const filtered = useMemo(() => {
  return items.filter((item) => item.name.includes(query));
}, [items, query]);
```

**Interview tip:** Know the difference between useCallback and useMemo.

---

## 7️⃣ useRef - Mutable Values & DOM Access

**Syntax:**

```javascript
const ref = useRef(initialValue);
// Access with ref.current
```

**When to use:** DOM access, storing mutable values, timers

**Example:**

```javascript
const inputRef = useRef(null);
const focusInput = () => inputRef.current?.focus();

<input ref={inputRef} />;
```

**Key difference:** Doesn't trigger re-render on update

**Interview tip:** Distinguish between useRef and useState use cases.

---

## 8️⃣ useLayoutEffect - Before Paint

**Syntax:**

```javascript
useLayoutEffect(() => {
  // Runs BEFORE browser paint
}, [deps]);
```

**When to use:** DOM measurements, preventing visual flashing

**Important:** Blocks rendering (use sparingly)

**Example:**

```javascript
useLayoutEffect(() => {
  const { height } = ref.current.getBoundingClientRect();
  setHeight(height);
}, []);
```

**Interview tip:** Explain when to use vs useEffect (rarely needed).

---

## 📊 Comparison Matrix

| Hook            | Triggers Re-render | Mutable | Use for       |
| --------------- | ------------------ | ------- | ------------- |
| useState        | ✅                 | ❌      | UI state      |
| useEffect       | ❌                 | ❌      | Side effects  |
| useContext      | ✅                 | ❌      | Global state  |
| useReducer      | ✅                 | ❌      | Complex state |
| useCallback     | ❌                 | ❌      | Functions     |
| useMemo         | ❌                 | ❌      | Values        |
| useRef          | ❌                 | ✅      | DOM/timers    |
| useLayoutEffect | ❌                 | ❌      | Measurements  |

---

## 🚀 Common Patterns

### Data Fetching

```javascript
useEffect(() => {
  setLoading(true);
  fetch(url)
    .then((r) => r.json())
    .then(setData)
    .catch(setError)
    .finally(() => setLoading(false));
}, [url]);
```

### Form Input

```javascript
const [formData, setFormData] = useState({ name: "", email: "" });
const handleChange = (e) => {
  setFormData((prev) => ({ ...prev, [e.target.name]: e.target.value }));
};
```

### Global State with Context

```javascript
const Store = createContext();
function Provider({ children }) {
  const [state, setState] = useState(initial);
  return (
    <Store.Provider value={{ state, setState }}>{children}</Store.Provider>
  );
}
const { state } = useContext(Store);
```

### Performance Optimization

```javascript
const MemoComponent = React.memo(Component);
const handleClick = useCallback(() => {}, [deps]);
const value = useMemo(() => compute(), [deps]);
```

---

## ❌ Common Mistakes

```javascript
// ❌ Conditional hooks
if (condition) {
  const [state, setState] = useState(0);
}

// ❌ Missing dependencies
useEffect(() => {
  console.log(count);
}, []); // Should include [count]

// ❌ Using state for non-rendering data
const count = useRef(0);
count.current++; // Won't update UI

// ❌ Not cleaning up
useEffect(() => {
  const listener = () => {};
  window.addEventListener("resize", listener);
  // Missing return cleanup!
}, []);

// ❌ Mutating state
const [obj, setObj] = useState({ count: 0 });
obj.count++; // Wrong! Use spread
```

---

## ✅ Best Practices

1. **Keep hooks at top level** - Never conditional
2. **Include dependencies** - Let ESLint guide you
3. **Use functional updates** - When depending on previous state
4. **Clean up effects** - Return cleanup function
5. **Use custom hooks** - For reusable logic
6. **Prefer composition** - Over multiple hooks
7. **Memoize wisely** - Only when proven necessary
8. **Test your logic** - Especially custom hooks

---

## 🎯 Interview Questions Quick Answers

**Q: What are the Rules of Hooks?**
A: Only call hooks at top level and only in React components.

**Q: When does useEffect run?**
A: After render, with timing controlled by dependency array.

**Q: Why do we need dependencies?**
A: To prevent stale closures and infinite loops.

**Q: useState vs useReducer?**
A: useState for simple, useReducer for complex related state.

**Q: How to prevent infinite loops?**
A: Include all dependencies and use functional updates.

**Q: What's prop drilling?**
A: Passing props through many intermediate components unnecessarily.

**Q: When to use useCallback?**
A: When passing to React.memo children or as effect dependency.

**Q: When to use useMemo?**
A: For expensive computations passed to memoized components.

---

## 📝 Quick Checklist

Before interviews, you should be able to:

- [ ] Implement useState with multiple states
- [ ] Handle useEffect with cleanup
- [ ] Use useContext to avoid prop drilling
- [ ] Write a useReducer with proper reducer function
- [ ] Explain when useCallback is needed
- [ ] Know difference between useCallback and useMemo
- [ ] Access DOM with useRef
- [ ] Know when useLayoutEffect is necessary
- [ ] Spot memory leaks
- [ ] Write custom hooks

---

**Confidence Level After Mastering:** 🔥🔥🔥 Ready for interviews!

Print this page or bookmark it for quick reference during learning.
