# useDimensions Hook - Screen Dimensions Tracking

## 📌 Definition

**useDimensions** (from `@react-native-community/hooks`) is a hook that tracks screen dimensions and updates when the screen size changes (device rotation, split-screen mode, etc.). It provides width, height, and orientation information.

---

## 🎯 Purpose & What It Does

- **Tracks screen size** - width and height
- **Updates on rotation** - automatically adjusts to orientation changes
- **Responsive layouts** - build layouts that adapt to any screen
- **Get safe area** - account for notches and safe zones
- **Perfect for responsive design** - one hook for all sizing

---

## 💡 Real-World Examples

### Example 1: Simple Screen Dimensions

```javascript
import { useDimensions } from "@react-native-community/hooks";
import { View, Text } from "react-native";

function ScreenInfo() {
  const { window, screen } = useDimensions();

  return (
    <View>
      <Text>Window Width: {window.width}</Text>
      <Text>Window Height: {window.height}</Text>
      <Text>Screen Width: {screen.width}</Text>
      <Text>Screen Height: {screen.height}</Text>
    </View>
  );
}
```

### Example 2: Responsive Grid Layout

```javascript
import { useDimensions } from "@react-native-community/hooks";
import { View, FlatList, Image } from "react-native";

function ResponsiveGallery({ items }) {
  const { window } = useDimensions();

  const numColumns = window.width > 600 ? 3 : 2;
  const itemSize = window.width / numColumns;

  return (
    <FlatList
      data={items}
      numColumns={numColumns}
      renderItem={({ item }) => (
        <Image
          source={{ uri: item.image }}
          style={{ width: itemSize, height: itemSize }}
        />
      )}
      key={numColumns}
    />
  );
}
```

### Example 3: Center Content Based on Screen

```javascript
import { useDimensions } from "@react-native-community/hooks";
import { View, Text } from "react-native";

function CenteredContent() {
  const { window } = useDimensions();

  return (
    <View
      style={{
        width: window.width,
        height: window.height,
        justifyContent: "center",
        alignItems: "center",
      }}
    >
      <Text>Centered Content</Text>
    </View>
  );
}
```

### Example 4: Custom useDimensions Hook

```javascript
import { useState, useEffect } from "react";
import { useWindowDimensions } from "react-native";

function useDimensions() {
  const window = useWindowDimensions();
  const [screen, setScreen] = useState(window);

  useEffect(() => {
    // In React Native, window and screen are usually the same
    setScreen(window);
  }, [window]);

  return { window, screen };
}

// Usage
function MyComponent() {
  const { window } = useDimensions();
  return (
    <Text>
      {window.width} x {window.height}
    </Text>
  );
}
```

### Example 5: Adapt UI for Different Screen Sizes

```javascript
import { useDimensions } from "@react-native-community/hooks";
import { View, Text, ScrollView } from "react-native";

function AdaptiveLayout() {
  const { window } = useDimensions();

  const isSmallScreen = window.width < 380;
  const isMediumScreen = window.width >= 380 && window.width < 600;
  const isLargeScreen = window.width >= 600;

  return (
    <ScrollView
      style={{ flex: 1, padding: isSmallScreen ? 10 : isLargeScreen ? 40 : 20 }}
    >
      {isSmallScreen && <Text>📱 Small Screen</Text>}
      {isMediumScreen && <Text>📱 Medium Screen</Text>}
      {isLargeScreen && <Text>🖥️ Large Screen</Text>}

      <Text style={{ fontSize: isSmallScreen ? 14 : isLargeScreen ? 20 : 16 }}>
        Responsive Text
      </Text>
    </ScrollView>
  );
}
```

---

## ❓ Interview Q&A

**Q1: What's the difference between window and screen dimensions?**

A: Usually the same, but window excludes notches/safe areas in some cases.

```javascript
const { window, screen } = useDimensions();
// window.width - usable width (excluding notches)
// screen.width - actual screen width
```

**Q2: How do you update on dimension changes?**

A: useWindowDimensions automatically updates on size changes.

```javascript
const dimensions = useWindowDimensions(); // Auto-updates on rotation
```

**Q3: How do you calculate item size for grid?**

A: Divide window width by number of columns.

```javascript
const numColumns = window.width > 600 ? 3 : 2;
const itemSize = window.width / numColumns;
```

---

## 📝 Practice Exercise

**Task:** Create adaptive card layout:

1. 1 column on small screens
2. 2 columns on medium screens
3. 3+ columns on large screens

**Solution:**

```javascript
import { useDimensions } from "@react-native-community/hooks";
import { View, Text, FlatList } from "react-native";

function AdaptiveCards({ items }) {
  const { window } = useDimensions();

  const getNumColumns = () => {
    if (window.width > 900) return 4;
    if (window.width > 600) return 3;
    if (window.width > 380) return 2;
    return 1;
  };

  const numColumns = getNumColumns();
  const cardWidth = window.width / numColumns - 20;

  return (
    <FlatList
      data={items}
      numColumns={numColumns}
      renderItem={({ item }) => (
        <View
          style={{
            width: cardWidth,
            margin: 10,
            padding: 15,
            backgroundColor: "#f0f0f0",
            borderRadius: 10,
          }}
        >
          <Text style={{ fontWeight: "bold" }}>{item.title}</Text>
          <Text>{item.description}</Text>
        </View>
      )}
      keyExtractor={(item) => item.id}
      key={numColumns}
    />
  );
}
```

---

## 🎓 Key Takeaways

✅ **Tracks screen dimensions** - width and height  
✅ **Auto-updates on rotation** - reactive to changes  
✅ **Responsive layouts** - one hook for all sizing  
✅ **Calculate item sizes** - perfect for grids

---

**Next:** Learn about useAnimatedValue for Reanimated animations.
