# useOrientation Hook - Device Orientation Detection

## 📌 Definition

**useOrientation** is a custom hook (or from `react-native-orientation-locker`) that detects and tracks device orientation (portrait/landscape). It listens to orientation changes and provides current orientation state with the ability to lock/unlock orientation.

---

## 🎯 Purpose & What It Does

- **Tracks orientation** - portrait, landscape, portrait-upside-down
- **Listens to changes** - updates when device rotates
- **Lock/unlock orientation** - force specific orientation
- **Get screen dimensions** - width, height in current orientation
- **Perfect for responsive layouts** - adapt UI to orientation

---

## 💡 Real-World Examples

### Example 1: Simple Orientation Detection

```javascript
import { useOrientation } from "@react-native-orientation-locker";
import { View, Text } from "react-native";

function OrientationAware() {
  const { orientation } = useOrientation();

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Text style={{ fontSize: 24 }}>Current: {orientation}</Text>
      <Text>
        {orientation === "PORTRAIT" ? "Phone is upright" : "Phone is sideways"}
      </Text>
    </View>
  );
}
```

### Example 2: Responsive Layout Based on Orientation

```javascript
import { useOrientation } from "@react-native-orientation-locker";
import { View, Text, FlatList, useWindowDimensions } from "react-native";

function ResponsiveGallery({ images }) {
  const { orientation } = useOrientation();
  const { width } = useWindowDimensions();

  const numColumns = orientation === "PORTRAIT" ? 2 : 4;

  return (
    <FlatList
      data={images}
      numColumns={numColumns}
      renderItem={({ item }) => (
        <View style={{ width: width / numColumns, height: 200 }}>
          <Image source={{ uri: item.url }} style={{ flex: 1 }} />
        </View>
      )}
      key={numColumns}
      keyExtractor={(item) => item.id}
    />
  );
}
```

### Example 3: Lock Orientation for Video

```javascript
import { useOrientation } from "@react-native-orientation-locker";
import { useEffect } from "react";
import { View } from "react-native";
import Video from "react-native-video";

function VideoPlayer() {
  const { lockOrientation, unlockOrientation } = useOrientation();

  useEffect(() => {
    // Lock to landscape when playing video
    lockOrientation("LANDSCAPE");

    return () => {
      // Unlock when leaving
      unlockOrientation();
    };
  }, []);

  return (
    <View style={{ flex: 1 }}>
      <Video
        source={{ uri: "https://example.com/video.mp4" }}
        style={{ flex: 1 }}
        resizeMode="contain"
      />
    </View>
  );
}
```

### Example 4: Custom Orientation Hook

```javascript
import { useState, useEffect } from "react";
import { useWindowDimensions } from "react-native";

function useOrientation() {
  const { width, height } = useWindowDimensions();
  const [orientation, setOrientation] = useState("PORTRAIT");

  useEffect(() => {
    const isPortrait = height > width;
    setOrientation(isPortrait ? "PORTRAIT" : "LANDSCAPE");
  }, [width, height]);

  return {
    orientation,
    isPortrait: orientation === "PORTRAIT",
    isLandscape: orientation === "LANDSCAPE",
    width,
    height,
  };
}

// Usage
function MyComponent() {
  const { orientation, isPortrait } = useOrientation();
  return <Text>{isPortrait ? "Portrait" : "Landscape"}</Text>;
}
```

### Example 5: Conditional Features Based on Orientation

```javascript
import { useOrientation } from "@react-native-orientation-locker";
import { View, Text, Image } from "react-native";

function ProductDetail({ product }) {
  const { orientation } = useOrientation();
  const isPortrait = orientation === "PORTRAIT";

  return (
    <View style={{ flex: 1, padding: 10 }}>
      {isPortrait ? (
        // Portrait: Image on top, details below
        <>
          <Image
            source={{ uri: product.image }}
            style={{ width: "100%", height: 300 }}
          />
          <View style={{ flex: 1 }}>
            <Text>{product.name}</Text>
            <Text>{product.description}</Text>
            <Text>${product.price}</Text>
          </View>
        </>
      ) : (
        // Landscape: Image left, details right
        <View style={{ flexDirection: "row" }}>
          <Image
            source={{ uri: product.image }}
            style={{ width: "50%", height: "100%" }}
          />
          <View style={{ flex: 1, padding: 10 }}>
            <Text>{product.name}</Text>
            <Text>{product.description}</Text>
            <Text>${product.price}</Text>
          </View>
        </View>
      )}
    </View>
  );
}
```

---

## ❓ Interview Q&A

**Q1: How do you implement useOrientation?**

A: Use useWindowDimensions to detect orientation changes.

```javascript
function useOrientation() {
  const { width, height } = useWindowDimensions();
  return width > height ? "LANDSCAPE" : "PORTRAIT";
}
```

**Q2: How do you lock orientation?**

A: Use react-native-orientation-locker or native modules.

```javascript
import Orientation from "react-native-orientation-locker";

// Lock to portrait
Orientation.lockToPortrait();

// Lock to landscape
Orientation.lockToLandscape();

// Unlock
Orientation.unlockAllOrientations();
```

**Q3: What's the difference between orientation and screen dimensions?**

A: Orientation is just portrait/landscape, dimensions include actual width/height values.

```javascript
const orientation = "PORTRAIT"; // just a label
const { width, height } = useWindowDimensions(); // actual dimensions
```

**Q4: How do you prevent rotation for specific screens?**

A: Lock orientation when mounting, unlock when unmounting.

```javascript
useEffect(() => {
  Orientation.lockToPortrait();
  return () => Orientation.unlockAllOrientations();
}, []);
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Not Unlocking Orientation

```javascript
// BAD - orientation stays locked after leaving screen
useEffect(() => {
  Orientation.lockToLandscape();
}, []);

// GOOD - unlock on cleanup
useEffect(() => {
  Orientation.lockToLandscape();
  return () => Orientation.unlockAllOrientations();
}, []);
```

### ❌ Mistake 2: Using height > width (Can Fail on Tablets)

```javascript
// BAD - tablet in portrait might be wider than tall
const isPortrait = height > width;

// GOOD - explicitly state orientation
const { orientation } = Orientation.getOrientation();
```

---

## 📝 Practice Exercise

**Task:** Create a document viewer that:

1. Shows portrait/landscape orientation
2. Locks to landscape when in full-screen mode
3. Adapts layout based on orientation
4. Shows toggle to enter/exit full-screen

**Solution:**

```javascript
import { useOrientation } from "@react-native-orientation-locker";
import { useState, useEffect } from "react";
import {
  View,
  Text,
  TouchableOpacity,
  useWindowDimensions,
} from "react-native";

function DocumentViewer({ document }) {
  const { orientation, lockOrientation, unlockAllOrientations } =
    useOrientation();
  const [isFullScreen, setIsFullScreen] = useState(false);
  const { width, height } = useWindowDimensions();

  useEffect(() => {
    if (isFullScreen) {
      lockOrientation("LANDSCAPE");
    } else {
      unlockAllOrientations();
    }

    return () => unlockAllOrientations();
  }, [isFullScreen]);

  const isPortrait = orientation === "PORTRAIT";

  return (
    <View style={{ flex: 1, padding: isFullScreen ? 0 : 10 }}>
      {/* Header */}
      {!isFullScreen && (
        <View style={{ marginBottom: 10 }}>
          <Text style={{ fontSize: 18, fontWeight: "bold" }}>
            {document.title}
          </Text>
          <Text>Orientation: {orientation}</Text>
        </View>
      )}

      {/* Document Content */}
      <View
        style={{
          flex: 1,
          backgroundColor: "#f5f5f5",
          justifyContent: "center",
          alignItems: "center",
        }}
      >
        <Text>{document.content}</Text>
        <Text style={{ marginTop: 10 }}>
          {isPortrait ? "📱 Portrait" : "🔄 Landscape"}
        </Text>
      </View>

      {/* Controls */}
      {!isFullScreen && (
        <TouchableOpacity
          onPress={() => setIsFullScreen(true)}
          style={{ padding: 10, backgroundColor: "blue", marginTop: 10 }}
        >
          <Text style={{ color: "white" }}>Full Screen</Text>
        </TouchableOpacity>
      )}

      {isFullScreen && (
        <TouchableOpacity
          onPress={() => setIsFullScreen(false)}
          style={{ padding: 10, backgroundColor: "red" }}
        >
          <Text style={{ color: "white" }}>Exit Full Screen</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}

export default DocumentViewer;
```

---

## 🎓 Key Takeaways

✅ **Tracks device orientation** - portrait vs landscape  
✅ **Responsive layouts** - adapt UI based on orientation  
✅ **Lock/unlock orientation** - force specific orientation  
✅ **Works with dimensions** - get actual width/height  
✅ **Perfect for full-screen mode** - video players, documents

---

**Next:** Learn about useNetInfo for network connectivity detection.
