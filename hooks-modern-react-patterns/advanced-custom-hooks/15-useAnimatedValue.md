# useAnimatedValue Hook - Reanimated Animations (Worklet)

## 📌 Definition

**useAnimatedValue** (from `react-native-reanimated`) is a hook for creating animated values that run on the native thread using Reanimated. Animations run at 60fps without blocking the JS thread, enabling smooth high-performance animations.

---

## 🎯 Purpose & What It Does

- **Native thread animations** - smooth 60fps animations
- **Doesn't block JS** - UI stays responsive
- **Gesture-driven** - perfect with gesture handlers
- **Worklets** - run code on native thread
- **Declarative animations** - easier than Animated API

---

## 💡 Real-World Examples

### Example 1: Simple Animated Counter

```javascript
import Animated, {
  useAnimatedStyle,
  withSpring,
} from "react-native-reanimated";
import { useSharedValue } from "react-native-reanimated";
import { View, TouchableOpacity, Text } from "react-native";

function AnimatedCounter() {
  const counter = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: counter.value }],
  }));

  const handlePress = () => {
    counter.value = withSpring(counter.value + 1);
  };

  return (
    <View>
      <Animated.View
        style={[
          { width: 100, height: 100, backgroundColor: "blue" },
          animatedStyle,
        ]}
      />
      <TouchableOpacity onPress={handlePress}>
        <Text>Animate</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 2: Gesture-Driven Animation

```javascript
import Animated, {
  useAnimatedStyle,
  useAnimatedGestureHandler,
} from "react-native-reanimated";
import { useSharedValue } from "react-native-reanimated";
import { PanGestureHandler } from "react-native-gesture-handler";

function DraggableBox() {
  const x = useSharedValue(0);
  const y = useSharedValue(0);

  const gestureHandler = useAnimatedGestureHandler({
    onStart: (_, ctx) => {
      ctx.startX = x.value;
      ctx.startY = y.value;
    },
    onActive: (event, ctx) => {
      x.value = ctx.startX + event.translationX;
      y.value = ctx.startY + event.translationY;
    },
  });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: x.value }, { translateY: y.value }],
  }));

  return (
    <PanGestureHandler onGestureEvent={gestureHandler}>
      <Animated.View
        style={[
          animatedStyle,
          { width: 100, height: 100, backgroundColor: "blue" },
        ]}
      />
    </PanGestureHandler>
  );
}
```

### Example 3: Scroll-Driven Animation

```javascript
import Animated, {
  useAnimatedStyle,
  useAnimatedScrollHandler,
} from "react-native-reanimated";
import { useSharedValue } from "react-native-reanimated";
import { ScrollView } from "react-native";

function ScrollAnimation({ items }) {
  const scrollY = useSharedValue(0);

  const scrollHandler = useAnimatedScrollHandler({
    onScroll: (event) => {
      scrollY.value = event.contentOffset.y;
    },
  });

  const animatedHeaderStyle = useAnimatedStyle(() => ({
    opacity: Math.min(1, scrollY.value / 100),
    height: Math.max(50, 150 - scrollY.value),
  }));

  return (
    <>
      <Animated.View style={animatedHeaderStyle}>
        {/* Header that animates on scroll */}
      </Animated.View>

      <Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16}>
        {/* Content */}
      </Animated.ScrollView>
    </>
  );
}
```

### Example 4: Shared Value Worklet

```javascript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  runOnUI,
} from "react-native-reanimated";

function WorkletExample() {
  const sharedValue = useSharedValue(0);

  const handlePress = () => {
    // Run on UI thread
    runOnUI(() => {
      "worklet"; // Mark as worklet
      sharedValue.value = Math.random() * 100;
    })();
  };

  const animatedStyle = useAnimatedStyle(() => ({
    width: sharedValue.value,
  }));

  return (
    <Animated.View
      style={[{ height: 50, backgroundColor: "blue" }, animatedStyle]}
    />
  );
}
```

### Example 5: Interpolation

```javascript
import Animated, {
  useAnimatedStyle,
  interpolate,
} from "react-native-reanimated";
import { useSharedValue } from "react-native-reanimated";

function InterpolatedAnimation() {
  const progress = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    backgroundColor: interpolate(
      progress.value,
      [0, 1],
      ["#FF0000", "#00FF00"],
    ),
    transform: [
      {
        rotate: interpolate(progress.value, [0, 1], [0, 360]) + "deg",
      },
    ],
  }));

  return <Animated.View style={animatedStyle} />;
}
```

---

## ❓ Interview Q&A

**Q1: What's the difference between Reanimated and React Native's Animated?**

A: Reanimated runs on UI thread (60fps), Animated shares JS thread (60fps but can jank).

```javascript
// React Native Animated (can block JS)
Animated.timing(value, { duration: 500 }).start();

// Reanimated (runs on UI thread)
value.value = withTiming(100);
```

**Q2: What's a worklet?**

A: A function marked with 'worklet' that runs on the native/UI thread instead of JS thread.

```javascript
const handlePress = () => {
  runOnUI(() => {
    "worklet"; // Marks function as worklet
    sharedValue.value = 100;
  })();
};
```

**Q3: How do you animate between values?**

A: Use withTiming, withSpring, or withDecay.

```javascript
value.value = withTiming(100); // Linear animation
value.value = withSpring(100); // Spring animation
value.value = withDecay(50); // Decay animation
```

---

## 📝 Practice Exercise

**Task:** Create a toggle button with animated rotation

**Solution:**

```javascript
import Animated, {
  useAnimatedStyle,
  withSpring,
} from "react-native-reanimated";
import { useSharedValue } from "react-native-reanimated";
import { TouchableOpacity, Text, View } from "react-native";

function AnimatedToggle() {
  const rotation = useSharedValue(0);
  const [isOn, setIsOn] = useState(false);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ rotate: `${rotation.value}deg` }],
  }));

  const handlePress = () => {
    setIsOn(!isOn);
    rotation.value = withSpring(isOn ? 0 : 180);
  };

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Animated.View
        style={[
          animatedStyle,
          { width: 100, height: 100, backgroundColor: isOn ? "green" : "red" },
        ]}
      />
      <TouchableOpacity onPress={handlePress} style={{ marginTop: 20 }}>
        <Text>{isOn ? "ON" : "OFF"}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 🎓 Key Takeaways

✅ **Native thread animations** - 60fps smooth  
✅ **Doesn't block JS** - responsive UI  
✅ **Worklets** - code runs on UI thread  
✅ **Shared values** - communication between JS and UI

---

**Next:** Learn about useForm for form state management with React Hook Form.
