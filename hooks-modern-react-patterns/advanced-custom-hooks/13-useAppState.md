# useAppState Hook - App Lifecycle Management

## 📌 Definition

**useAppState** (from `@react-native-camera-roll/camera-roll` or native `AppState`) is a hook that tracks the current state of the app (foreground, background, or inactive). It's useful for pausing/resuming timers, syncing data, or saving state when app goes to background.

---

## 🎯 Purpose & What It Does

- **Tracks app state** - active, background, inactive
- **Listens to lifecycle changes** - when app enters/leaves foreground
- **Perfect for cleanup** - pause timers when backgrounded
- **Save state** - persist data before app closes
- **Refresh on resume** - reload data when app comes back

---

## 💡 Real-World Examples

### Example 1: Pause Timer When App Backgrounded

```javascript
import { useAppState } from "@react-native-community/hooks";
import { useState, useEffect } from "react";
import { View, Text, TouchableOpacity } from "react-native";

function BackgroundTimer() {
  const appState = useAppState();
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    if (appState !== "active") return;

    const interval = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);

    return () => clearInterval(interval);
  }, [appState]);

  return (
    <View>
      <Text>Timer: {seconds}s (paused when backgrounded)</Text>
      <Text>App State: {appState}</Text>
    </View>
  );
}
```

### Example 2: Refresh Data When App Resumes

```javascript
import { useAppState } from "@react-native-community/hooks";
import { useEffect, useState } from "react";
import { useQuery, useQueryClient } from "@tanstack/react-query";
import { View, Text } from "react-native";

function RefreshOnResume() {
  const appState = useAppState();
  const queryClient = useQueryClient();

  useEffect(() => {
    if (appState === "active") {
      // Refresh data when app comes to foreground
      queryClient.refetchQueries();
    }
  }, [appState, queryClient]);

  const { data } = useQuery({
    queryKey: ["user-data"],
    queryFn: () => fetch("/api/user").then((r) => r.json()),
  });

  return <Text>Data: {data?.name}</Text>;
}
```

### Example 3: Save State When Backgrounded

```javascript
import { useAppState } from "@react-native-community/hooks";
import { useEffect, useState } from "react";
import { AsyncStorage, View, TextInput } from "react-native";

function AutoSaveForm() {
  const appState = useAppState();
  const [formData, setFormData] = useState("");

  useEffect(() => {
    if (appState === "background") {
      // Save form data when app goes to background
      AsyncStorage.setItem("formData", formData);
    }
  }, [appState, formData]);

  useEffect(() => {
    // Load saved data on mount
    AsyncStorage.getItem("formData").then((saved) => {
      if (saved) setFormData(saved);
    });
  }, []);

  return (
    <TextInput
      value={formData}
      onChangeText={setFormData}
      placeholder="Type something..."
    />
  );
}
```

### Example 4: Custom useAppState Hook

```javascript
import { useState, useEffect } from "react";
import { AppState } from "react-native";

function useAppState() {
  const [appState, setAppState] = useState(AppState.currentState);

  useEffect(() => {
    const subscription = AppState.addEventListener("change", setAppState);
    return () => subscription.remove();
  }, []);

  return appState;
}

// Usage
function MyComponent() {
  const appState = useAppState();
  return <Text>App State: {appState}</Text>;
}
```

### Example 5: Track App Visibility for Analytics

```javascript
import { useAppState } from "@react-native-community/hooks";
import { useEffect, useRef } from "react";

function AnalyticsTracker() {
  const appState = useAppState();
  const sessionStartTime = useRef(Date.now());

  useEffect(() => {
    if (appState === "active") {
      // App just came to foreground
      trackEvent("app_opened");
    } else if (appState === "background") {
      // App about to go to background
      const sessionDuration = Date.now() - sessionStartTime.current;
      trackEvent("app_closed", { duration: sessionDuration });
    }
  }, [appState]);

  const trackEvent = (event, data) => {
    // Send to analytics service
    console.log(`Event: ${event}`, data);
  };

  return null;
}
```

---

## ❓ Interview Q&A

**Q1: What are the different app states?**

A: active, background, inactive

```javascript
useAppState(); // Returns one of:
// 'active' - app is in foreground
// 'background' - app is in background
// 'inactive' - app is transitioning (iOS)
```

**Q2: How do you implement useAppState?**

A: Listen to AppState change events.

```javascript
function useAppState() {
  const [state, setState] = useState(AppState.currentState);

  useEffect(() => {
    const sub = AppState.addEventListener("change", setState);
    return () => sub.remove();
  }, []);

  return state;
}
```

**Q3: When should you save data?**

A: When appState changes from 'active' to 'background'.

```javascript
useEffect(() => {
  if (appState === "background") {
    saveDataToStorage();
  }
}, [appState]);
```

**Q4: How do you detect when app comes to foreground?**

A: Check if appState changed to 'active'.

```javascript
useEffect(() => {
  if (appState === "active") {
    refreshData();
  }
}, [appState]);
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Not Checking for 'active' State

```javascript
// BAD - runs interval in background too
useEffect(() => {
  const id = setInterval(() => doSomething(), 1000);
  return () => clearInterval(id);
}, []);

// GOOD - only runs when active
useEffect(() => {
  if (appState !== "active") return;
  const id = setInterval(() => doSomething(), 1000);
  return () => clearInterval(id);
}, [appState]);
```

### ❌ Mistake 2: Not Cleaning Up Listeners

```javascript
// BAD - memory leak
const subscription = AppState.addEventListener("change", callback);

// GOOD - remove listener
const subscription = AppState.addEventListener("change", callback);
return () => subscription.remove();
```

---

## 📝 Practice Exercise

**Task:** Create a music player that:

1. Pauses when app backgrounded
2. Resumes when app comes to foreground
3. Saves playback position
4. Shows app state

**Solution:**

```javascript
import { useAppState } from "@react-native-community/hooks";
import { useState, useEffect, useRef } from "react";
import { View, Text, TouchableOpacity, AsyncStorage } from "react-native";

function MusicPlayer({ song }) {
  const appState = useAppState();
  const [isPlaying, setIsPlaying] = useState(false);
  const [position, setPosition] = useState(0);
  const positionRef = useRef(position);

  // Update position while playing
  useEffect(() => {
    if (!isPlaying || appState !== "active") return;

    const interval = setInterval(() => {
      setPosition((p) => p + 1);
      positionRef.current = position + 1;
    }, 1000);

    return () => clearInterval(interval);
  }, [isPlaying, appState]);

  // Pause when backgrounded
  useEffect(() => {
    if (appState === "background") {
      setIsPlaying(false);
      // Save position
      AsyncStorage.setItem("musicPosition", positionRef.current.toString());
    } else if (appState === "active") {
      // Optionally resume
      // setIsPlaying(true);
    }
  }, [appState]);

  // Load saved position on mount
  useEffect(() => {
    AsyncStorage.getItem("musicPosition").then((saved) => {
      if (saved) setPosition(parseInt(saved));
    });
  }, []);

  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins}:${secs.toString().padStart(2, "0")}`;
  };

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 18, fontWeight: "bold", marginBottom: 10 }}>
        {song.title}
      </Text>

      <Text style={{ marginBottom: 10 }}>
        App State: {appState} {appState !== "active" && "🔇"}
      </Text>

      <Text style={{ fontSize: 24, marginBottom: 20 }}>
        {formatTime(position)} / {formatTime(song.duration)}
      </Text>

      <TouchableOpacity
        onPress={() => setIsPlaying(!isPlaying)}
        disabled={appState !== "active"}
        style={{ padding: 10, backgroundColor: "blue" }}
      >
        <Text style={{ color: "white" }}>
          {isPlaying ? "⏸ Pause" : "▶️ Play"}
        </Text>
      </TouchableOpacity>

      {appState !== "active" && (
        <Text style={{ marginTop: 10, color: "red" }}>
          Music paused while app in background
        </Text>
      )}
    </View>
  );
}

export default MusicPlayer;
```

---

## 🎓 Key Takeaways

✅ **Tracks app lifecycle** - active, background, inactive  
✅ **Pause on background** - stop expensive operations  
✅ **Resume on foreground** - refresh data  
✅ **Save state** - persist data before backgrounding  
✅ **Analytics** - track user engagement

---

**Next:** Learn about useDimensions for screen dimension tracking.
