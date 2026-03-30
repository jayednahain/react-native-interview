# useSharedValue & useAnimatedStyle - Reanimated Advanced

## 📌 Definition

**useSharedValue** is a Reanimated hook that creates a value that can be safely shared between JS thread and UI (native) thread. **useAnimatedStyle** derives animated styles from shared values. Together they enable performant animations.

---

## 🎯 Purpose & What It Does

- **useSharedValue** - creates thread-safe animated value
- **useAnimatedStyle** - derives styles from shared values
- **worklets** - functions that run on UI thread
- **No JS blocking** - animations run at 60fps independently
- **Synchronous access** - read values without re-renders

---

## 💡 Real-World Examples

### Example 1: Basic Shared Value Animation

```javascript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from "react-native-reanimated";
import { TouchableOpacity, View, Text } from "react-native";

function SharedValueExample() {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePress = () => {
    scale.value = withSpring(scale.value === 1 ? 1.2 : 1);
  };

  return (
    <View>
      <Animated.View
        style={[
          animatedStyle,
          { width: 100, height: 100, backgroundColor: "blue" },
        ]}
      />
      <TouchableOpacity onPress={handlePress}>
        <Text>Scale</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 2: Interpolated Animations

```javascript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  interpolate,
} from "react-native-reanimated";

function InterpolatedColor() {
  const progress = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    backgroundColor: interpolate(
      progress.value,
      [0, 1],
      [0xff0000ff, 0x00ff00ff], // Red to Green
    ),
  }));

  return <Animated.View style={animatedStyle} />;
}
```

### Example 3: Multiple Shared Values

```javascript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
} from "react-native-reanimated";

function MultipleValues() {
  const x = useSharedValue(0);
  const y = useSharedValue(0);
  const opacity = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: x.value }, { translateY: y.value }],
    opacity: opacity.value,
  }));

  return <Animated.View style={animatedStyle} />;
}
```

### Example 4: Worklet with Shared Value

```javascript
import Animated, { useSharedValue, runOnUI } from "react-native-reanimated";

function WorkletAnimation() {
  const value = useSharedValue(0);

  const updateValue = () => {
    runOnUI(() => {
      "worklet"; // Mark as worklet
      value.value += 10; // Run on UI thread
    })();
  };

  return null;
}
```

### Example 5: Gesture-Driven Shared Value

```javascript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  useAnimatedGestureHandler,
} from "react-native-reanimated";
import { PanGestureHandler } from "react-native-gesture-handler";

function GestureDriven() {
  const translationX = useSharedValue(0);

  const gestureHandler = useAnimatedGestureHandler({
    onStart: (_, ctx) => {
      ctx.startX = translationX.value;
    },
    onActive: (event, ctx) => {
      translationX.value = ctx.startX + event.translationX;
    },
  });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translationX.value }],
  }));

  return (
    <PanGestureHandler onGestureEvent={gestureHandler}>
      <Animated.View style={[animatedStyle, { width: 100, height: 100 }]} />
    </PanGestureHandler>
  );
}
```

---

## ❓ Interview Q&A

**Q1: Why use Reanimated instead of React Native Animated?**

A: Reanimated runs on UI thread, doesn't block JS, animations stay smooth even if JS is busy.

```javascript
// React Native Animated - shares JS thread
Animated.timing(value, { duration: 500 }).start();

// Reanimated - runs on UI thread
value.value = withTiming(500);
```

**Q2: What's a worklet and why do you need it?**

A: Worklet is a function that runs on UI thread instead of JS. Marked with 'worklet' string.

```javascript
const handlePress = () => {
  runOnUI(() => {
    "worklet"; // This makes it a worklet
    sharedValue.value = 100; // Runs on UI thread
  })();
};
```

**Q3: How do you read a shared value?**

A: Access .value property. Can be read synchronously without re-renders.

```javascript
const value = useSharedValue(0);

const currentValue = value.value; // Synchronous read
console.log(currentValue); // Works instantly
```

**Q4: How do you animate between values?**

A: Use withTiming, withSpring, or withDecay from Reanimated.

```javascript
value.value = withTiming(100, { duration: 500 });
value.value = withSpring(100);
value.value = withDecay(0.998);
```

---

## 📝 Practice Exercise

**Task:** Create a toggle switch with animated background

**Solution:**

```javascript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  interpolateColor,
} from "react-native-reanimated";
import { TouchableOpacity, View } from "react-native";
import { useState } from "react";

function AnimatedToggleSwitch() {
  const [isOn, setIsOn] = useState(false);
  const progress = useSharedValue(0);

  const animatedSwitchStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: interpolate(progress.value, [0, 1], [0, 20]) }],
  }));

  const animatedContainerStyle = useAnimatedStyle(() => ({
    backgroundColor: interpolateColor(
      progress.value,
      [0, 1],
      ["#ccc", "#4CAF50"],
    ),
  }));

  const handlePress = () => {
    setIsOn(!isOn);
    progress.value = withSpring(isOn ? 0 : 1);
  };

  return (
    <TouchableOpacity onPress={handlePress}>
      <Animated.View
        style={[
          animatedContainerStyle,
          {
            width: 60,
            height: 30,
            borderRadius: 15,
            padding: 2,
            justifyContent: "center",
          },
        ]}
      >
        <Animated.View
          style={[
            animatedSwitchStyle,
            {
              width: 26,
              height: 26,
              borderRadius: 13,
              backgroundColor: "white",
            },
          ]}
        />
      </Animated.View>
    </TouchableOpacity>
  );
}

export default AnimatedToggleSwitch;
```

---

## 🎓 Key Takeaways

✅ **useSharedValue** - thread-safe animated values  
✅ **useAnimatedStyle** - derive styles from shared values  
✅ **Worklets** - run code on UI thread  
✅ **No JS blocking** - smooth 60fps animations  
✅ **Perfect for gestures** - gesture-driven animations

---

**All Advanced Hooks Complete! 🎉**
