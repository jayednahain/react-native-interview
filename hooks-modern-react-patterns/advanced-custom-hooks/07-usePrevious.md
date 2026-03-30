# usePrevious Hook - Track Previous Values

## 📌 Definition

**usePrevious** is a custom hook that stores and returns the previous value of a variable. It's useful for comparing current and previous values, detecting changes, or handling animations based on value transitions.

---

## 🎯 Purpose & What It Does

- **Stores previous value** in ref
- **Compares current vs previous** for change detection
- **No re-render** triggered (uses ref, not state)
- **Works with any type** - primitives, objects, arrays
- **Perfect for animations** - animate transitions between values

---

## 💡 Real-World Examples

### Example 1: Simple Previous Value Tracking

```javascript
import { usePrevious } from "./hooks/usePrevious";
import { useState } from "react";
import { View, Text, TouchableOpacity } from "react-native";

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <View>
      <Text>
        Now: {count}, Before: {prevCount}
      </Text>
      <TouchableOpacity onPress={() => setCount(count + 1)}>
        <Text>Increment</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 2: Detect When Value Changes

```javascript
import { usePrevious } from "./hooks/usePrevious";
import { useEffect, useState } from "react";

function UserProfileUpdater({ userId }) {
  const [profile, setProfile] = useState(null);
  const prevUserId = usePrevious(userId);

  useEffect(() => {
    // Only fetch if userId changed (not on first render)
    if (prevUserId && prevUserId !== userId) {
      fetchProfile(userId);
    }
  }, [userId, prevUserId]);

  return <View>{/* show profile */}</View>;
}
```

### Example 3: Animate Value Changes

```javascript
import { usePrevious } from "./hooks/usePrevious";
import { useState, useEffect } from "react";
import { Animated, View, Text } from "react-native";

function AnimatedCounter({ number }) {
  const prevNumber = usePrevious(number);
  const [animValue] = useState(new Animated.Value(0));

  useEffect(() => {
    if (prevNumber !== number) {
      Animated.timing(animValue, {
        toValue: 1,
        duration: 500,
        useNativeDriver: false,
      }).start();
    }
  }, [number, prevNumber, animValue]);

  return (
    <View>
      <Text>{number}</Text>
      {prevNumber && <Text>Changed from {prevNumber}</Text>}
    </View>
  );
}
```

### Example 4: Form Change Detection

```javascript
import { usePrevious } from "./hooks/usePrevious";
import { useState } from "react";

function EditableForm({ initialData }) {
  const [formData, setFormData] = useState(initialData);
  const prevData = usePrevious(formData);

  const hasChanges = JSON.stringify(formData) !== JSON.stringify(prevData);

  return (
    <View>
      {/* form fields */}
      <TouchableOpacity disabled={!hasChanges}>
        <Text>{hasChanges ? "Save Changes" : "No Changes"}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 5: Track Multiple Previous Values

```javascript
import { usePrevious } from "./hooks/usePrevious";
import { useState } from "react";

function ComplexTracking({ userId, postId }) {
  const prevUserId = usePrevious(userId);
  const prevPostId = usePrevious(postId);

  const userChanged = prevUserId !== userId;
  const postChanged = prevPostId !== postId;

  return (
    <View>
      {userChanged && <Text>User switched to {userId}</Text>}
      {postChanged && <Text>Post switched to {postId}</Text>}
    </View>
  );
}
```

---

## ❓ Interview Q&A

**Q1: How do you implement usePrevious?**

A: Use useRef to store previous value and useEffect to update it.

```javascript
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}
```

**Q2: Why use useRef instead of useState?**

A: useRef doesn't trigger re-render. Perfect for storing values you don't need to react to.

```javascript
// Bad - causes infinite loop
const [prev, setPrev] = useState(value);
useEffect(() => {
  setPrev(value);
}, [value]);

// Good - no re-render
const prev = useRef();
useEffect(() => {
  prev.current = value;
}, [value]);
```

**Q3: Can usePrevious be used in conditions?**

A: Yes, but it's undefined on first render. Always check.

```javascript
const prevValue = usePrevious(value);

// Safe check
if (prevValue && prevValue !== value) {
  // value changed
}
```

**Q4: How do you use usePrevious with objects?**

A: Works the same, but compare by reference or use deep comparison.

```javascript
const prevUser = usePrevious(user);

// Shallow comparison
if (prevUser !== user) {
  /* changed */
}

// Deep comparison
if (JSON.stringify(prevUser) !== JSON.stringify(user)) {
  /* changed */
}
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Not Checking for undefined

```javascript
// BAD - prevValue is undefined on first render
if (prevValue !== value) {
  /* always true */
}

// GOOD - check existence
if (prevValue !== undefined && prevValue !== value) {
}
```

### ❌ Mistake 2: Using Previous in Dependency Array

```javascript
// BAD - infinite loop
useEffect(() => {
  setPrev(value);
}, [value, prev]);

// GOOD - prev should not be dependency
useEffect(() => {
  ref.current = value;
}, [value]);
```

---

## 📝 Practice Exercise

**Task:** Create a component that shows:

1. Current and previous count
2. Shows "Increased" or "Decreased"
3. Shows difference amount

**Solution:**

```javascript
import { usePrevious } from "./hooks/usePrevious";
import { useState } from "react";
import { View, Text, TouchableOpacity } from "react-native";

function CounterWithChange() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  const difference = count - (prevCount ?? 0);
  const direction = difference > 0 ? "↑" : difference < 0 ? "↓" : "=";

  return (
    <View>
      <Text style={{ fontSize: 24 }}>{count}</Text>
      {prevCount !== undefined && (
        <View>
          <Text>Previous: {prevCount}</Text>
          <Text>
            Change: {direction} {Math.abs(difference)}
          </Text>
        </View>
      )}
      <TouchableOpacity onPress={() => setCount(count + 1)}>
        <Text>+1</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={() => setCount(count - 1)}>
        <Text>-1</Text>
      </TouchableOpacity>
    </View>
  );
}

export default CounterWithChange;
```

---

## 🎓 Key Takeaways

✅ **Tracks previous values** without causing re-renders  
✅ **Perfect for change detection** - compare current vs previous  
✅ **Works with useEffect** - detect when dependencies change  
✅ **Use for animations** - animate transitions  
✅ **Simple implementation** - just 3 lines of code

---

**Next:** Learn about useToggle for boolean state management.
