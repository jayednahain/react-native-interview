# Essential Hooks - Complete Learning Guide

## 📚 Overview

Essential hooks are the foundation of modern React development. These are the hooks you'll use in almost every React component. Master these first before moving to advanced hooks.

---

## 🎯 Learning Path

### Module 1: Basic State & Effects

1. **[useState](./01-useState.md)** - State management
2. **[useEffect](./02-useEffect.md)** - Side effects
3. **[useContext](./03-useContext.md)** - Avoid prop drilling

### Module 2: Advanced State & Refs

4. **[useReducer](./04-useReducer.md)** - Complex state logic
5. **[useCallback](./05-useCallback.md)** - Memoize functions
6. **[useMemo](./06-useMemo.md)** - Memoize values
7. **[useRef](./07-useRef.md)** - Persistent references
8. **[useLayoutEffect](./08-useLayoutEffect.md)** - Synchronous effects

---

## 🔥 Quick Hook Comparison

| Hook                | Purpose                  | When to Use                        |
| ------------------- | ------------------------ | ---------------------------------- |
| **useState**        | Manage component state   | Form inputs, toggles, counters     |
| **useEffect**       | Run side effects         | Fetch data, subscriptions, cleanup |
| **useContext**      | Access context           | Avoid prop drilling                |
| **useReducer**      | Complex state logic      | Multiple related state, reducers   |
| **useCallback**     | Memoize functions        | Prevent re-renders, dependencies   |
| **useMemo**         | Memoize values           | Expensive calculations             |
| **useRef**          | Get/store mutable values | DOM access, persist across renders |
| **useLayoutEffect** | Synchronous effects      | DOM measurements before paint      |

---

## 📖 Hook Rules (Critical!)

```javascript
// Rule 1: Only call hooks at the top level
// ❌ DON'T - Inside conditions
if (someCondition) {
  const [state, setState] = useState(0);
}

// ✅ DO - At top level
const [state, setState] = useState(0);

// Rule 2: Only call hooks from React components or custom hooks
// ❌ DON'T - In regular JavaScript functions
function regularFunction() {
  const [state, setState] = useState(0); // Error!
}

// ✅ DO - In React components or custom hooks
function MyComponent() {
  const [state, setState] = useState(0); // Good!
}

// Rule 3: Exhaustive dependencies in useEffect
// ❌ DON'T - Missing dependency
useEffect(() => {
  console.log(count); // Uses stale value
}, []);

// ✅ DO - Include all dependencies
useEffect(() => {
  console.log(count);
}, [count]);
```

---

## 🎓 Study Recommendations

### Week 1-2: Fundamentals

- Focus on **useState** and **useEffect**
- Understand dependencies and cleanup
- Practice with simple examples

### Week 3-4: Core Concepts

- Add **useContext** to your toolkit
- Learn **useReducer** for complex state
- Start building real apps

### Week 5-6: Optimization

- Master **useCallback** and **useMemo**
- Learn when and why to memoize
- Understand performance implications

### Week 7-8: Advanced

- **useRef** for DOM access
- **useLayoutEffect** for measurements
- Custom hooks patterns

---

## 💡 Real-World Architecture

A typical modern React app architecture using hooks:

```
App
├── ThemeProvider (useContext)
│   ├── AuthProvider (useReducer + useEffect)
│   │   ├── Home Screen
│   │   ├── Profile Screen (useContext for auth, useEffect for data)
│   │   └── Settings Screen (useState for preferences)
│   └── ...
```

---

## 🚀 Common Patterns

### Pattern 1: Data Fetching with Loading State

```javascript
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        setData(await response.json());
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    fetchData();
  }, [url]);

  return { data, loading, error };
}

// Usage:
function UserList() {
  const { data: users, loading, error } = useApi("/api/users");
  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  return <List data={users} />;
}
```

### Pattern 2: Form Handling

```javascript
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await onSubmit(values);
    } catch (err) {
      setErrors(err.errors || {});
    }
  };

  return { values, errors, handleChange, handleSubmit };
}

// Usage:
function LoginForm() {
  const { values, handleChange, handleSubmit } = useForm(
    { email: "", password: "" },
    async (values) => {
      await loginUser(values);
    },
  );

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" value={values.email} onChange={handleChange} />
      <input name="password" value={values.password} onChange={handleChange} />
      <button type="submit">Login</button>
    </form>
  );
}
```

### Pattern 3: Toggle Hook

```javascript
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  const toggle = useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}

// Usage:
function Modal() {
  const [isOpen, toggleModal] = useToggle();
  return (
    <>
      <button onClick={toggleModal}>Open</button>
      {isOpen && <div>Modal Content</div>}
    </>
  );
}
```

---

## 🎯 Interview Preparation Checklist

- [ ] Understand useState lifecycle
- [ ] Know useEffect dependencies and cleanup
- [ ] Explain context without prop drilling
- [ ] Difference between useReducer and useState
- [ ] When to use useCallback and useMemo
- [ ] useRef vs useState
- [ ] useLayoutEffect vs useEffect
- [ ] Can explain stale closures
- [ ] Can fix infinite loops in useEffect
- [ ] Can create custom hooks

---

## 📚 Additional Resources

- [React Official Hooks Documentation](https://react.dev/reference/react)
- [Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [Custom Hooks Best Practices](https://react.dev/learn/reusing-logic-with-custom-hooks)

---

**Let's start learning! Pick a hook above and dive deep.** 🚀
