# useMutation Hook - Handle Server Mutations

## 📌 Definition

**useMutation** (from React Query / TanStack Query) is a hook that handles server mutations (POST, PUT, DELETE operations). Unlike useQuery which fetches data, useMutation manages state changes on the server and provides features like optimistic updates, loading states, and automatic cache invalidation.

---

## 🎯 Purpose & What It Does

useMutation solves mutation problems:

- **Sends data to server** (POST, PUT, DELETE)
- **Manages mutation states** (loading, error, success)
- **Optimistic updates** - update UI before server confirms
- **Rollback on error** - revert UI changes if mutation fails
- **Automatic cache invalidation** - clear related queries
- **Retry logic** with exponential backoff

---

## 🔄 Class Component Equivalent & Reasoning

### Class Component Approach

```javascript
class CreatePost extends React.Component {
  state = {
    title: "",
    loading: false,
    error: null,
  };

  handleSubmit = async () => {
    this.setState({ loading: true, error: null });
    try {
      const response = await fetch("/api/posts", {
        method: "POST",
        body: JSON.stringify({ title: this.state.title }),
      });
      const post = await response.json();
      // Manual cache invalidation needed
      this.props.refetchPosts();
      this.setState({ title: "" });
    } catch (error) {
      this.setState({ error });
    } finally {
      this.setState({ loading: false });
    }
  };

  render() {
    const { loading, error, title } = this.state;
    return (
      <View>
        <TextInput
          value={title}
          onChangeText={(t) => this.setState({ title: t })}
        />
        <TouchableOpacity onPress={this.handleSubmit} disabled={loading}>
          <Text>{loading ? "Creating..." : "Create"}</Text>
        </TouchableOpacity>
        {error && <Text>{error.message}</Text>}
      </View>
    );
  }
}
```

### Modern Hook Approach

```javascript
function CreatePost() {
  const [title, setTitle] = useState("");
  const mutation = useMutation({
    mutationFn: (newPost) =>
      fetch("/api/posts", {
        method: "POST",
        body: JSON.stringify(newPost),
      }).then((r) => r.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] });
      setTitle("");
    },
  });

  return (
    <View>
      <TextInput value={title} onChangeText={setTitle} />
      <TouchableOpacity onPress={() => mutation.mutate({ title })}>
        <Text>{mutation.isPending ? "Creating..." : "Create"}</Text>
      </TouchableOpacity>
      {mutation.isError && <Text>{mutation.error.message}</Text>}
    </View>
  );
}
```

### Why Hooks Are Better

✅ **Automatic cache invalidation** - just specify queryKey  
✅ **Optimistic updates** - update UI immediately  
✅ **Rollback on error** - automatic  
✅ **Less boilerplate** code  
✅ **Better error handling**  
✅ **Easy state management**

---

## 💡 Real-World Examples

### Example 1: Simple Mutation with Cache Invalidation

```javascript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { View, TextInput, TouchableOpacity, Text } from "react-native";
import { useState } from "react";

function AddTodo() {
  const [title, setTitle] = useState("");
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newTodo) =>
      fetch("https://api.example.com/todos", {
        method: "POST",
        body: JSON.stringify(newTodo),
      }).then((r) => r.json()),
    onSuccess: () => {
      // Invalidate queries so they refetch
      queryClient.invalidateQueries({ queryKey: ["todos"] });
      setTitle("");
    },
  });

  return (
    <View>
      <TextInput
        value={title}
        onChangeText={setTitle}
        placeholder="Add a todo..."
      />
      <TouchableOpacity onPress={() => mutation.mutate({ title })}>
        <Text>{mutation.isPending ? "Adding..." : "Add"}</Text>
      </TouchableOpacity>
      {mutation.isError && <Text>Error: {mutation.error.message}</Text>}
    </View>
  );
}
```

### Example 2: Optimistic Updates

```javascript
import { useMutation, useQueryClient } from "@tanstack/react-query";

function UpdateTodo({ todo }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (updated) =>
      fetch(`/api/todos/${todo.id}`, {
        method: "PUT",
        body: JSON.stringify(updated),
      }).then((r) => r.json()),

    // Optimistically update before request
    onMutate: async (newTodo) => {
      // Cancel ongoing queries
      await queryClient.cancelQueries({ queryKey: ["todos"] });

      // Get old data
      const oldData = queryClient.getQueryData(["todos"]);

      // Update UI immediately
      queryClient.setQueryData(["todos"], (old) =>
        old.map((t) => (t.id === todo.id ? newTodo : t)),
      );

      return { oldData }; // Save for rollback
    },

    // Rollback on error
    onError: (err, newTodo, context) => {
      queryClient.setQueryData(["todos"], context.oldData);
    },
  });

  return (
    <TouchableOpacity
      onPress={() => mutation.mutate({ ...todo, completed: true })}
    >
      <Text>{mutation.isPending ? "Updating..." : "Mark Complete"}</Text>
    </TouchableOpacity>
  );
}
```

### Example 3: Update with Success Message

```javascript
import { useMutation } from "@tanstack/react-query";
import { useState } from "react";

function UpdateProfile({ userId }) {
  const [successMessage, setSuccessMessage] = useState("");

  const mutation = useMutation({
    mutationFn: (formData) =>
      fetch(`/api/users/${userId}`, {
        method: "PUT",
        body: JSON.stringify(formData),
      }).then((r) => r.json()),

    onSuccess: (data) => {
      setSuccessMessage("Profile updated successfully!");
      setTimeout(() => setSuccessMessage(""), 3000);
    },
  });

  return (
    <View>
      {successMessage && (
        <Text style={{ color: "green" }}>{successMessage}</Text>
      )}
      <TouchableOpacity onPress={() => mutation.mutate({ name: "John Doe" })}>
        <Text>{mutation.isPending ? "Saving..." : "Update Profile"}</Text>
      </TouchableOpacity>
      {mutation.isError && (
        <Text style={{ color: "red" }}>{mutation.error.message}</Text>
      )}
    </View>
  );
}
```

### Example 4: Delete with Confirmation

```javascript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { Alert, TouchableOpacity, Text } from "react-native";

function DeleteTodo({ todoId }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (id) =>
      fetch(`/api/todos/${id}`, { method: "DELETE" }).then((r) => r.json()),

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["todos"] });
      Alert.alert("Deleted", "Todo was deleted successfully");
    },

    onError: (error) => {
      Alert.alert("Error", error.message);
    },
  });

  const handleDelete = () => {
    Alert.alert("Delete?", "Are you sure?", [
      { text: "Cancel", onPress: () => {} },
      {
        text: "Delete",
        onPress: () => mutation.mutate(todoId),
        style: "destructive",
      },
    ]);
  };

  return (
    <TouchableOpacity onPress={handleDelete}>
      <Text>{mutation.isPending ? "Deleting..." : "Delete"}</Text>
    </TouchableOpacity>
  );
}
```

### Example 5: Mutation with Variables

```javascript
import { useMutation } from "@tanstack/react-query";

function UpdateUserRole() {
  const mutation = useMutation({
    mutationFn: ({ userId, role }) =>
      fetch(`/api/users/${userId}/role`, {
        method: "PUT",
        body: JSON.stringify({ role }),
      }).then((r) => r.json()),
  });

  return (
    <View>
      <TouchableOpacity
        onPress={() => mutation.mutate({ userId: 1, role: "admin" })}
      >
        <Text>Make Admin</Text>
      </TouchableOpacity>
      <TouchableOpacity
        onPress={() => mutation.mutate({ userId: 1, role: "user" })}
      >
        <Text>Make User</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## ❓ Interview Q&A

**Q1: What's the difference between mutation.mutate() and mutation.mutateAsync()?**

A:

- `mutate()` - Fire and forget, use callbacks (onSuccess, onError)
- `mutateAsync()` - Returns promise, use async/await

```javascript
// Using mutate() with callbacks
mutation.mutate(data, {
  onSuccess: () => console.log("Success"),
  onError: (error) => console.error("Error", error),
});

// Using mutateAsync() with promises
try {
  const result = await mutation.mutateAsync(data);
  console.log("Success:", result);
} catch (error) {
  console.error("Error:", error);
}
```

**Q2: How do you implement optimistic updates and why?**

A: Optimistic updates show changes immediately before the server confirms. It makes UI feel faster. You manually update cache, then rollback if the request fails.

```javascript
onMutate: async (newData) => {
  // 1. Cancel any ongoing queries
  await queryClient.cancelQueries({ queryKey: ['items'] });

  // 2. Save old data for rollback
  const oldData = queryClient.getQueryData(['items']);

  // 3. Update UI immediately
  queryClient.setQueryData(['items'], (old) =>
    old.map(item => item.id === newData.id ? newData : item)
  );

  // 4. Return old data for error handling
  return { oldData };
},
onError: (err, newData, context) => {
  // 5. Rollback on error
  queryClient.setQueryData(['items'], context.oldData);
},
```

**Q3: What are the difference between onSuccess, onError, and onSettled?**

A:

- `onSuccess` - Runs when mutation succeeds
- `onError` - Runs when mutation fails
- `onSettled` - Runs regardless (success or error)

```javascript
useMutation({
  mutationFn: updateUser,
  onSuccess: (data) => console.log("User updated:", data),
  onError: (error) => console.error("Update failed:", error),
  onSettled: () => console.log("Request finished"),
});
```

**Q4: How do you handle multiple mutations in sequence?**

A: Use `mutateAsync()` to chain operations or nest callbacks.

```javascript
// Sequential mutations
async function handleCreateAndNotify(userData) {
  try {
    const user = await createUserMutation.mutateAsync(userData);
    await sendEmailMutation.mutateAsync({ userId: user.id });
  } catch (error) {
    console.error("Failed:", error);
  }
}
```

**Q5: How do you retry failed mutations?**

A: Use the `retry` option to automatically retry failed mutations.

```javascript
useMutation({
  mutationFn: updateData,
  retry: 3,
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
});
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Forgetting to Invalidate Related Queries

```javascript
// BAD - data becomes stale
mutation.mutate(newData, {
  onSuccess: () => {
    // Cache not invalidated
  },
});

// GOOD - refresh related data
mutation.mutate(newData, {
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["items"] });
  },
});
```

### ❌ Mistake 2: Not Handling Optimistic Update Rollback

```javascript
// BAD - no rollback on error
onMutate: (newData) => {
  queryClient.setQueryData(['items'], newData);
},

// GOOD - can rollback
onMutate: (newData) => {
  const oldData = queryClient.getQueryData(['items']);
  queryClient.setQueryData(['items'], newData);
  return { oldData };
},
onError: (err, newData, context) => {
  queryClient.setQueryData(['items'], context.oldData);
},
```

### ❌ Mistake 3: Using Wrong State Names

```javascript
// BAD - v4 naming (old)
mutation.isLoading;

// GOOD - v5 naming (current)
mutation.isPending;
```

---

## 📝 Practice Exercise

**Task:** Create a form that:

1. Creates a new user with optimistic update
2. Shows loading state during submission
3. Displays error message if creation fails
4. Rolls back on error
5. Clears form on success
6. Validates form before sending

**Solution:**

```javascript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { useState } from "react";
import { View, TextInput, TouchableOpacity, Text, Alert } from "react-native";

function CreateUserForm() {
  const [form, setForm] = useState({ name: "", email: "" });
  const queryClient = useQueryClient();

  const validateForm = () => {
    if (!form.name.trim()) return "Name required";
    if (!form.email.trim()) return "Email required";
    if (!form.email.includes("@")) return "Invalid email";
    return null;
  };

  const mutation = useMutation({
    mutationFn: (newUser) =>
      fetch("/api/users", {
        method: "POST",
        body: JSON.stringify(newUser),
      }).then((r) => r.json()),

    onMutate: async (newUser) => {
      await queryClient.cancelQueries({ queryKey: ["users"] });
      const oldData = queryClient.getQueryData(["users"]);
      queryClient.setQueryData(["users"], (old) => [...(old || []), newUser]);
      return { oldData };
    },

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["users"] });
      setForm({ name: "", email: "" });
      Alert.alert("Success", "User created!");
    },

    onError: (error, newUser, context) => {
      queryClient.setQueryData(["users"], context.oldData);
      Alert.alert("Error", error.message);
    },
  });

  const handleSubmit = () => {
    const error = validateForm();
    if (error) {
      Alert.alert("Validation Error", error);
      return;
    }
    mutation.mutate(form);
  };

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        value={form.name}
        onChangeText={(name) => setForm({ ...form, name })}
        placeholder="Name"
        style={{ borderWidth: 1, padding: 10, marginBottom: 10 }}
      />
      <TextInput
        value={form.email}
        onChangeText={(email) => setForm({ ...form, email })}
        placeholder="Email"
        style={{ borderWidth: 1, padding: 10, marginBottom: 10 }}
      />
      <TouchableOpacity onPress={handleSubmit} disabled={mutation.isPending}>
        <Text>{mutation.isPending ? "Creating..." : "Create User"}</Text>
      </TouchableOpacity>
    </View>
  );
}

export default CreateUserForm;
```

---

## 🎓 Key Takeaways

✅ **useMutation handles server mutations** - POST, PUT, DELETE  
✅ **Automatic state management** - loading, error, success  
✅ **Optimistic updates** - update UI before server confirms  
✅ **Automatic rollback** - revert changes on error  
✅ **Cache invalidation** - refresh related queries  
✅ **Flexible callbacks** - onSuccess, onError, onSettled  
✅ **Sequential mutations** - chain operations together

---

## 🔗 Related Hooks

- **useQuery** - For fetching server data
- **useInfiniteQuery** - For infinite/paginated mutations
- **queryClient.invalidateQueries()** - Force refresh

---

**Next:** Learn about useInfiniteQuery for handling infinite scrolling and pagination.
