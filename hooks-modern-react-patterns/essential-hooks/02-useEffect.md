# useEffect Hook - Complete Guide

---

## 📌 Definition

**useEffect** is a React Hook that lets you perform side effects in functional components. Side effects are actions that interact with the outside world (fetching data, subscribing to events, modifying the DOM, setting timers, etc.).

---

## 🎯 Purpose & What It Does

### Purpose

- **Run side effects** - Execute code after render (API calls, subscriptions, logging)
- **Replace lifecycle methods** - Combines componentDidMount, componentDidUpdate, and componentWillUnmount
- **Control timing** - Run effects at specific times using dependency arrays
- **Cleanup resources** - Prevent memory leaks by cleaning up subscriptions/timers

### What It Actually Does

1. Accepts a callback function that runs AFTER the component renders
2. The callback runs AFTER the DOM has been updated
3. Can optionally return a cleanup function
4. Uses a dependency array to control when the effect runs

---

## 🔄 Class Component vs Hooks Comparison

### Class Component (Old Way)

```javascript
class UserFeed extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [],
      loading: true,
    };
  }

  // Runs once when component mounts
  componentDidMount() {
    this.fetchUsers();
    this.unsubscribe = this.subscribeToUpdates();
  }

  // Runs every time component updates
  componentDidUpdate(prevProps, prevState) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUsers();
    }
  }

  // Cleanup: Runs before component unmounts
  componentWillUnmount() {
    this.unsubscribe();
  }

  fetchUsers = async () => {
    try {
      const response = await fetch(`/api/users/${this.props.userId}`);
      const data = await response.json();
      this.setState({ users: data, loading: false });
    } catch (error) {
      console.error(error);
    }
  };

  subscribeToUpdates = () => {
    // Subscribe logic
    return () => {
      // Unsubscribe logic
    };
  };

  render() {
    return <div>{/* render logic */}</div>;
  }
}
```

### Hooks Version (Modern Way)

```javascript
function UserFeed({ userId }) {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  // Replaces componentDidMount + componentDidUpdate + componentWillUnmount
  useEffect(() => {
    let unsubscribe;

    const fetchUsers = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUsers(data);
        setLoading(false);
      } catch (error) {
        console.error(error);
      }
    };

    fetchUsers();
    unsubscribe = subscribeToUpdates();

    // Cleanup function (like componentWillUnmount)
    return () => {
      unsubscribe();
    };
  }, [userId]); // Dependency array (like componentDidUpdate)

  return <div>{/* render logic */}</div>;
}
```

### Why We Switched to Hooks

| Aspect                | Class Component                  | useEffect Hook                     |
| --------------------- | -------------------------------- | ---------------------------------- |
| **Code organization** | Split across 3 lifecycle methods | Grouped by feature in one effect   |
| **Related logic**     | Scattered in different methods   | Together in one callback           |
| **Cleanup**           | In separate componentWillUnmount | In return statement of same effect |
| **Readability**       | Hard to follow related logic     | Easy to see cause and effect       |
| **Reusability**       | Can't easily reuse logic         | Extract into custom hooks          |

---

## 🎨 The Dependency Array Explained

The dependency array controls **when** the effect runs:

```javascript
// ❌ No dependency array - runs EVERY RENDER
useEffect(() => {
  console.log("This runs after EVERY render");
});

// ✅ Empty array [] - runs ONCE (on mount)
useEffect(() => {
  console.log("This runs only once when component mounts");
}, []);

// ✅ With dependencies [count, name] - runs when count OR name changes
useEffect(() => {
  console.log("This runs when count or name changes");
}, [count, name]);
```

---

## 💡 Real-World Use Cases

1. **Fetching Data** - Load data when component mounts
2. **Subscriptions** - Subscribe to database updates
3. **Event Listeners** - Add/remove window event listeners
4. **Timers** - Set intervals and timeouts
5. **Analytics** - Track user actions
6. **LocalStorage** - Save/load data
7. **Polling** - Periodic data fetching

---

## 📝 Hook Examples

### Example 1: Fetch Data on Mount

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, ActivityIndicator, FlatList } from "react-native";

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Runs once when component mounts (like componentDidMount)
  useEffect(() => {
    const fetchUsers = async () => {
      try {
        // Simulating: GET https://xyzzy.com/api/users
        const response = await fetch(
          "https://jsonplaceholder.typicode.com/users",
        );
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError("Failed to fetch users");
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []); // Empty dependency = run once on mount

  if (loading) return <ActivityIndicator size="large" />;
  if (error) return <Text style={{ color: "red" }}>{error}</Text>;

  return (
    <FlatList
      data={users}
      keyExtractor={(item) => item.id.toString()}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
}

export default UserList;
```

---

### Example 2: Dependency Array - Refetch When Props Change

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, ActivityIndicator } from "react-native";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        // Simulating: GET https://xyzzy.com/api/users/{userId}
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`,
        );
        const data = await response.json();
        setUser(data);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]); // Re-run when userId changes

  if (loading) return <ActivityIndicator />;

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 20, fontWeight: "bold" }}>{user?.name}</Text>
      <Text>{user?.email}</Text>
      <Text>{user?.phone}</Text>
    </View>
  );
}

export default UserProfile;
```

---

### Example 3: Cleanup Function - Unsubscribe/Remove Listeners

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, Dimensions } from "react-native";

function ResponsiveComponent() {
  const [windowWidth, setWindowWidth] = useState(
    Dimensions.get("window").width,
  );

  useEffect(() => {
    // Subscribe to dimension changes
    const subscription = Dimensions.addEventListener("change", ({ window }) => {
      setWindowWidth(window.width);
    });

    // Cleanup function (like componentWillUnmount)
    return () => {
      subscription?.remove();
    };
  }, []);

  return (
    <View>
      <Text>Window Width: {windowWidth}</Text>
    </View>
  );
}

export default ResponsiveComponent;
```

---

### Example 4: Timer with Cleanup

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, Button } from "react-native";

function StopWatch() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    // Don't run timer if not running
    if (!isRunning) return;

    // Set up interval
    const intervalId = setInterval(() => {
      setSeconds((prevSeconds) => prevSeconds + 1);
    }, 1000);

    // Cleanup: Clear interval when component unmounts or isRunning changes
    return () => clearInterval(intervalId);
  }, [isRunning]); // Re-run when isRunning changes

  return (
    <View style={{ alignItems: "center", padding: 20 }}>
      <Text style={{ fontSize: 32 }}>{seconds}s</Text>
      <Button
        title={isRunning ? "Stop" : "Start"}
        onPress={() => setIsRunning(!isRunning)}
      />
      <Button
        title="Reset"
        onPress={() => {
          setSeconds(0);
          setIsRunning(false);
        }}
      />
    </View>
  );
}

export default StopWatch;
```

---

### Example 5: Multiple Effects - Different Concerns

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, TextInput, AsyncStorage } from "react-native";

function UserPreferences({ userId }) {
  const [theme, setTheme] = useState("light");
  const [notifications, setNotifications] = useState(true);

  // Effect 1: Fetch user preferences on mount or userId change
  useEffect(() => {
    const fetchPreferences = async () => {
      try {
        const response = await fetch(
          `https://xyzzy.com/api/users/${userId}/preferences`,
        );
        const data = await response.json();
        setTheme(data.theme);
        setNotifications(data.notifications);
      } catch (error) {
        console.error("Failed to fetch preferences:", error);
      }
    };

    fetchPreferences();
  }, [userId]);

  // Effect 2: Save theme to local storage when theme changes
  useEffect(() => {
    AsyncStorage.setItem("userTheme", theme);
  }, [theme]);

  // Effect 3: Save notifications to local storage when notifications change
  useEffect(() => {
    AsyncStorage.setItem("userNotifications", notifications.toString());
  }, [notifications]);

  return (
    <View style={{ padding: 20 }}>
      <Text>Current Theme: {theme}</Text>
      <Text>Notifications: {notifications ? "On" : "Off"}</Text>
    </View>
  );
}

export default UserPreferences;
```

---

### Example 6: API Call with Abort (Cancel Request)

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, ActivityIndicator } from "react-native";

function CancellableUserFetch({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Create abort controller to cancel fetch if component unmounts
    const abortController = new AbortController();

    const fetchUser = async () => {
      try {
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`,
          { signal: abortController.signal },
        );
        const data = await response.json();
        setUser(data); // Only set if request succeeded
        setLoading(false);
      } catch (error) {
        if (error.name !== "AbortError") {
          console.error("Failed to fetch:", error);
        }
      }
    };

    fetchUser();

    // Cleanup: Abort the fetch if component unmounts
    return () => abortController.abort();
  }, [userId]);

  if (loading) return <ActivityIndicator />;

  return (
    <View style={{ padding: 20 }}>
      <Text>{user?.name}</Text>
      <Text>{user?.email}</Text>
    </View>
  );
}

export default CancellableUserFetch;
```

---

## ❓ Interview Questions & Answers

### Q1: What is the difference between useEffect and componentDidMount/componentDidUpdate?

**Answer:**

```javascript
// componentDidMount (once on mount)
componentDidMount() {
  console.log('Component mounted');
}
// Equivalent in useEffect:
useEffect(() => {
  console.log('Component mounted');
}, []); // Empty dependency array

// componentDidUpdate (every update, with condition)
componentDidUpdate(prevProps) {
  if (prevProps.id !== this.props.id) {
    console.log('ID changed');
  }
}
// Equivalent in useEffect:
useEffect(() => {
  console.log('ID changed');
}, [id]); // Dependency array with specific variable
```

---

### Q2: Why do we need a cleanup function in useEffect?

**Answer:**
The cleanup function prevents memory leaks and manages resources properly:

```javascript
useEffect(() => {
  // Setup
  const subscription = someService.subscribe((data) => {
    setData(data);
  });

  // Cleanup - runs before next effect or unmount
  return () => {
    subscription.unsubscribe(); // Prevent memory leak
  };
}, []);
```

**Common cleanup scenarios:**

- Unsubscribe from subscriptions
- Clear intervals/timeouts
- Remove event listeners
- Cancel pending API requests

---

### Q3: What happens if you don't include a dependency in the dependency array?

**Answer:**
The effect won't re-run when that variable changes:

```javascript
const [count, setCount] = useState(0);

// ❌ Missing 'count' in dependency array
useEffect(() => {
  console.log("Count is:", count); // Always logs 0 (stale closure)
}, []); // 'count' should be here

// ✅ Correct
useEffect(() => {
  console.log("Count is:", count); // Logs updated value
}, [count]); // Include all dependencies
```

This is called a **stale closure** - the effect uses old values.

---

### Q4: When should you use useEffect vs useState?

**Answer:**

- **useState** - For storing component state (UI data)
- **useEffect** - For side effects (API calls, subscriptions, etc.)

```javascript
function SearchUsers({ query }) {
  const [results, setResults] = useState([]); // State - what to display

  useEffect(() => {
    // Side effect - fetch data when query changes
    const fetchResults = async () => {
      const data = await fetch(`/api/search?q=${query}`);
      setResults(await data.json());
    };
    fetchResults();
  }, [query]);

  return <div>{/* render results */}</div>;
}
```

---

### Q5: Is it safe to ignore the ESLint warning about missing dependencies?

**Answer:**
**NO!** Always follow the exhaustive-deps rule. Ignoring it causes bugs:

```javascript
const [count, setCount] = useState(0);

// ❌ DON'T DO THIS - missing dependency
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1); // Always increments from 0
  }, 1000);
  return () => clearInterval(timer);
}, []); // Missing 'count'

// ✅ CORRECT
useEffect(() => {
  const timer = setInterval(() => {
    setCount((prevCount) => prevCount + 1); // Use functional update
  }, 1000);
  return () => clearInterval(timer);
}, []); // OR add [count] and use functional updates
```

---

### Q6: What is the purpose of returning a function from useEffect?

**Answer:**
The returned function is a cleanup function that runs:

1. Before the effect runs again
2. Before the component unmounts

```javascript
useEffect(() => {
  const unsubscribe = firebaseDB.onSnapshot((data) => {
    setData(data);
  });

  // Cleanup function
  return () => {
    unsubscribe(); // Prevent memory leak
  };
}, []);
```

---

## 🔑 Key Takeaways

1. **useEffect runs after render** - After DOM is painted
2. **Dependencies control when it runs** - [] = once, [deps] = when deps change, no array = every render
3. **Cleanup prevents memory leaks** - Return cleanup function for subscriptions/timers
4. **Similar to 3 lifecycle methods** - Combines componentDidMount, componentDidUpdate, componentWillUnmount
5. **Use functional updates** - For state that depends on previous value in effects

---

## 🚨 Common Mistakes

```javascript
// ❌ Infinite loop - no dependency array
useEffect(() => {
  setCount(count + 1); // Triggers re-render, effect runs again
});

// ❌ Stale closure - missing dependency
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count); // Always 0 even if count changes
  }, 1000);
  return () => clearInterval(timer);
}, []);

// ✅ Correct approaches
useEffect(() => {
  console.log(count);
}, [count]); // Re-run when count changes

useEffect(() => {
  setCount((prev) => prev + 1); // Use functional update
}, []); // No dependency needed
```

---

## 📚 Practice Exercise

Create a component that:

1. Has a search input field
2. Fetches matching users from an API when the input changes
3. Displays loading state while fetching
4. Shows error message if fetch fails
5. Displays results in a list
6. Cancels the previous request if a new one is made

**Hint:** Use useEffect with dependency array, cleanup function, and AbortController.

---

**Next:** Learn about [useContext Hook](./03-useContext.md)
