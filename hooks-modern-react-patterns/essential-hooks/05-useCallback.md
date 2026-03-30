# useCallback Hook - Complete Guide

---

## 📌 Definition

**useCallback** is a React Hook that lets you cache a function definition between re-renders. It returns a memoized function that only changes if one of its dependencies has changed.

---

## 🎯 Purpose & What It Does

### Purpose

- **Memoize functions** - Cache function reference across renders
- **Prevent unnecessary re-renders** - Prevent child component re-renders when function prop doesn't change
- **Optimize performance** - Avoid recreating functions unnecessarily
- **Dependency optimization** - Tell React when dependencies actually change

### What It Actually Does

1. Takes a callback function and dependency array
2. Returns a memoized version of the callback
3. Only recreates function if dependencies change
4. Otherwise returns the same function reference
5. Helps child components detect prop changes correctly

---

## 💡 Real-World Use Cases

1. **Pass callbacks to optimized children** - With React.memo
2. **Function dependencies in effects** - useEffect that depends on a function
3. **Event handlers** - Maintain same function reference
4. **Custom hooks** - Return functions that shouldn't recreate
5. **API request functions** - Maintain stable references

---

## 📝 Hook Examples

### Example 1: Basic useCallback

```javascript
import React, { useState, useCallback } from "react";
import { View, Text, Button } from "react-native";

function Counter() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);

  // ❌ Without useCallback - new function every render
  // const increment = () => setCount(count + 1);

  // ✅ With useCallback - same function unless [count] changes
  const increment = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  }, []);

  return (
    <View style={{ padding: 20 }}>
      <Text>Count: {count}</Text>
      <Text>Other: {other}</Text>
      <Button title="Increment Count" onPress={increment} />
      <Button title="Increment Other" onPress={() => setOther(other + 1)} />
    </View>
  );
}

export default Counter;
```

---

### Example 2: useCallback with Dependencies

```javascript
import React, { useState, useCallback } from "react";
import { View, TextInput, Button, FlatList, Text } from "react-native";

function SearchUsers() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);

  // Function recreates ONLY when query changes
  const handleSearch = useCallback(async () => {
    if (!query.trim()) return;

    try {
      // Simulating: GET https://xyzzy.com/api/search?q={query}
      const response = await fetch(
        `https://jsonplaceholder.typicode.com/users?name_like=${query}`,
      );
      const data = await response.json();
      setResults(data);
    } catch (error) {
      console.error("Search failed:", error);
    }
  }, [query]); // Recreate when query changes

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        placeholder="Search users..."
        value={query}
        onChangeText={setQuery}
        style={{ borderBottomWidth: 1, marginBottom: 10, padding: 10 }}
      />
      <Button title="Search" onPress={handleSearch} />

      <FlatList
        data={results}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => <Text>{item.name}</Text>}
      />
    </View>
  );
}

export default SearchUsers;
```

---

### Example 3: useCallback with Child Component Optimization

```javascript
import React, { useState, useCallback, memo } from "react";
import { View, Button, Text } from "react-native";

// Optimized child component - only re-renders if props change
const OptimizedButton = memo(({ label, onClick }) => {
  console.log("OptimizedButton rendered:", label);
  return <Button title={label} onPress={onClick} />;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [color, setColor] = useState("blue");

  // ❌ Without useCallback - creates new function every render
  // const handleIncrement = () => setCount(count + 1);

  // ✅ With useCallback - same function reference
  const handleIncrement = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  }, []);

  const handleChangeColor = useCallback(() => {
    setColor(color === "blue" ? "red" : "blue");
  }, [color]);

  return (
    <View style={{ padding: 20 }}>
      <Text>Count: {count}</Text>

      {/* This component won't re-render if handleIncrement reference stays same */}
      <OptimizedButton label="Increment" onClick={handleIncrement} />

      {/* This component will re-render when handleChangeColor changes */}
      <OptimizedButton label="Change Color" onClick={handleChangeColor} />
    </View>
  );
}

export default Parent;
```

---

### Example 4: useCallback in useEffect Dependency

```javascript
import React, { useState, useCallback, useEffect } from "react";
import { View, Text, ActivityIndicator } from "react-native";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  // Memoize fetch function so it's stable
  const fetchUser = useCallback(async () => {
    setLoading(true);
    try {
      const response = await fetch(
        `https://jsonplaceholder.typicode.com/users/${userId}`,
      );
      const data = await response.json();
      setUser(data);
    } catch (error) {
      console.error("Failed to fetch:", error);
    } finally {
      setLoading(false);
    }
  }, [userId]); // Recreate when userId changes

  // Now we can safely include fetchUser in dependencies
  useEffect(() => {
    fetchUser();
  }, [fetchUser]);

  if (loading) return <ActivityIndicator />;

  return (
    <View style={{ padding: 20 }}>
      <Text>{user?.name}</Text>
      <Text>{user?.email}</Text>
    </View>
  );
}

export default UserProfile;
```

---

### Example 5: Form Handler with useCallback

```javascript
import React, { useState, useCallback } from "react";
import { View, TextInput, Button, Text } from "react-native";

function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  // Memoized form submission
  const handleSubmit = useCallback(async () => {
    setError("");
    setLoading(true);

    try {
      // Simulating API call to https://xyzzy.com/api/login
      const response = await fetch(
        "https://jsonplaceholder.typicode.com/users/1",
        {
          method: "POST",
          body: JSON.stringify({ email, password }),
        },
      );

      if (!response.ok) {
        setError("Invalid credentials");
        return;
      }

      const user = await response.json();
      console.log("Logged in:", user.name);
      // Navigate to home screen
    } catch (err) {
      setError("Login failed. Please try again.");
    } finally {
      setLoading(false);
    }
  }, [email, password]); // Recreate when email or password changes

  const handleEmailChange = useCallback((text) => {
    setEmail(text);
  }, []);

  const handlePasswordChange = useCallback((text) => {
    setPassword(text);
  }, []);

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 20, marginBottom: 20 }}>Login</Text>

      <TextInput
        placeholder="Email"
        value={email}
        onChangeText={handleEmailChange}
        style={{ borderBottomWidth: 1, marginBottom: 15, padding: 10 }}
      />

      <TextInput
        placeholder="Password"
        secureTextEntry
        value={password}
        onChangeText={handlePasswordChange}
        style={{ borderBottomWidth: 1, marginBottom: 20, padding: 10 }}
      />

      {error && <Text style={{ color: "red", marginBottom: 10 }}>{error}</Text>}

      <Button
        title={loading ? "Logging in..." : "Login"}
        onPress={handleSubmit}
        disabled={loading}
      />
    </View>
  );
}

export default LoginForm;
```

---

## ❓ Interview Questions & Answers

### Q1: When should you use useCallback?

**Answer:**
Use useCallback when:

1. You pass a callback as a prop to a memoized child component
2. The callback is a dependency in another hook
3. You're micro-optimizing performance (usually not needed)

```javascript
// ✅ Good use - callback passed to memo'd child
const MyButton = memo(({ onClick }) => <button onClick={onClick} />);
function Parent() {
  const handleClick = useCallback(() => console.log("clicked"), []);
  return <MyButton onClick={handleClick} />;
}

// ❌ Not necessary - callback not used as dependency
function Parent() {
  const handleClick = useCallback(() => console.log("clicked"), []);
  return <button onClick={handleClick} />; // useCallback not needed
}
```

---

### Q2: What's the difference between useCallback and useMemo?

**Answer:**

```javascript
// useCallback - memoizes function
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// useMemo - memoizes computed value
const memoizedValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);

// Equivalent
const memoizedCallback = useCallback(fn, deps);
// is the same as
const memoizedCallback = useMemo(() => fn, deps);
```

---

### Q3: Can useCallback cause infinite loops?

**Answer:**
If used incorrectly in useEffect dependencies:

```javascript
// ❌ Infinite loop - callback depends on count, recreates every render
function Component() {
  const [count, setCount] = useState(0);

  const callback = useCallback(() => {
    setCount(count + 1); // Uses count
  }, [count]); // Depends on count

  useEffect(() => {
    callback(); // Runs on every callback change
  }, [callback]); // Infinite loop!
}

// ✅ Fix - use functional update
function Component() {
  const [count, setCount] = useState(0);

  const callback = useCallback(() => {
    setCount((prev) => prev + 1); // Functional update
  }, []); // No dependencies needed
}
```

---

### Q4: Does useCallback affect performance negatively?

**Answer:**
Not usually, but be aware:

```javascript
// useCallback has overhead:
// 1. Comparing dependencies
// 2. Creating closure
// 3. Keeping function in memory

// Only use when:
// - Passing to memoized components
// - Using as effect dependency
// - Performance is proven to be an issue
```

---

## 🔑 Key Takeaways

1. **useCallback memoizes functions** - Maintains same reference
2. **Use with memoized children** - To prevent unnecessary re-renders
3. **Use in effect dependencies** - Keep functions stable
4. **Not needed for all callbacks** - Only when necessary
5. **Compare costs** - useCallback has overhead too

---

## 📚 Practice Exercise

Create a **Form Component** with:

1. Multiple input fields
2. Form submission handler with useCallback
3. Individual input handlers with useCallback
4. Submit button component that's memoized
5. Log renders to see when components update
6. Compare with/without useCallback

---

**Next:** Learn about [useMemo Hook](./06-useMemo.md)
