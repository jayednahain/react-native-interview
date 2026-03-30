# useDebounce Hook - Complete Guide

---

## 📌 Definition

**useDebounce** is a custom hook that delays updating a value until after a specified time has passed since the value was last changed. Useful for search inputs, API calls, and auto-save features.

---

## 🎯 Purpose & What It Does

### Purpose

- **Reduce API calls** - Don't call API on every keystroke
- **Improve performance** - Delay expensive operations
- **Better UX** - Fewer server requests, smoother experience
- **Auto-save** - Wait for user to stop typing before saving
- **Search optimization** - Debounce search input

### What It Actually Does

1. Takes a value and delay time
2. Returns debounced version of value
3. Updates only after delay passes since last change
4. Cancels pending update if value changes again
5. Useful with useEffect dependencies

---

## 💡 Real-World Use Cases

1. **Search inputs** - Wait before searching
2. **Auto-save** - Save form after user stops typing
3. **API calls** - Throttle API requests
4. **Window resize** - Handle resize events efficiently
5. **Input validation** - Validate after user stops typing
6. **Analytics** - Track user input without spam

---

## 📝 Hook Implementation & Examples

### Implementation

```javascript
import { useState, useEffect } from "react";

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    // Set up timeout
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Cleanup: cancel timeout if value changes before delay
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

export default useDebounce;
```

---

### Example 1: Search with Debounce

```javascript
import React, { useState } from "react";
import {
  View,
  TextInput,
  FlatList,
  Text,
  ActivityIndicator,
} from "react-native";
import useDebounce from "./useDebounce";

function SearchUsers() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [searchCount, setSearchCount] = useState(0);

  // Debounce the query - only search after user stops typing for 500ms
  const debouncedQuery = useDebounce(query, 500);

  // API call only happens when debounced query changes
  React.useEffect(() => {
    if (!debouncedQuery.trim()) {
      setResults([]);
      return;
    }

    const fetchResults = async () => {
      setLoading(true);
      setSearchCount((prev) => prev + 1); // Track API calls
      try {
        // Simulating: GET https://xyzzy.com/api/search?q={query}
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users?name_like=${debouncedQuery}`,
        );
        const data = await response.json();
        setResults(data);
      } catch (error) {
        console.error("Search failed:", error);
      } finally {
        setLoading(false);
      }
    };

    fetchResults();
  }, [debouncedQuery]);

  return (
    <View style={{ padding: 20, flex: 1 }}>
      <TextInput
        placeholder="Search users..."
        value={query}
        onChangeText={setQuery}
        style={{ borderBottomWidth: 1, marginBottom: 10, padding: 10 }}
      />
      <Text style={{ fontSize: 12, color: "#666", marginBottom: 10 }}>
        API calls: {searchCount} (without debounce: would be {query.length})
      </Text>

      {loading && <ActivityIndicator />}

      <FlatList
        data={results}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={{ padding: 10, borderBottomWidth: 1 }}>
            <Text>{item.name}</Text>
            <Text style={{ fontSize: 12, color: "#666" }}>{item.email}</Text>
          </View>
        )}
      />
    </View>
  );
}

export default SearchUsers;
```

---

### Example 2: Auto-Save Form

```javascript
import React, { useState } from "react";
import { View, TextInput, Text } from "react-native";
import useDebounce from "./useDebounce";

function AutoSaveForm() {
  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");
  const [saveStatus, setSaveStatus] = useState("unsaved"); // unsaved, saving, saved

  // Debounce both fields
  const debouncedTitle = useDebounce(title, 1000);
  const debouncedContent = useDebounce(content, 1000);

  // Auto-save when debounced values change
  React.useEffect(() => {
    if (!debouncedTitle && !debouncedContent) return;

    const saveToServer = async () => {
      setSaveStatus("saving");
      try {
        // Simulating: POST https://xyzzy.com/api/posts
        await fetch("https://jsonplaceholder.typicode.com/posts", {
          method: "POST",
          body: JSON.stringify({
            title: debouncedTitle,
            body: debouncedContent,
          }),
        });
        setSaveStatus("saved");

        // Reset to unsaved after 2 seconds
        setTimeout(() => setSaveStatus("unsaved"), 2000);
      } catch (error) {
        setSaveStatus("error");
        console.error("Save failed:", error);
      }
    };

    saveToServer();
  }, [debouncedTitle, debouncedContent]);

  const statusColor = {
    unsaved: "#FF9800",
    saving: "#2196F3",
    saved: "#4CAF50",
    error: "#F44336",
  };

  return (
    <View style={{ padding: 20 }}>
      <View style={{ marginBottom: 10 }}>
        <Text>
          Status:
          <Text style={{ color: statusColor[saveStatus], fontWeight: "bold" }}>
            {" " + saveStatus}
          </Text>
        </Text>
      </View>

      <Text style={{ fontWeight: "bold", marginBottom: 5 }}>Title</Text>
      <TextInput
        value={title}
        onChangeText={setTitle}
        placeholder="Post title..."
        style={{ borderBottomWidth: 1, marginBottom: 15, padding: 10 }}
      />

      <Text style={{ fontWeight: "bold", marginBottom: 5 }}>Content</Text>
      <TextInput
        value={content}
        onChangeText={setContent}
        placeholder="Post content..."
        multiline
        numberOfLines={5}
        style={{
          borderWidth: 1,
          padding: 10,
          marginBottom: 15,
          textAlignVertical: "top",
        }}
      />

      <Text style={{ fontSize: 12, color: "#666" }}>
        * Auto-saves 1 second after you stop typing
      </Text>
    </View>
  );
}

export default AutoSaveForm;
```

---

### Example 3: Input Validation with Debounce

```javascript
import React, { useState } from "react";
import { View, TextInput, Text } from "react-native";
import useDebounce from "./useDebounce";

function EmailValidator() {
  const [email, setEmail] = useState("");
  const [validationStatus, setValidationStatus] = useState(""); // checking, valid, invalid
  const [error, setError] = useState("");

  const debouncedEmail = useDebounce(email, 500);

  React.useEffect(() => {
    if (!debouncedEmail) {
      setValidationStatus("");
      return;
    }

    const validateEmail = async () => {
      setValidationStatus("checking");
      try {
        // Simulating: GET https://xyzzy.com/api/validate-email?email={email}
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/1`,
        );
        const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(debouncedEmail);

        if (isValid) {
          setValidationStatus("valid");
          setError("");
        } else {
          setValidationStatus("invalid");
          setError("Invalid email format");
        }
      } catch (error) {
        setValidationStatus("error");
        setError("Validation failed");
      }
    };

    validateEmail();
  }, [debouncedEmail]);

  const borderColor = {
    "": "#ccc",
    checking: "#2196F3",
    valid: "#4CAF50",
    invalid: "#F44336",
    error: "#F44336",
  };

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontWeight: "bold", marginBottom: 5 }}>Email</Text>
      <TextInput
        value={email}
        onChangeText={setEmail}
        placeholder="Enter email..."
        style={{
          borderBottomWidth: 2,
          borderBottomColor: borderColor[validationStatus],
          marginBottom: 10,
          padding: 10,
        }}
      />

      {validationStatus === "checking" && (
        <Text style={{ color: "#2196F3" }}>Validating...</Text>
      )}
      {validationStatus === "valid" && (
        <Text style={{ color: "#4CAF50" }}>✓ Email is valid</Text>
      )}
      {error && <Text style={{ color: "#F44336" }}>{error}</Text>}
    </View>
  );
}

export default EmailValidator;
```

---

## ❓ Interview Questions & Answers

### Q1: What's the difference between debounce and throttle?

**Answer:**

```javascript
// Debounce - wait until user stops (0 -> -> -> FIRE after delay)
// Good for: search, auto-save, validation

// Throttle - fire at regular intervals (0 -> FIRE -> -> FIRE -> FIRE)
// Good for: scroll events, resize, mouse move

// Example: User types 10 characters
// Debounce: 1 API call (after user stops)
// Throttle: 3-4 API calls (at intervals)
```

---

### Q2: How do you cancel a pending debounce?

**Answer:**
The cleanup function automatically cancels:

```javascript
useEffect(() => {
  const handler = setTimeout(() => {
    setDebouncedValue(value);
  }, delay);

  // Cleanup cancels the timeout
  return () => clearTimeout(handler);
}, [value, delay]);
```

---

### Q3: Can you combine debounce with search parameters?

**Answer:**
**Yes**, use both in useEffect:

```javascript
function SearchWithFilters() {
  const [query, setQuery] = useState("");
  const [category, setCategory] = useState("all");

  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    // Search is triggered when debounced query OR category changes
    if (debouncedQuery || category !== "all") {
      performSearch(debouncedQuery, category);
    }
  }, [debouncedQuery, category]);
}
```

---

## 🔑 Key Takeaways

1. **Debounce delays execution** - Waits after value stops changing
2. **Perfect for search** - Reduce API calls on typing
3. **Auto-save friendly** - Wait for user to finish
4. **Cleanup cancels** - Automatically managed by useEffect
5. **Customizable delay** - Adjust based on use case

---

## 📚 Practice Exercise

Create a `useDebounce` hook and build:

1. Search input with live results
2. Auto-save form that saves after user stops typing
3. Email validation that checks after delay
4. Resize event handler
5. Display API call count to show optimization

---

**Next:** Learn about [useToggle Hook](./06-useToggle.md)
