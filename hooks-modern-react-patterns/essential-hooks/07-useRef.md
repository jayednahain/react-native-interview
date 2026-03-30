# useRef Hook - Complete Guide

---

## 📌 Definition

**useRef** is a React Hook that lets you access DOM nodes or keep a mutable value that persists across renders. Unlike state, updating a ref doesn't trigger a re-render.

---

## 🎯 Purpose & What It Does

### Purpose

- **Access DOM elements** - Get direct access to DOM nodes (focus, play/pause video)
- **Keep mutable values** - Store values that persist across renders without re-renders
- **Store previous values** - Track previous state/props
- **Store timers/intervals** - Keep references to setTimeout/setInterval IDs
- **Maintain identity** - Keep same object reference across renders

### What It Actually Does

1. Creates an object with a .current property
2. Persists same object reference across renders
3. Updating .current doesn't trigger re-render
4. Returns same object on every render
5. Can store any value in .current

---

## 🔄 Class Component vs Hooks Comparison

### Class Component (Old Way)

```javascript
class TextInput extends React.Component {
  constructor(props) {
    super(props);
    // Create ref using React.createRef()
    this.inputRef = React.createRef();
  }

  focusInput = () => {
    // Access DOM node
    this.inputRef.current.focus();
  };

  render() {
    return (
      <>
        <input ref={this.inputRef} />
        <button onClick={this.focusInput}>Focus Input</button>
      </>
    );
  }
}
```

### Hooks Version (Modern Way)

```javascript
function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
}
```

---

## ⚠️ Important: useRef vs useState

| Aspect                 | useState        | useRef               |
| ---------------------- | --------------- | -------------------- |
| **Triggers re-render** | Yes             | No                   |
| **Mutable**            | No (immutable)  | Yes (mutable)        |
| **Update timing**      | Asynchronous    | Synchronous          |
| **Use case**           | Rendering state | Non-rendering values |
| **DOM access**         | No              | Yes                  |

```javascript
function Example() {
  // ❌ Wrong - use state for rendering values
  const count = useRef(0);
  count.current++; // Doesn't update UI

  // ✅ Correct - use state for rendering
  const [count, setCount] = useState(0);
}
```

---

## 💡 Real-World Use Cases

1. **Managing focus/selection** - Focus input, select text
2. **Triggering animations** - Play/pause animations
3. **Managing timers** - Keep track of setInterval IDs
4. **DOM measurements** - Get element size/position
5. **Storing previous values** - Track previous props
6. **Media control** - Play/pause video or audio
7. **Integrating with 3rd party libraries** - jQuery, D3, etc.

---

## 📝 Hook Examples

### Example 1: Focus Management

```javascript
import React, { useRef } from "react";
import { View, TextInput, Button } from "react-native";

function SearchBox() {
  const inputRef = useRef(null);

  const focusInput = () => {
    // Access DOM element directly
    inputRef.current?.focus();
  };

  const clearInput = () => {
    if (inputRef.current) {
      inputRef.current.clear();
      inputRef.current.focus();
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        ref={inputRef}
        placeholder="Search..."
        style={{ borderBottomWidth: 1, padding: 10, marginBottom: 10 }}
      />
      <Button title="Focus Input" onPress={focusInput} />
      <Button title="Clear Input" onPress={clearInput} />
    </View>
  );
}

export default SearchBox;
```

---

### Example 2: Mutable Value Without Re-render

```javascript
import React, { useRef, useState } from "react";
import { View, Text, Button } from "react-native";

function Counter() {
  const [renderCount, setRenderCount] = useState(0);

  // This value persists but doesn't trigger re-render
  const clickCountRef = useRef(0);

  const handleClick = () => {
    // Update ref - NO re-render
    clickCountRef.current++;
    console.log("Clicked", clickCountRef.current, "times");
  };

  const triggerRender = () => {
    // This triggers re-render
    setRenderCount(renderCount + 1);
  };

  return (
    <View style={{ padding: 20 }}>
      <Text>Render Count: {renderCount}</Text>
      <Text>
        Click Count (not shown, check console): {clickCountRef.current}
      </Text>

      <Button title="Click (no render)" onPress={handleClick} />
      <Button title="Force Render" onPress={triggerRender} />
    </View>
  );
}

export default Counter;
```

---

### Example 3: Managing Timers

```javascript
import React, { useRef, useState } from "react";
import { View, Text, Button } from "react-native";

function StopWatch() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  // Store interval ID to clear it later
  const intervalRef = useRef(null);

  const startTimer = () => {
    setIsRunning(true);
    // Store interval ID
    intervalRef.current = setInterval(() => {
      setTime((prevTime) => prevTime + 1);
    }, 1000);
  };

  const stopTimer = () => {
    setIsRunning(false);
    // Clear using stored ID
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };

  const resetTimer = () => {
    stopTimer();
    setTime(0);
  };

  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins.toString().padStart(2, "0")}:${secs.toString().padStart(2, "0")}`;
  };

  return (
    <View style={{ padding: 20, alignItems: "center" }}>
      <Text style={{ fontSize: 32, marginBottom: 20 }}>{formatTime(time)}</Text>

      <Button
        title={isRunning ? "Stop" : "Start"}
        onPress={isRunning ? stopTimer : startTimer}
      />
      <Button title="Reset" onPress={resetTimer} color="red" />
    </View>
  );
}

export default StopWatch;
```

---

### Example 4: Tracking Previous Value

```javascript
import React, { useRef, useEffect, useState } from "react";
import { View, Text, TextInput, Button } from "react-native";

function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value; // Update ref AFTER render
  }, [value]);

  return ref.current; // Return previous value
}

function NameComparison() {
  const [name, setName] = useState("John");
  const previousName = usePrevious(name);

  return (
    <View style={{ padding: 20 }}>
      <Text>Current Name: {name}</Text>
      <Text>Previous Name: {previousName}</Text>

      <TextInput
        value={name}
        onChangeText={setName}
        placeholder="Enter name"
        style={{ borderBottomWidth: 1, padding: 10, marginVertical: 10 }}
      />

      {previousName !== name && (
        <Text style={{ color: "blue" }}>
          Name changed from "{previousName}" to "{name}"
        </Text>
      )}
    </View>
  );
}

export default NameComparison;
```

---

### Example 5: API Call with Cleanup

```javascript
import React, { useRef, useEffect, useState } from "react";
import { View, Text, Button, ActivityIndicator } from "react-native";

function UserFetch({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  // Store abort controller reference
  const abortControllerRef = useRef(null);

  useEffect(() => {
    setLoading(true);

    // Create new abort controller
    abortControllerRef.current = new AbortController();

    const fetchUser = async () => {
      try {
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`,
          { signal: abortControllerRef.current.signal },
        );
        const data = await response.json();
        setUser(data);
      } catch (error) {
        if (error.name !== "AbortError") {
          console.error("Failed to fetch:", error);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchUser();

    // Cleanup: abort request if component unmounts
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [userId]);

  if (loading) return <ActivityIndicator />;
  if (!user) return <Text>No user found</Text>;

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 18, fontWeight: "bold" }}>{user.name}</Text>
      <Text>{user.email}</Text>
      <Text>{user.phone}</Text>
      <Text>{user.website}</Text>
    </View>
  );
}

export default UserFetch;
```

---

## ❓ Interview Questions & Answers

### Q1: When should you use useRef vs useState?

**Answer:**

```javascript
// Use useState for rendering data
const [count, setCount] = useState(0); // Component re-renders when it changes

// Use useRef for non-rendering data
const clickCount = useRef(0); // Component doesn't re-render when it changes

// Specific use cases:
// useState: Form inputs, toggle states, data displayed on UI
// useRef: DOM access, timers, previous values, integration with libraries
```

---

### Q2: Does updating a ref cause a re-render?

**Answer:**
**No**, updating a ref does NOT trigger a re-render:

```javascript
function Example() {
  const ref = useRef(0);
  const [state, setState] = useState(0);

  const handleClick = () => {
    ref.current++; // No re-render
    setState(state + 1); // Re-renders component
  };

  return <button onClick={handleClick}>Click</button>;
}
```

---

### Q3: Can you use useRef for storing DOM elements?

**Answer:**
**Yes**, that's one of the primary uses:

```javascript
function VideoPlayer() {
  const videoRef = useRef(null);

  const play = () => videoRef.current.play();
  const pause = () => videoRef.current.pause();

  return (
    <>
      <video ref={videoRef} />
      <button onClick={play}>Play</button>
      <button onClick={pause}>Pause</button>
    </>
  );
}
```

---

### Q4: What's the difference between useRef and creating a regular variable?

**Answer:**

```javascript
function Example() {
  // ❌ Wrong - new object every render
  const obj = {};

  // ✅ Correct - same object every render
  const objRef = useRef({});

  // When component re-renders:
  // - obj will be a NEW empty object
  // - objRef.current will be the SAME object
}
```

---

### Q5: How do you initialize a useRef?

**Answer:**

```javascript
// With initial value
const ref = useRef(initialValue);

// Common initializations
const inputRef = useRef(null); // For DOM elements
const countRef = useRef(0); // For numbers
const listRef = useRef([]); // For arrays
const configRef = useRef({}); // For objects
```

---

## 🔑 Key Takeaways

1. **useRef persists across renders** - Same object reference
2. **Doesn't trigger re-render** - Useful for non-UI values
3. **DOM access** - Primary use case
4. **Mutable by nature** - Directly modify .current
5. **Complements useState** - Use together for optimal results

---

## 🚨 Common Mistakes

```javascript
// ❌ Using ref for values that need to display
function Counter() {
  const count = useRef(0);
  count.current++; // UI never updates!
}

// ✅ Use state for display values
function Counter() {
  const [count, setCount] = useState(0); // UI updates
}

// ❌ Creating new ref on each render
function Example() {
  const ref = useRef(Math.random()); // New value each render!
}

// ✅ Initialize with constant
function Example() {
  const ref = useRef(null); // Same ref each render
}

// ❌ Mutating state directly (looks like ref but wrong)
function Example() {
  const [obj, setObj] = useState({ count: 0 });
  obj.count++; // Don't do this! Wrong!
  // useRef is fine with this
}
```

---

## 📚 Practice Exercise

Create a **Video Player Component** with:

1. Video element with ref
2. Play/Pause buttons
3. Progress bar to seek
4. Volume control
5. Full screen toggle
6. Current time and duration display
7. Use useRef to access video element

---

**Next:** Learn about [useLayoutEffect Hook](./08-useLayoutEffect.md)
