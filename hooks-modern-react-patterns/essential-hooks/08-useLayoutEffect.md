# useLayoutEffect Hook - Complete Guide

---

## 📌 Definition

**useLayoutEffect** is a React Hook similar to useEffect, but it fires synchronously after all DOM mutations, before the browser has a chance to paint. It's useful for measurements and updates that need to happen before the user sees the changes.

---

## 🎯 Purpose & What It Does

### Purpose

- **DOM measurements** - Get element dimensions before rendering
- **Synchronous updates** - Update before visual paint
- **Prevent visual flashing** - Avoid showing incorrect values
- **Layout calculations** - Calculate positions based on element size
- **DOM mutations** - Update DOM before user sees it

### What It Actually Does

1. Runs AFTER DOM updates but BEFORE paint
2. Blocks the browser from painting
3. Runs synchronously (blocking)
4. Use for measurements that affect layout
5. Most of the time use useEffect instead

---

## ⚠️ useEffect vs useLayoutEffect Timing

```
Render Component
         ↓
DOM Updated
         ↓
useLayoutEffect ← Runs HERE (BEFORE paint)
         ↓
Browser Paints (User sees UI)
         ↓
useEffect ← Runs HERE (AFTER paint)
```

---

## 💡 Real-World Use Cases

1. **DOM measurements** - Get width/height before render
2. **Calculating positions** - Position tooltips, popovers
3. **Reading layout values** - Get scroll position, element position
4. **Preventing flashing** - Update before user sees initial render
5. **Animation setup** - Initialize animations before paint

---

## 📝 Hook Examples

### Example 1: Measure Element Size

```javascript
import React, { useRef, useLayoutEffect, useState } from "react";
import { View, Text } from "react-native";

function MeasureElement() {
  const boxRef = useRef(null);
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });

  useLayoutEffect(() => {
    // Measure element dimensions BEFORE paint
    if (boxRef.current) {
      // In React Native, use measure()
      boxRef.current.measure((x, y, width, height) => {
        setDimensions({ width, height });
      });
    }
  }, []);

  return (
    <View style={{ padding: 20 }}>
      <View
        ref={boxRef}
        style={{
          width: 200,
          height: 100,
          backgroundColor: "blue",
          marginBottom: 20,
        }}
      />
      <Text>Width: {dimensions.width}</Text>
      <Text>Height: {dimensions.height}</Text>
    </View>
  );
}

export default MeasureElement;
```

---

### Example 2: Position Tooltip Before Paint

```javascript
import React, { useRef, useLayoutEffect, useState } from "react";
import { View, Text, Button } from "react-native";

function TooltipExample() {
  const targetRef = useRef(null);
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const [showTooltip, setShowTooltip] = useState(false);

  useLayoutEffect(() => {
    if (showTooltip && targetRef.current) {
      // Get target position BEFORE tooltip appears
      targetRef.current.measure((x, y, width, height, pageX, pageY) => {
        setPosition({
          top: pageY + height + 10,
          left: pageX,
        });
      });
    }
  }, [showTooltip]);

  return (
    <View style={{ padding: 20 }}>
      <Button
        ref={targetRef}
        title="Hover for tooltip"
        onPress={() => setShowTooltip(!showTooltip)}
      />

      {showTooltip && (
        <View
          style={{
            position: "absolute",
            top: position.top,
            left: position.left,
            backgroundColor: "black",
            padding: 10,
            borderRadius: 5,
          }}
        >
          <Text style={{ color: "white" }}>Helpful tooltip!</Text>
        </View>
      )}
    </View>
  );
}

export default TooltipExample;
```

---

### Example 3: Prevent Layout Shift

```javascript
import React, { useRef, useLayoutEffect, useState } from "react";
import { View, Text, Button, ActivityIndicator } from "react-native";

function PreventLayoutShift() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const containerRef = useRef(null);

  useLayoutEffect(() => {
    if (loading) {
      // Set container height BEFORE loading state changes
      // This prevents layout shift
      if (containerRef.current) {
        containerRef.current.setNativeProps({
          style: { minHeight: 200 }, // Reserve space
        });
      }
    }
  }, [loading]);

  const handleFetch = async () => {
    setLoading(true);
    try {
      // Simulating: GET https://xyzzy.com/api/data
      const response = await fetch(
        "https://jsonplaceholder.typicode.com/posts/1",
      );
      const result = await response.json();
      setData(result);
    } catch (error) {
      console.error("Failed to fetch:", error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <View ref={containerRef} style={{ padding: 20 }}>
      {loading && <ActivityIndicator />}

      {data && (
        <View>
          <Text style={{ fontSize: 18, fontWeight: "bold" }}>{data.title}</Text>
          <Text>{data.body}</Text>
        </View>
      )}

      {!loading && !data && <Button title="Load Data" onPress={handleFetch} />}
    </View>
  );
}

export default PreventLayoutShift;
```

---

### Example 4: Scroll Position Restoration

```javascript
import React, { useRef, useLayoutEffect, useState } from "react";
import { View, Text, ScrollView, Button, FlatList } from "react-native";

function ScrollRestoration() {
  const scrollRef = useRef(null);
  const [scrollY, setScrollY] = useState(0);
  const [items, setItems] = useState(
    Array.from({ length: 100 }, (_, i) => ({ id: i, text: `Item ${i + 1}` })),
  );

  // Save scroll position before navigation
  const saveScrollPosition = () => {
    if (scrollRef.current) {
      scrollRef.current.measure((x, y, width, height, pageX, pageY) => {
        setScrollY(pageY);
      });
    }
  };

  // Restore scroll position BEFORE paint
  useLayoutEffect(() => {
    if (scrollRef.current && scrollY > 0) {
      scrollRef.current.scrollTo({ y: scrollY, animated: false });
    }
  }, [items]);

  return (
    <View style={{ flex: 1 }}>
      <Button title="Save Scroll Position" onPress={saveScrollPosition} />

      <FlatList
        ref={scrollRef}
        data={items}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={{ padding: 20, borderBottomWidth: 1 }}>
            <Text>{item.text}</Text>
          </View>
        )}
      />
    </View>
  );
}

export default ScrollRestoration;
```

---

### Example 5: Animation Setup

```javascript
import React, { useRef, useLayoutEffect, useState } from "react";
import { View, Text, Button, Animated } from "react-native";

function AnimationSetup() {
  const animationRef = useRef(new Animated.Value(0)).current;
  const [isAnimating, setIsAnimating] = useState(false);

  useLayoutEffect(() => {
    // Setup animation before paint
    if (isAnimating) {
      Animated.timing(animationRef, {
        toValue: 1,
        duration: 1000,
        useNativeDriver: false,
      }).start(() => setIsAnimating(false));
    }
  }, [isAnimating, animationRef]);

  const handleStartAnimation = () => {
    animationRef.setValue(0);
    setIsAnimating(true);
  };

  const animatedStyle = {
    opacity: animationRef,
    transform: [
      {
        scale: animationRef.interpolate({
          inputRange: [0, 1],
          outputRange: [1, 1.5],
        }),
      },
    ],
  };

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Animated.View
        style={[
          {
            width: 100,
            height: 100,
            backgroundColor: "blue",
            marginBottom: 20,
          },
          animatedStyle,
        ]}
      />

      <Button
        title={isAnimating ? "Animating..." : "Start Animation"}
        onPress={handleStartAnimation}
        disabled={isAnimating}
      />
    </View>
  );
}

export default AnimationSetup;
```

---

## ❓ Interview Questions & Answers

### Q1: When should you use useLayoutEffect instead of useEffect?

**Answer:**
Use useLayoutEffect when you need to:

1. Measure DOM elements
2. Prevent visual flashing
3. Read layout values before paint

Use useEffect for almost everything else (data fetching, subscriptions, etc.)

```javascript
// ❌ Wrong - useLayoutEffect for data fetching
useLayoutEffect(() => {
  fetchData(); // Blocks rendering!
}, []);

// ✅ Correct - useEffect for data fetching
useEffect(() => {
  fetchData(); // Doesn't block rendering
}, []);

// ✅ Correct - useLayoutEffect for measurements
useLayoutEffect(() => {
  const rect = ref.current.getBoundingClientRect();
  console.log(rect); // Get position before paint
}, []);
```

---

### Q2: Can useLayoutEffect cause performance issues?

**Answer:**
**Yes**, useLayoutEffect blocks the browser from painting, so it can hurt performance:

```javascript
// ❌ Bad - blocks painting for 500ms
useLayoutEffect(() => {
  // Heavy computation
  for (let i = 0; i < 1000000000; i++) {
    Math.sqrt(i);
  }
}, []);

// ✅ Good - minimal computation before paint
useLayoutEffect(() => {
  // Just get measurements
  const width = ref.current.offsetWidth;
  setState(width);
}, []);
```

**Rule of thumb:** useLayoutEffect should complete in < 16ms to not drop frames.

---

### Q3: What's the difference between useLayoutEffect and useEffect?

**Answer:**
| Aspect | useEffect | useLayoutEffect |
|--------|-----------|-----------------|
| **Timing** | After paint | Before paint |
| **Blocks paint** | No | Yes |
| **Use for** | Most things | Measurements |
| **Performance** | Better | Slower if misused |

---

### Q4: Is useLayoutEffect commonly used?

**Answer:**
**No**, it's rarely needed. Most developers use useEffect 95% of the time:

```javascript
// Common cases where useLayoutEffect might be needed:
1. Tooltip positioning
2. Modal positioning
3. Preventing layout flash
4. Reading scroll position
5. Complex DOM measurements

// Almost always use useEffect for:
1. Data fetching
2. Subscriptions
3. Event listeners
4. Timers
5. Updating state
```

---

## 🔑 Key Takeaways

1. **useLayoutEffect runs before paint** - Blocks rendering
2. **Use sparingly** - Prefer useEffect in most cases
3. **For measurements** - Get DOM values before user sees
4. **Prevent flashing** - Update before visual paint
5. **Performance impact** - Can hurt performance if misused

---

## 🚨 Common Mistakes

```javascript
// ❌ Using for data fetching (blocks paint)
useLayoutEffect(() => {
  fetchData(); // Slow!
}, []);

// ✅ Use useEffect instead
useEffect(() => {
  fetchData();
}, []);

// ❌ Heavy computation (blocks paint)
useLayoutEffect(() => {
  // Loop for 5 seconds - blocks rendering!
  while (Date.now() < start + 5000) {}
}, []);

// ✅ Minimal code in useLayoutEffect
useLayoutEffect(() => {
  // Just get measurements
  const rect = ref.current.getBoundingClientRect();
}, []);
```

---

## 📚 When to Use Each Hook

```javascript
// useEffect - 95% of the time
useEffect(() => {
  fetchData();
  subscribe();
}, []);

// useLayoutEffect - Measurements and preventing flashing
useLayoutEffect(() => {
  const measurements = ref.current.getBoundingClientRect();
}, []);

// useRef - Access DOM without triggering renders
const ref = useRef();

// useState - Track UI state
const [count, setCount] = useState(0);
```

---

## 📚 Practice Exercise

Create a **Responsive Popover Component** that:

1. Shows a popover near a button
2. Measures button position using useLayoutEffect
3. Positions popover above/below button based on space
4. Prevents popover from going off-screen
5. Updates position when window resizes
6. Smooth animation on open/close

---

**Summary of Essential Hooks Complete!** 🎉

You've now learned all 8 essential hooks:

1. ✅ useState
2. ✅ useEffect
3. ✅ useContext
4. ✅ useReducer
5. ✅ useCallback
6. ✅ useMemo
7. ✅ useRef
8. ✅ useLayoutEffect

**Next:** Move on to Advanced/Custom Hooks or State Management Solutions!
