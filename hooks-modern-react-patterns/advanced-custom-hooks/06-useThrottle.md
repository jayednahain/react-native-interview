# useThrottle Hook - Throttle Function Calls

## 📌 Definition

**useThrottle** is a custom hook that limits how frequently a function can be called. It ensures a function is invoked at most once per specified time interval. Perfect for expensive operations like scroll handlers, window resize, or API calls triggered by rapid user input.

---

## 🎯 Purpose & What It Does

useThrottle solves performance problems:

- **Limits function calls** - max once per N milliseconds
- **Reduces unnecessary computation** - especially for events
- **Improves performance** - prevents handler spam
- **Conserves API calls** - throttle expensive requests
- **Smooth animations** - prevents jank from event handlers

**Key Difference from Debounce:**

- **Throttle**: Executes every N milliseconds (event fires continuously at intervals)
- **Debounce**: Executes after N milliseconds of inactivity (event fires only when user stops)

---

## 🔄 Class Component Equivalent

### Class Component Approach

```javascript
class ScrollPerformance extends React.Component {
  throttleTimer = null;

  handleScroll = (event) => {
    if (this.throttleTimer) return;

    this.throttleTimer = setTimeout(() => {
      this.throttleTimer = null;
      this.onScroll(event);
    }, 1000);
  };

  onScroll = (event) => {
    console.log("Scroll position:", event.nativeEvent.contentOffset.y);
  };

  render() {
    return (
      <ScrollView onScroll={this.handleScroll}>{/* content */}</ScrollView>
    );
  }
}
```

### Modern Hook Approach

```javascript
function ScrollPerformance() {
  const onScroll = useThrottle((event) => {
    console.log("Scroll position:", event.nativeEvent.contentOffset.y);
  }, 1000);

  return <ScrollView onScroll={onScroll}>{/* content */}</ScrollView>;
}
```

---

## 💡 Real-World Examples

### Example 1: Throttle Scroll Events

```javascript
import { useThrottle } from "./hooks/useThrottle";
import { ScrollView, Text } from "react-native";

function InfiniteScrollWithThrottle() {
  const onScroll = useThrottle((event) => {
    const scrollPosition = event.nativeEvent.contentOffset.y;
    console.log("User scrolled to:", scrollPosition);
    // Load more items logic
  }, 500); // Call at most once every 500ms

  return (
    <ScrollView onScroll={onScroll}>
      {/* Large list content */}
      <Text>Item 1</Text>
      <Text>Item 2</Text>
      {/* ... many items ... */}
    </ScrollView>
  );
}
```

### Example 2: Throttle Window Resize

```javascript
import { useThrottle } from "./hooks/useThrottle";
import { useEffect, useState } from "react";
import { useWindowDimensions } from "react-native";

function ResponsiveLayout() {
  const [columnCount, setColumnCount] = useState(2);
  const { width } = useWindowDimensions();

  const handleResize = useThrottle(() => {
    setColumnCount(width > 600 ? 3 : 2);
  }, 300);

  useEffect(() => {
    handleResize();
  }, [width]);

  return (
    <View>
      <Text>Columns: {columnCount}</Text>
    </View>
  );
}
```

### Example 3: Throttle Mouse/Touch Events

```javascript
import { useThrottle } from "./hooks/useThrottle";
import { View, PanResponder, useRef } from "react-native";

function DraggableElement() {
  const pan = useRef(new Animated.ValueXY()).current;

  const handlePan = useThrottle((x, y) => {
    pan.setValue({ x, y });
  }, 16); // ~60fps throttle (1000/60 ≈ 16ms)

  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderMove: (e, { dx, dy }) => {
        handlePan(dx, dy);
      },
    }),
  ).current;

  return (
    <Animated.View
      {...panResponder.panHandlers}
      style={[pan.getLayout(), { width: 100, height: 100 }]}
    />
  );
}
```

### Example 4: Throttle API Calls During Search

```javascript
import { useThrottle } from "./hooks/useThrottle";
import { useState } from "react";
import { TextInput } from "react-native";

function SearchUsers() {
  const [results, setResults] = useState([]);

  const searchAPI = useThrottle(async (query) => {
    if (query.length < 2) return;
    const response = await fetch(`/api/users?search=${query}`);
    const data = await response.json();
    setResults(data);
  }, 500); // Max 1 request per 500ms

  return <TextInput onChangeText={searchAPI} placeholder="Search users..." />;
}
```

### Example 5: Throttle Save Operations

```javascript
import { useThrottle } from "./hooks/useThrottle";
import { useState } from "react";

function DocumentEditor() {
  const [content, setContent] = useState("");

  const saveDocument = useThrottle(async (text) => {
    console.log("Saving...", text);
    await fetch("/api/document", {
      method: "PUT",
      body: JSON.stringify({ content: text }),
    });
  }, 3000); // Save at most every 3 seconds

  const handleChange = (text) => {
    setContent(text);
    saveDocument(text);
  };

  return <TextInput value={content} onChangeText={handleChange} />;
}
```

---

## ❓ Interview Q&A

**Q1: What's the difference between throttle and debounce?**

A:

- **Throttle**: Executes every N ms while event is firing (continuous intervals)
- **Debounce**: Executes after N ms of no events (single execution after pause)

```javascript
// Throttle: calls every 300ms while scrolling
useThrottle(handleScroll, 300);
// Output: 0ms, 300ms, 600ms, 900ms, ...

// Debounce: calls 300ms after user stops scrolling
useDebounce(handleScroll, 300);
// Output: 1200ms (300ms after last event)
```

**Q2: When would you use throttle vs debounce?**

A:

- **Throttle**: Scroll events, resize handlers, game loops, continuous monitoring
- **Debounce**: Search input, form validation, auto-save, expensive API calls

**Q3: How do you implement throttle?**

A: Track last execution time and skip if within throttle window.

```javascript
function useThrottle(callback, delay) {
  const lastRun = useRef(null);

  return useCallback(
    (...args) => {
      const now = Date.now();
      if (!lastRun.current || now - lastRun.current >= delay) {
        callback(...args);
        lastRun.current = now;
      }
    },
    [callback, delay],
  );
}
```

**Q4: What's `leading` vs `trailing` execution?**

A:

- **Leading**: Execute immediately, then wait N ms
- **Trailing**: Wait N ms, then execute, then wait N ms

```javascript
// Leading (default)
handleScroll(); // Executes immediately
// Wait 300ms
// Next scroll at 250ms: ignored
// Next scroll at 400ms: executes

// Trailing
// Wait 300ms
// Executes first scroll
// Wait 300ms
// Executes second scroll
```

**Q5: How do you cancel a pending throttled call?**

A: Return cleanup function from the hook.

```javascript
function useThrottle(callback, delay) {
  const timeoutId = useRef(null);

  const throttled = useCallback(
    (...args) => {
      if (timeoutId.current) clearTimeout(timeoutId.current);
      timeoutId.current = setTimeout(() => {
        callback(...args);
      }, delay);
    },
    [callback, delay],
  );

  return throttled;
}
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Throttle vs Debounce Confusion

```javascript
// BAD - using debounce for scroll (fires too late)
const handleScroll = useDebounce(onScroll, 300);

// GOOD - use throttle for scroll (fires at intervals)
const handleScroll = useThrottle(onScroll, 300);
```

### ❌ Mistake 2: Creating New Throttled Function Every Render

```javascript
// BAD - loses throttle state on re-render
const handleClick = () => {
  const throttled = throttle(expensiveOperation, 1000);
  throttled();
};

// GOOD - throttle is memoized
const throttledClick = useThrottle(expensiveOperation, 1000);
const handleClick = () => throttledClick();
```

### ❌ Mistake 3: Using Throttle Delay That's Too Long

```javascript
// BAD - 5 second throttle = poor UX
useThrottle(handleScroll, 5000);

// GOOD - 300-500ms for most cases
useThrottle(handleScroll, 300);
```

---

## 📝 Practice Exercise

**Task:** Create a draggable box that:

1. Throttles drag position updates to 60fps (~16ms)
2. Shows current X,Y position
3. Throttles position logging to console
4. Shows how many times per second it updates

**Solution:**

```javascript
import { useThrottle } from "./hooks/useThrottle";
import { useState, useRef } from "react";
import { View, Text, PanResponder, Animated } from "react-native";

function DraggableBox() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [callCount, setCallCount] = useState(0);
  const pan = useRef(new Animated.ValueXY()).current;
  const countRef = useRef(0);

  const updatePosition = useThrottle((x, y) => {
    setPosition({ x, y });
    countRef.current++;
  }, 16); // ~60fps

  // Monitor update frequency
  useThrottle(() => {
    setCallCount(countRef.current);
    countRef.current = 0;
  }, 1000);

  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderMove: (e, { dx, dy }) => {
        updatePosition(dx, dy);
        pan.setValue({ x: dx, y: dy });
      },
    }),
  ).current;

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Text>
        X: {position.x.toFixed(0)}, Y: {position.y.toFixed(0)}
      </Text>
      <Text>Updates/sec: {callCount}</Text>

      <Animated.View
        {...panResponder.panHandlers}
        style={[
          pan.getLayout(),
          { width: 100, height: 100, backgroundColor: "blue" },
        ]}
      />
    </View>
  );
}

export default DraggableBox;
```

---

## 🎓 Key Takeaways

✅ **Throttle limits function frequency** - once per N milliseconds  
✅ **Improves performance** - reduces handler calls  
✅ **Perfect for scroll/resize** - continuous events  
✅ **Use with PanResponder** - smooth dragging at 60fps  
✅ **Different from debounce** - fires continuously vs once after pause

---

**Next:** Learn about usePrevious for tracking previous values.
