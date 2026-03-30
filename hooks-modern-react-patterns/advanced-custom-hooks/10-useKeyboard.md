# useKeyboard Hook - Keyboard Management

## 📌 Definition

**useKeyboard** is a custom hook (or library hook from `@react-native-keyboard-aware`) that tracks keyboard visibility and height. It allows you to adjust UI when the keyboard appears/disappears, and provides the keyboard height for layout calculations.

---

## 🎯 Purpose & What It Does

- **Tracks keyboard visibility** - is it showing or not
- **Detects keyboard height** - how much space it takes
- **Listens to show/hide events** - keyboardDidShow, keyboardDidHide
- **Synchronizes layout** - adjust UI for keyboard
- **Works with animations** - animate content when keyboard appears

---

## 💡 Real-World Examples

### Example 1: Adjust View When Keyboard Opens

```javascript
import { useKeyboard } from "@react-native-keyboard-aware";
import { View, TextInput } from "react-native";

function ChatInput() {
  const { keyboardHeight, isKeyboardVisible } = useKeyboard();

  return (
    <View style={{ flex: 1 }}>
      {/* Chat messages */}
      <View style={{ flex: 1 }}>{/* Messages go here */}</View>

      {/* Input area - moves up with keyboard */}
      <View
        style={{
          paddingBottom: isKeyboardVisible ? keyboardHeight : 0,
          backgroundColor: "white",
          padding: 10,
        }}
      >
        <TextInput
          placeholder="Type message..."
          style={{ borderWidth: 1, padding: 10 }}
        />
      </View>
    </View>
  );
}
```

### Example 2: Hide Content When Keyboard Opens

```javascript
import { useKeyboard } from "@react-native-keyboard-aware";
import { View, Text, TextInput, Animated } from "react-native";

function LoginForm() {
  const { isKeyboardVisible } = useKeyboard();

  return (
    <View style={{ flex: 1 }}>
      {!isKeyboardVisible && (
        <View
          style={{ flex: 1, justifyContent: "center", alignItems: "center" }}
        >
          <Text style={{ fontSize: 32, marginBottom: 20 }}>Welcome</Text>
          <Text>App Logo Here</Text>
        </View>
      )}

      <View style={{ padding: 20 }}>
        <TextInput placeholder="Email" />
        <TextInput placeholder="Password" secureTextEntry />
      </View>
    </View>
  );
}
```

### Example 3: Animate Content with Keyboard

```javascript
import { useKeyboard } from "@react-native-keyboard-aware";
import { useEffect, useState } from "react";
import { View, Animated, TextInput } from "react-native";

function AnimatedForm() {
  const { keyboardHeight, isKeyboardVisible } = useKeyboard();
  const [translateY] = useState(new Animated.Value(0));

  useEffect(() => {
    Animated.timing(translateY, {
      toValue: isKeyboardVisible ? -keyboardHeight / 2 : 0,
      duration: 250,
      useNativeDriver: true,
    }).start();
  }, [isKeyboardVisible, keyboardHeight, translateY]);

  return (
    <Animated.View
      style={{
        flex: 1,
        transform: [{ translateY }],
      }}
    >
      <View style={{ flex: 1, justifyContent: "flex-end", padding: 20 }}>
        <TextInput placeholder="Name" />
        <TextInput placeholder="Email" />
      </View>
    </Animated.View>
  );
}
```

### Example 4: Keyboard-Aware ScrollView

```javascript
import { KeyboardAwareScrollView } from "react-native-keyboard-aware-scroll-view";
import { TextInput, View } from "react-native";

function LongForm() {
  return (
    <KeyboardAwareScrollView
      style={{ flex: 1 }}
      contentContainerStyle={{ padding: 20 }}
    >
      <TextInput placeholder="Field 1" />
      <TextInput placeholder="Field 2" />
      <TextInput placeholder="Field 3" />
      {/* ... many more fields ... */}
      <TextInput placeholder="Field 20" />
    </KeyboardAwareScrollView>
  );
}
```

### Example 5: Custom Keyboard Hook Implementation

```javascript
import { useState, useEffect } from "react";
import { Keyboard } from "react-native";

function useKeyboard() {
  const [keyboardHeight, setKeyboardHeight] = useState(0);
  const [isKeyboardVisible, setIsKeyboardVisible] = useState(false);

  useEffect(() => {
    const showListener = Keyboard.addListener("keyboardDidShow", (event) => {
      setKeyboardHeight(event.endCoordinates.height);
      setIsKeyboardVisible(true);
    });

    const hideListener = Keyboard.addListener("keyboardDidHide", () => {
      setKeyboardHeight(0);
      setIsKeyboardVisible(false);
    });

    return () => {
      showListener.remove();
      hideListener.remove();
    };
  }, []);

  return { keyboardHeight, isKeyboardVisible };
}

// Usage
function MyComponent() {
  const { keyboardHeight, isKeyboardVisible } = useKeyboard();
  // Use keyboard state
}
```

---

## ❓ Interview Q&A

**Q1: How do you implement useKeyboard?**

A: Listen to keyboardDidShow/keyboardDidHide events.

```javascript
function useKeyboard() {
  const [keyboardHeight, setKeyboardHeight] = useState(0);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const showListener = Keyboard.addListener("keyboardDidShow", (e) => {
      setKeyboardHeight(e.endCoordinates.height);
      setIsVisible(true);
    });

    const hideListener = Keyboard.addListener("keyboardDidHide", () => {
      setKeyboardHeight(0);
      setIsVisible(false);
    });

    return () => {
      showListener.remove();
      hideListener.remove();
    };
  }, []);

  return { keyboardHeight, isVisible };
}
```

**Q2: What keyboard events are available?**

A: keyboardWillShow, keyboardDidShow, keyboardWillHide, keyboardDidHide

```javascript
// iOS - both will/did events
Keyboard.addListener('keyboardWillShow', ...)
Keyboard.addListener('keyboardDidShow', ...)

// Android - only Did events
Keyboard.addListener('keyboardDidShow', ...)
```

**Q3: How do you programmatically dismiss keyboard?**

A: Use Keyboard.dismiss()

```javascript
import { Keyboard } from "react-native";

const handleBlur = () => {
  Keyboard.dismiss();
};
```

**Q4: When should you use KeyboardAwareScrollView?**

A: When you have a form with many fields that might go below keyboard. It auto-scrolls focused field into view.

```javascript
import { KeyboardAwareScrollView } from "react-native-keyboard-aware-scroll-view";

<KeyboardAwareScrollView>
  {/* Long form with many fields */}
</KeyboardAwareScrollView>;
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Not Removing Listeners

```javascript
// BAD - memory leak
Keyboard.addListener("keyboardDidShow", handler);

// GOOD - remove on unmount
useEffect(() => {
  const listener = Keyboard.addListener("keyboardDidShow", handler);
  return () => listener.remove();
}, []);
```

### ❌ Mistake 2: Using Will Events on Android

```javascript
// BAD - Android doesn't support will events
Keyboard.addListener("keyboardWillShow", handler);

// GOOD - use Did events for cross-platform
Keyboard.addListener("keyboardDidShow", handler);
```

### ❌ Mistake 3: Wrong Keyboard Height Property

```javascript
// BAD
const height = event.endCoordinates.y;

// GOOD
const height = event.endCoordinates.height;
```

---

## 📝 Practice Exercise

**Task:** Create a chat screen that:

1. Shows keyboard height
2. Scrolls to bottom when keyboard opens
3. Animated input field
4. Shows keyboard status

**Solution:**

```javascript
import { useKeyboard } from "./hooks/useKeyboard";
import { useRef, useEffect } from "react";
import {
  View,
  Text,
  TextInput,
  ScrollView,
  Animated,
  FlatList,
} from "react-native";

function ChatScreen() {
  const { keyboardHeight, isKeyboardVisible } = useKeyboard();
  const scrollViewRef = useRef(null);
  const [translateY] = Animated.useState(new Animated.Value(0));

  const messages = [
    { id: 1, text: "Hello!" },
    { id: 2, text: "How are you?" },
    { id: 3, text: "Great, thanks!" },
  ];

  useEffect(() => {
    Animated.timing(translateY, {
      toValue: isKeyboardVisible ? -keyboardHeight / 2 : 0,
      duration: 250,
      useNativeDriver: true,
    }).start();
  }, [isKeyboardVisible, keyboardHeight]);

  useEffect(() => {
    if (isKeyboardVisible && scrollViewRef.current) {
      scrollViewRef.current.scrollToEnd({ animated: true });
    }
  }, [isKeyboardVisible]);

  return (
    <View style={{ flex: 1, backgroundColor: "#fff" }}>
      {/* Status */}
      <View style={{ padding: 10, backgroundColor: "#f0f0f0" }}>
        <Text>Keyboard: {isKeyboardVisible ? "Visible" : "Hidden"}</Text>
        <Text>Height: {keyboardHeight}px</Text>
      </View>

      {/* Messages */}
      <FlatList
        ref={scrollViewRef}
        data={messages}
        renderItem={({ item }) => (
          <View style={{ padding: 10, borderBottomWidth: 1 }}>
            <Text>{item.text}</Text>
          </View>
        )}
        keyExtractor={(item) => item.id.toString()}
      />

      {/* Input - animated with keyboard */}
      <Animated.View
        style={{
          transform: [{ translateY }],
          paddingBottom: isKeyboardVisible ? keyboardHeight : 10,
          padding: 10,
          backgroundColor: "#fff",
          borderTopWidth: 1,
        }}
      >
        <TextInput
          placeholder="Type message..."
          style={{ borderWidth: 1, padding: 10, borderRadius: 20 }}
        />
      </Animated.View>
    </View>
  );
}

export default ChatScreen;
```

---

## 🎓 Key Takeaways

✅ **Tracks keyboard visibility & height** - manage layout  
✅ **Automatic listener cleanup** - prevents memory leaks  
✅ **Works with animations** - smooth keyboard transitions  
✅ **KeyboardAwareScrollView** - auto-scroll to active field  
✅ **Platform differences** - iOS has will/did, Android only did

---

**Next:** Learn about useOrientation for device orientation detection.
