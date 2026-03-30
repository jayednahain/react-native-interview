# Advanced & Custom Hooks - Complete Learning Guide

---

## 📚 Overview

Advanced hooks are built on top of essential hooks and solve specific, real-world problems. Custom hooks are created by combining essential hooks to encapsulate reusable logic.

---

## 🎯 Learning Path

### Module 1: Custom Hooks (Foundation)

1. **[useAsync](./01-useAsync.md)** - Handle async operations
2. **[useFetch](./02-useFetch.md)** - Data fetching with state
3. **[useDebounce](./03-useDebounce.md)** - Debounce values
4. **[useThrottle](./04-useThrottle.md)** - Throttle function calls
5. **[usePrevious](./05-usePrevious.md)** - Track previous values
6. **[useToggle](./06-useToggle.md)** - Boolean state toggling
7. **[useLocalStorage](./07-useLocalStorage.md)** - Persist to localStorage
8. **[useForm](./08-useForm.md)** - Form handling and validation

### Module 2: React Query Hooks

9. **[useQuery](./09-useQuery.md)** - Server state management
10. **[useMutation](./10-useMutation.md)** - Optimistic updates
11. **[useInfiniteQuery](./11-useInfiniteQuery.md)** - Infinite scrolling

### Module 3: React Native Specialized Hooks

12. **[useKeyboard](./12-useKeyboard.md)** - Keyboard management
13. **[useOrientation](./13-useOrientation.md)** - Device orientation
14. **[useNetInfo](./14-useNetInfo.md)** - Network connectivity
15. **[useAppState](./15-useAppState.md)** - App lifecycle
16. **[useDimensions](./16-useDimensions.md)** - Screen dimensions
17. **[useAnimatedValue](./17-useAnimatedValue.md)** - Reanimated animations
18. **[useSharedValue](./18-useSharedValue.md)** - Reanimated shared values
19. **[useAnimatedStyle](./19-useAnimatedStyle.md)** - Animated styles

---

## 🔥 Quick Hook Comparison

| Hook                | Purpose                    | Complexity | Most Used |
| ------------------- | -------------------------- | ---------- | --------- |
| **useAsync**        | Async operation handler    | Medium     | ⭐⭐⭐    |
| **useFetch**        | API data fetching          | Medium     | ⭐⭐⭐    |
| **useDebounce**     | Debounce input/values      | Low        | ⭐⭐⭐    |
| **useThrottle**     | Throttle function calls    | Low        | ⭐⭐      |
| **usePrevious**     | Track previous value       | Low        | ⭐⭐      |
| **useToggle**       | Boolean toggling           | Low        | ⭐⭐⭐    |
| **useLocalStorage** | Persist to storage         | Medium     | ⭐⭐⭐    |
| **useForm**         | Form state management      | High       | ⭐⭐⭐    |
| **useQuery**        | Server state (React Query) | High       | ⭐⭐⭐    |
| **useMutation**     | Server mutations           | High       | ⭐⭐⭐    |
| **useKeyboard**     | Keyboard events            | Medium     | ⭐⭐      |
| **useAppState**     | App foreground/background  | Medium     | ⭐⭐      |
| **useNetInfo**      | Network connectivity       | Low        | ⭐⭐      |

---

## 🎓 When to Build vs When to Use Libraries

### Build Custom Hooks When:

1. Logic is specific to your app
2. You need fine-grained control
3. It's a simple wrapper around hooks
4. You want to avoid external dependencies

### Use Library Hooks When:

1. Problem is solved by popular libraries
2. Need advanced features (caching, retry logic)
3. Community-maintained (React Query, Formik)
4. Significant maintenance burden to build

---

## 📖 Common Patterns

### Pattern 1: Async Handler

```javascript
function useAsync(asyncFn, immediate = true) {
  const [status, setStatus] = useState(immediate ? "pending" : "idle");
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(async () => {
    try {
      setStatus("pending");
      const response = await asyncFn();
      setValue(response);
      setStatus("success");
    } catch (error) {
      setError(error);
      setStatus("error");
    }
  }, [asyncFn]);

  useEffect(() => {
    if (immediate) execute();
  }, [execute, immediate]);

  return { execute, status, value, error };
}
```

### Pattern 2: Debounce

```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}
```

### Pattern 3: Form State

```javascript
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = useCallback((field, value) => {
    setValues((v) => ({ ...v, [field]: value }));
  }, []);

  const handleSubmit = useCallback(async () => {
    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } catch (error) {
      setErrors(error.errors || {});
    } finally {
      setIsSubmitting(false);
    }
  }, [values, onSubmit]);

  return { values, errors, handleChange, handleSubmit, isSubmitting };
}
```

---

## 🎯 Interview Checklist

Before interviews, be able to:

- [ ] Explain what custom hooks are and why they're useful
- [ ] Create a simple custom hook (useFetch, useDebounce)
- [ ] Understand React Query's useQuery and useMutation
- [ ] Know when to use custom vs library hooks
- [ ] Handle loading/error states properly
- [ ] Implement debounce and throttle
- [ ] Create a reusable form hook
- [ ] Understand infinite queries
- [ ] Know React Native specific hooks
- [ ] Build composition of multiple hooks

---

## 🚀 Project Ideas Using Advanced Hooks

### Beginner

1. Search bar with useDebounce
2. Todo app with useLocalStorage
3. Toggle sidebar with useToggle
4. Form with useForm hook

### Intermediate

5. Data fetching with useFetch and error handling
6. Infinite scroll with useInfiniteQuery
7. Search with debounce + API call
8. Form with validation and submission

### Advanced

9. Complete CRUD app with React Query
10. Chat app with real-time updates
11. E-commerce with complex filters
12. Analytics dashboard with multiple queries

---

## 📚 Study Timeline

### Week 1: Custom Hooks Foundation

- useAsync, useFetch, useDebounce
- Build simple custom hooks
- Practice creating reusable logic

### Week 2: Form & State Hooks

- useForm, useToggle, usePrevious
- useLocalStorage persistence
- Form validation patterns

### Week 3: React Query

- useQuery basics
- useMutation with optimistic updates
- useInfiniteQuery for pagination

### Week 4: Specialized Hooks

- React Native specific hooks
- Animations with Reanimated
- App lifecycle and device info

---

## 🔗 Resources

### Official Docs

- [React Query Documentation](https://tanstack.com/query/latest)
- [Reanimated Documentation](https://docs.swmansion.com/react-native-reanimated/)
- [React Hook Form](https://react-hook-form.com/)

### Recommended Reading

- Custom Hooks Patterns
- React Query Best Practices
- Form Handling in React Native

---

**Let's start with [useAsync Hook](./01-useAsync.md)** 🚀
