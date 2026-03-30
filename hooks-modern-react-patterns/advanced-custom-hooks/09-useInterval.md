# useInterval Hook - Declarative setInterval

## 📌 Definition

**useInterval** is a custom hook that wraps `setInterval` in a declarative React way. It safely handles cleanup, dependency tracking, and pause/resume functionality without memory leaks.

---

## 🎯 Purpose & What It Does

- **Declarative setInterval** - not imperative like raw setInterval
- **Automatic cleanup** - prevents memory leaks
- **Pause/resume** - stop and restart interval easily
- **Dynamic callbacks** - updates without restart
- **Prevents stale closures** - always runs latest callback

---

## 💡 Real-World Examples

### Example 1: Simple Counter Timer

```javascript
import { useInterval } from "./hooks/useInterval";
import { useState } from "react";
import { View, Text, TouchableOpacity } from "react-native";

function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useInterval(
    () => setSeconds((s) => s + 1),
    isRunning ? 1000 : null, // null pauses interval
  );

  return (
    <View>
      <Text>Seconds: {seconds}</Text>
      <TouchableOpacity onPress={() => setIsRunning(!isRunning)}>
        <Text>{isRunning ? "Stop" : "Start"}</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={() => setSeconds(0)}>
        <Text>Reset</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 2: Polling API Data

```javascript
import { useInterval } from "./hooks/useInterval";
import { useState, useEffect } from "react";
import { View, Text } from "react-native";

function LiveDataFeed() {
  const [data, setData] = useState(null);
  const [isPolling, setIsPolling] = useState(true);

  useInterval(
    async () => {
      const response = await fetch("/api/live-data");
      const newData = await response.json();
      setData(newData);
    },
    isPolling ? 5000 : null, // Poll every 5 seconds
  );

  return (
    <View>
      <Text>Latest: {data?.value}</Text>
      <TouchableOpacity onPress={() => setIsPolling(!isPolling)}>
        <Text>{isPolling ? "Stop Polling" : "Start Polling"}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 3: Countdown Timer

```javascript
import { useInterval } from "./hooks/useInterval";
import { useState } from "react";
import { View, Text, TouchableOpacity } from "react-native";

function Countdown({ initialSeconds = 60 }) {
  const [remaining, setRemaining] = useState(initialSeconds);
  const [isActive, setIsActive] = useState(false);

  useInterval(
    () => {
      setRemaining((r) => {
        if (r <= 1) {
          setIsActive(false);
          return initialSeconds;
        }
        return r - 1;
      });
    },
    isActive ? 1000 : null,
  );

  const minutes = Math.floor(remaining / 60);
  const seconds = remaining % 60;

  return (
    <View>
      <Text style={{ fontSize: 32 }}>
        {minutes}:{seconds.toString().padStart(2, "0")}
      </Text>
      <TouchableOpacity onPress={() => setIsActive(!isActive)}>
        <Text>{isActive ? "Pause" : "Start"}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 4: Animation Loop

```javascript
import { useInterval } from "./hooks/useInterval";
import { useState } from "react";
import { View, Animated } from "react-native";

function AnimatedLoader() {
  const [rotation, setRotation] = useState(0);

  useInterval(() => {
    setRotation((r) => (r + 6) % 360); // 6deg per frame = 60 frames/sec
  }, 16); // ~60fps

  return (
    <View
      style={{
        width: 50,
        height: 50,
        backgroundColor: "blue",
        transform: [{ rotate: `${rotation}deg` }],
      }}
    />
  );
}
```

### Example 5: Health Check with Status Display

```javascript
import { useInterval } from "./hooks/useInterval";
import { useState } from "react";
import { View, Text, TouchableOpacity } from "react-native";

function ServerHealthCheck() {
  const [status, setStatus] = useState("unknown");
  const [lastCheck, setLastCheck] = useState(null);
  const [isMonitoring, setIsMonitoring] = useState(true);

  useInterval(
    async () => {
      try {
        const response = await fetch("/api/health", {
          signal: AbortSignal.timeout(5000),
        });
        setStatus(response.ok ? "online" : "error");
      } catch (err) {
        setStatus("offline");
      }
      setLastCheck(new Date());
    },
    isMonitoring ? 10000 : null, // Check every 10 seconds
  );

  return (
    <View>
      <Text style={{ color: status === "online" ? "green" : "red" }}>
        Status: {status}
      </Text>
      <Text>Last check: {lastCheck?.toLocaleTimeString()}</Text>
      <TouchableOpacity onPress={() => setIsMonitoring(!isMonitoring)}>
        <Text>{isMonitoring ? "Stop Monitoring" : "Start Monitoring"}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## ❓ Interview Q&A

**Q1: How do you implement useInterval?**

A: Use useEffect to set/clear interval and return cleanup function.

```javascript
function useInterval(callback, delay) {
  const savedCallback = useRef();

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    if (delay === null) return;

    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

**Q2: Why pass null for delay to pause?**

A: When delay is null, the effect's cleanup runs without setting interval, effectively pausing.

```javascript
useInterval(
  () => setCount((c) => c + 1),
  isRunning ? 1000 : null, // null = paused
);
```

**Q3: Why use useRef for callback?**

A: Prevents stale closure. Interval always calls latest callback.

```javascript
// Without useRef - calls first callback
setInterval(() => callback(), delay); // stale closure

// With useRef - calls latest callback
useInterval(() => callback(), delay); // always fresh
```

**Q4: How do you stop an interval without null?**

A: Return null for delay to pause, or unmount component.

```javascript
const [isRunning, setIsRunning] = useState(true);
useInterval(() => doSomething(), isRunning ? 1000 : null);

// Stop by setting delay to null
setIsRunning(false);
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Raw setInterval (Causes Memory Leaks)

```javascript
// BAD - memory leak, interval never stops
useEffect(() => {
  setInterval(() => {
    setSeconds((s) => s + 1);
  }, 1000);
}, []);

// GOOD - cleanup on unmount
useInterval(() => {
  setSeconds((s) => s + 1);
}, 1000);
```

### ❌ Mistake 2: Stale Closure Without useRef

```javascript
// BAD - always fires original callback
function useInterval(callback, delay) {
  useEffect(() => {
    const id = setInterval(callback, delay);
    return () => clearInterval(id);
  }, [callback, delay]);
}

// GOOD - always calls latest callback
function useInterval(callback, delay) {
  const savedCallback = useRef();

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

---

## 📝 Practice Exercise

**Task:** Create a stopwatch with:

1. Start/Stop/Reset buttons
2. Display MM:SS:MS format
3. Lap tracking
4. Display all laps

**Solution:**

```javascript
import { useInterval } from "./hooks/useInterval";
import { useState } from "react";
import { View, Text, TouchableOpacity, FlatList } from "react-native";

function Stopwatch() {
  const [time, setTime] = useState(0); // milliseconds
  const [isRunning, setIsRunning] = useState(false);
  const [laps, setLaps] = useState([]);

  useInterval(
    () => {
      setTime((t) => t + 10);
    },
    isRunning ? 10 : null,
  );

  const formatTime = (ms) => {
    const minutes = Math.floor(ms / 60000);
    const seconds = Math.floor((ms % 60000) / 1000);
    const milliseconds = Math.floor((ms % 1000) / 10);

    return `${minutes.toString().padStart(2, "0")}:${seconds
      .toString()
      .padStart(2, "0")}:${milliseconds.toString().padStart(2, "0")}`;
  };

  const handleLap = () => {
    setLaps([{ id: Date.now(), time }, ...laps]);
  };

  const handleReset = () => {
    setTime(0);
    setIsRunning(false);
    setLaps([]);
  };

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 48, textAlign: "center" }}>
        {formatTime(time)}
      </Text>

      <View
        style={{
          flexDirection: "row",
          justifyContent: "space-around",
          marginTop: 20,
        }}
      >
        <TouchableOpacity onPress={() => setIsRunning(!isRunning)}>
          <Text>{isRunning ? "Stop" : "Start"}</Text>
        </TouchableOpacity>

        <TouchableOpacity onPress={handleLap} disabled={!isRunning}>
          <Text>Lap</Text>
        </TouchableOpacity>

        <TouchableOpacity onPress={handleReset}>
          <Text>Reset</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={laps}
        renderItem={({ item, index }) => (
          <Text>
            Lap {index + 1}: {formatTime(item.time)}
          </Text>
        )}
        keyExtractor={(item) => item.id.toString()}
        style={{ marginTop: 20 }}
      />
    </View>
  );
}

export default Stopwatch;
```

---

## 🎓 Key Takeaways

✅ **Declarative setInterval** - React way vs imperative  
✅ **Automatic cleanup** - prevents memory leaks  
✅ **Pause/resume** - pass null to pause  
✅ **Dynamic callbacks** - always uses latest callback  
✅ **Perfect for timers, polling, animations**

---

**Next:** Learn about useKeyboard for keyboard management.
