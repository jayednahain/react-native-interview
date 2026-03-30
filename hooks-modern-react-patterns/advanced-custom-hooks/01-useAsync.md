# useAsync Hook - Complete Guide

---

## 📌 Definition

**useAsync** is a custom hook that handles asynchronous operations (promises). It manages loading, error, and success states automatically, providing a clean way to work with async functions.

---

## 🎯 Purpose & What It Does

### Purpose

- **Handle async operations** - Execute async functions and manage state
- **Manage loading/error states** - Automatically track operation status
- **Error handling** - Catch and expose errors cleanly
- **Retry logic** - Easily re-execute async operations
- **Prevent memory leaks** - Clean up on unmount

### What It Actually Does

1. Takes an async function as parameter
2. Optionally executes immediately on mount
3. Tracks loading state while executing
4. Returns result or error when complete
5. Provides execute function to retry

---

## 💡 Real-World Use Cases

1. **API calls** - Fetch data with loading/error states
2. **File uploads** - Track upload progress and status
3. **Authentication** - Login/logout operations
4. **Data processing** - Heavy computations
5. **Batch operations** - Process multiple items
6. **Polling** - Repeated async checks

---

## 📝 Hook Implementation & Examples

### Implementation

```javascript
import { useState, useCallback, useEffect } from "react";

function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState("idle");
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  // Execute the async function
  const execute = useCallback(
    async (...args) => {
      setStatus("pending");
      setValue(null);
      setError(null);
      try {
        const response = await asyncFunction(...args);
        setValue(response);
        setStatus("success");
        return response;
      } catch (error) {
        setError(error);
        setStatus("error");
        throw error;
      }
    },
    [asyncFunction],
  );

  // Execute on mount if immediate is true
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { execute, status, value, error };
}

export default useAsync;
```

---

### Example 1: Basic API Call

```javascript
import React from "react";
import { View, Text, Button, ActivityIndicator } from "react-native";
import useAsync from "./useAsync";

function UserProfile({ userId }) {
  const fetchUser = async () => {
    // Simulating: GET https://xyzzy.com/api/users/{userId}
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/users/${userId}`,
    );
    if (!response.ok) throw new Error("Failed to fetch user");
    return response.json();
  };

  const {
    execute,
    status,
    value: user,
    error,
  } = useAsync(
    fetchUser,
    true, // execute on mount
  );

  if (status === "pending") return <ActivityIndicator size="large" />;
  if (status === "error")
    return <Text style={{ color: "red" }}>{error.message}</Text>;
  if (!user) return <Text>No user found</Text>;

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 18, fontWeight: "bold" }}>{user.name}</Text>
      <Text>{user.email}</Text>
      <Button title="Refresh" onPress={() => execute()} />
    </View>
  );
}

export default UserProfile;
```

---

### Example 2: Manual Execution with Parameters

```javascript
import React, { useState } from "react";
import { View, TextInput, Button, Text, ActivityIndicator } from "react-native";
import useAsync from "./useAsync";

function SearchUsers() {
  const [query, setQuery] = useState("");

  const searchUsers = async (searchQuery) => {
    if (!searchQuery.trim()) return [];
    // Simulating: GET https://xyzzy.com/api/search?q={query}
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/users?name_like=${searchQuery}`,
    );
    if (!response.ok) throw new Error("Search failed");
    return response.json();
  };

  const {
    execute,
    status,
    value: results,
    error,
  } = useAsync(
    searchUsers,
    false, // don't execute on mount
  );

  const handleSearch = () => {
    execute(query);
  };

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        placeholder="Search users..."
        value={query}
        onChangeText={setQuery}
        style={{ borderBottomWidth: 1, marginBottom: 10, padding: 10 }}
      />
      <Button title="Search" onPress={handleSearch} />

      {status === "pending" && <ActivityIndicator />}
      {error && <Text style={{ color: "red" }}>{error.message}</Text>}
      {results && (
        <View style={{ marginTop: 20 }}>
          <Text>{results.length} results found</Text>
          {results.map((user) => (
            <Text key={user.id}>{user.name}</Text>
          ))}
        </View>
      )}
    </View>
  );
}

export default SearchUsers;
```

---

### Example 3: File Upload with Progress

```javascript
import React, { useState } from "react";
import { View, Text, Button, ProgressBarAndroid, Alert } from "react-native";
import useAsync from "./useAsync";

function FileUpload() {
  const [progress, setProgress] = useState(0);

  const uploadFile = async (file) => {
    const formData = new FormData();
    formData.append("file", file);

    // Simulating: POST https://xyzzy.com/api/upload
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();

      // Track upload progress
      xhr.upload.addEventListener("progress", (event) => {
        if (event.lengthComputable) {
          const percentComplete = (event.loaded / event.total) * 100;
          setProgress(percentComplete);
        }
      });

      xhr.addEventListener("load", () => {
        if (xhr.status === 200) {
          resolve(JSON.parse(xhr.responseText));
        } else {
          reject(new Error("Upload failed"));
        }
      });

      xhr.addEventListener("error", () => reject(new Error("Network error")));

      xhr.open("POST", "https://xyzzy.com/api/upload");
      xhr.send(formData);
    });
  };

  const { execute, status, value, error } = useAsync(uploadFile, false);

  const handleUpload = async () => {
    // In real app, use react-native-image-picker or file picker
    const mockFile = { name: "test.jpg", data: "binary" };
    try {
      await execute(mockFile);
      Alert.alert("Success", "File uploaded successfully");
    } catch (err) {
      Alert.alert("Error", err.message);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <Button
        title={status === "pending" ? "Uploading..." : "Upload File"}
        onPress={handleUpload}
        disabled={status === "pending"}
      />

      {status === "pending" && (
        <View style={{ marginTop: 20 }}>
          <ProgressBarAndroid
            styleAttr="Horizontal"
            progress={progress / 100}
          />
          <Text>{Math.round(progress)}%</Text>
        </View>
      )}

      {error && (
        <Text style={{ color: "red", marginTop: 20 }}>{error.message}</Text>
      )}
      {value && (
        <Text style={{ color: "green", marginTop: 20 }}>Upload complete!</Text>
      )}
    </View>
  );
}

export default FileUpload;
```

---

### Example 4: Polling (Repeated Execution)

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, Button, ActivityIndicator } from "react-native";
import useAsync from "./useAsync";

function PollingExample() {
  const [isPolling, setIsPolling] = useState(false);

  const checkStatus = async () => {
    // Simulating: GET https://xyzzy.com/api/status
    const response = await fetch(
      "https://jsonplaceholder.typicode.com/posts/1",
    );
    if (!response.ok) throw new Error("Failed to fetch status");
    return response.json();
  };

  const { execute, status, value, error } = useAsync(checkStatus, false);

  // Set up polling
  useEffect(() => {
    if (!isPolling) return;

    const interval = setInterval(() => {
      execute();
    }, 3000); // Poll every 3 seconds

    return () => clearInterval(interval);
  }, [isPolling, execute]);

  return (
    <View style={{ padding: 20 }}>
      <Button
        title={isPolling ? "Stop Polling" : "Start Polling"}
        onPress={() => setIsPolling(!isPolling)}
      />

      {status === "pending" && <ActivityIndicator />}
      {error && <Text style={{ color: "red" }}>{error.message}</Text>}
      {value && <Text>Status: {value.status || "ok"}</Text>}
    </View>
  );
}

export default PollingExample;
```

---

## ❓ Interview Questions & Answers

### Q1: Why would you create a custom useAsync hook instead of just using useEffect?

**Answer:**
useAsync encapsulates the common pattern of handling async operations with loading/error states:

```javascript
// Without useAsync - repetitive pattern
function Component() {
  const [status, setStatus] = useState("idle");
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(async (fn) => {
    setStatus("pending");
    try {
      const result = await fn();
      setValue(result);
      setStatus("success");
    } catch (err) {
      setError(err);
      setStatus("error");
    }
  }, []);

  return { execute, status, value, error };
}

// With useAsync - cleaner
function Component() {
  const { execute, status, value, error } = useAsync(asyncFn);
}
```

---

### Q2: How do you handle cleanup for async operations?

**Answer:**
Use AbortController to cancel pending requests:

```javascript
function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState("idle");
  const abortControllerRef = useRef(null);

  const execute = useCallback(
    async (...args) => {
      abortControllerRef.current = new AbortController();
      setStatus("pending");
      try {
        const response = await asyncFunction(
          abortControllerRef.current.signal,
          ...args,
        );
        setStatus("success");
        return response;
      } catch (error) {
        if (error.name !== "AbortError") {
          setStatus("error");
        }
      }
    },
    [asyncFunction],
  );

  useEffect(() => {
    return () => {
      abortControllerRef.current?.abort();
    };
  }, []);

  return { execute, status };
}
```

---

### Q3: Should useAsync execute immediately or require manual execution?

**Answer:**
Both patterns are useful:

```javascript
// Immediate execution (auto-fetch on mount)
const { status, value } = useAsync(fetchUser, true);

// Manual execution (user-triggered)
const { execute, status } = useAsync(searchUsers, false);

// Hybrid: execute on dependency change
const { execute, status } = useAsync(fetchData, false);

useEffect(() => {
  execute(userId);
}, [userId, execute]);
```

---

## 🔑 Key Takeaways

1. **useAsync encapsulates async pattern** - DRY principle
2. **Automatic state management** - Loading, error, success
3. **Flexible execution** - Immediate or manual
4. **Retry support** - Execute function for retries
5. **Clean error handling** - Proper error propagation

---

## 📚 Practice Exercise

Create an `useAsync` hook and build:

1. User profile component that fetches on mount
2. Search component with manual execution
3. File upload with progress tracking
4. Polling status checker
5. Proper error handling and cleanup

---

**Next:** Learn about [useFetch Hook](./02-useFetch.md)
