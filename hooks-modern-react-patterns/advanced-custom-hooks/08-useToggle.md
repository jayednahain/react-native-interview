# useToggle Hook - Boolean State Management

## 📌 Definition

**useToggle** is a custom hook that simplifies managing boolean state. Instead of using `useState(false)` and manually toggling, useToggle provides a single hook with toggle, setTrue, and setFalse functions.

---

## 🎯 Purpose & What It Does

- **Simplifies boolean state** - common UI pattern
- **Provides toggle function** - flip boolean
- **Provides setTrue/setFalse** - explicit state setting
- **Reduces boilerplate** - 1 hook vs useState + setState
- **Perfect for modals, dropdowns, switches**

---

## 💡 Real-World Examples

### Example 1: Simple Toggle Modal

```javascript
import { useToggle } from "./hooks/useToggle";
import { View, Text, TouchableOpacity, Modal } from "react-native";

function ModalExample() {
  const [isOpen, toggleModal] = useToggle(false);

  return (
    <View>
      <TouchableOpacity onPress={toggleModal}>
        <Text>Open Modal</Text>
      </TouchableOpacity>

      <Modal visible={isOpen} onRequestClose={toggleModal}>
        <View>
          <Text>Modal Content</Text>
          <TouchableOpacity onPress={toggleModal}>
            <Text>Close</Text>
          </TouchableOpacity>
        </View>
      </Modal>
    </View>
  );
}
```

### Example 2: Toggle with Explicit Setters

```javascript
import { useToggle } from "./hooks/useToggle";
import { View, TouchableOpacity, Text, Switch } from "react-native";

function Settings() {
  const [isNotificationsOn, { toggle, setTrue, setFalse }] = useToggle(true);

  return (
    <View>
      <Text>Notifications</Text>
      <Switch value={isNotificationsOn} onValueChange={toggle} />

      <TouchableOpacity onPress={setTrue}>
        <Text>Enable</Text>
      </TouchableOpacity>

      <TouchableOpacity onPress={setFalse}>
        <Text>Disable</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 3: Multiple Toggles

```javascript
import { useToggle } from "./hooks/useToggle";
import { View, TouchableOpacity, Text } from "react-native";

function FilterPanel() {
  const [showFilters, toggleFilters] = useToggle(false);
  const [showSort, toggleSort] = useToggle(false);
  const [showSearch, toggleSearch] = useToggle(false);

  return (
    <View>
      <TouchableOpacity onPress={toggleFilters}>
        <Text>{showFilters ? "Hide" : "Show"} Filters</Text>
      </TouchableOpacity>
      {showFilters && <View>{/* filters */}</View>}

      <TouchableOpacity onPress={toggleSort}>
        <Text>{showSort ? "Hide" : "Show"} Sort</Text>
      </TouchableOpacity>
      {showSort && <View>{/* sort */}</View>}

      <TouchableOpacity onPress={toggleSearch}>
        <Text>{showSearch ? "Hide" : "Show"} Search</Text>
      </TouchableOpacity>
      {showSearch && <View>{/* search */}</View>}
    </View>
  );
}
```

### Example 4: Loading State Toggle

```javascript
import { useToggle } from "./hooks/useToggle";
import { useState } from "react";

function FormSubmit() {
  const [isLoading, { setTrue: startLoading, setFalse: stopLoading }] =
    useToggle(false);
  const [error, setError] = useState(null);

  const handleSubmit = async (formData) => {
    startLoading();
    try {
      const response = await fetch("/api/data", {
        method: "POST",
        body: JSON.stringify(formData),
      });
      const result = await response.json();
    } catch (err) {
      setError(err.message);
    } finally {
      stopLoading();
    }
  };

  return (
    <View>
      <TouchableOpacity onPress={handleSubmit} disabled={isLoading}>
        <Text>{isLoading ? "Submitting..." : "Submit"}</Text>
      </TouchableOpacity>
      {error && <Text style={{ color: "red" }}>{error}</Text>}
    </View>
  );
}
```

### Example 5: Keyboard Visibility Toggle

```javascript
import { useToggle } from "./hooks/useToggle";
import { useEffect } from "react";
import { Keyboard } from "react-native";

function KeyboardControl() {
  const [isKeyboardVisible, { setTrue, setFalse }] = useToggle(false);

  useEffect(() => {
    const showSubscription = Keyboard.addListener("keyboardDidShow", setTrue);
    const hideSubscription = Keyboard.addListener("keyboardDidHide", setFalse);

    return () => {
      showSubscription.remove();
      hideSubscription.remove();
    };
  }, [setTrue, setFalse]);

  return (
    <Text>{isKeyboardVisible ? "Keyboard visible" : "Keyboard hidden"}</Text>
  );
}
```

---

## ❓ Interview Q&A

**Q1: How do you implement useToggle?**

A: Use useState and return toggle function along with state.

```javascript
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);

  return [
    value,
    {
      toggle: () => setValue(!value),
      setTrue: () => setValue(true),
      setFalse: () => setValue(false),
    },
  ];
}
```

**Q2: Should useToggle return array or object?**

A: Both are valid, but array unpacking is simpler.

```javascript
// Array destructuring (easier)
const [isOpen, { toggle }] = useToggle();

// Object destructuring (more explicit)
const { value: isOpen, toggle } = useToggle();
```

**Q3: How do you combine useToggle with other state?**

A: Just use multiple hooks normally.

```javascript
const [isOpen, { toggle }] = useToggle();
const [count, setCount] = useState(0);

// Use them together
const handleClick = () => {
  toggle();
  setCount(count + 1);
};
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Mutating Return Value

```javascript
// BAD
const [isOpen, handlers] = useToggle();
handlers.toggle(); // Works but not React-like

// GOOD - call like function
const { toggle } = handlers;
onClick = { toggle };
```

---

## 📝 Practice Exercise

**Task:** Create a settings panel with toggles for:

1. Dark mode
2. Notifications
3. Show advanced options
4. Confirm when disabling important setting

**Solution:**

```javascript
import { useToggle } from "./hooks/useToggle";
import { View, TouchableOpacity, Text, Alert, Switch } from "react-native";

function SettingsPanel() {
  const [darkMode, darkModeHandlers] = useToggle(false);
  const [notifications, notifHandlers] = useToggle(true);
  const [advanced, advancedHandlers] = useToggle(false);

  const handleNotificationToggle = () => {
    if (notifications) {
      Alert.alert("Disable Notifications?", "You will miss important updates", [
        { text: "Keep On", onPress: () => {} },
        {
          text: "Disable",
          onPress: () => notifHandlers.setFalse(),
          style: "destructive",
        },
      ]);
    } else {
      notifHandlers.setTrue();
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <View style={{ marginBottom: 15 }}>
        <Text>Dark Mode</Text>
        <Switch value={darkMode} onValueChange={darkModeHandlers.toggle} />
      </View>

      <View style={{ marginBottom: 15 }}>
        <Text>Notifications</Text>
        <Switch
          value={notifications}
          onValueChange={handleNotificationToggle}
        />
      </View>

      <View style={{ marginBottom: 15 }}>
        <Text>Advanced Options</Text>
        <Switch value={advanced} onValueChange={advancedHandlers.toggle} />
      </View>

      {advanced && (
        <View style={{ padding: 10, backgroundColor: "#f0f0f0" }}>
          <Text>Advanced settings go here...</Text>
        </View>
      )}
    </View>
  );
}

export default SettingsPanel;
```

---

## 🎓 Key Takeaways

✅ **Simplifies boolean state** - cleaner than useState  
✅ **Provides toggle, setTrue, setFalse** - explicit control  
✅ **Perfect for modals and switches** - common UI patterns  
✅ **Reduces boilerplate** - 1 hook vs useState + handler

---

**Next:** Learn about useInterval for declarative timers.
