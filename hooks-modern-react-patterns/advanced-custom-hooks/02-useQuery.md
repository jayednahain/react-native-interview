# useQuery Hook - Server State Management

## 📌 Definition

**useQuery** (from React Query / TanStack Query) is a hook that manages server state by handling data fetching, caching, synchronization, and updates. It automatically manages request states (loading, error, success) and provides built-in features like caching, background refetching, and stale-while-revalidate patterns.

---

## 🎯 Purpose & What It Does

useQuery solves the problem of managing server state in modern applications:

- **Automatically fetches** data from API endpoints
- **Caches responses** to avoid redundant requests
- **Manages loading/error states** automatically
- **Background refetching** keeps data fresh
- **Automatic retry** on failure
- **Deduplication** - multiple components requesting same data = 1 request
- **Optimistic updates** when combined with useMutation

---

## 🔄 Class Component Equivalent & Reasoning

### Class Component Approach (Old Way)

```javascript
class UserProfile extends React.Component {
  state = {
    user: null,
    loading: true,
    error: null,
  };

  componentDidMount() {
    this.fetchUser();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }

  fetchUser = async () => {
    try {
      this.setState({ loading: true, error: null });
      const response = await fetch(`/api/users/${this.props.userId}`);
      const user = await response.json();
      this.setState({ user, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  };

  render() {
    const { user, loading, error } = this.state;
    if (loading) return <Text>Loading...</Text>;
    if (error) return <Text>Error: {error.message}</Text>;
    return <Text>{user.name}</Text>;
  }
}
```

### Modern Hook Approach (New Way)

```javascript
function UserProfile({ userId }) {
  const {
    data: user,
    isLoading,
    error,
  } = useQuery({
    queryKey: ["user", userId],
    queryFn: () => fetch(`/api/users/${userId}`).then((r) => r.json()),
  });

  if (isLoading) return <Text>Loading...</Text>;
  if (error) return <Text>Error: {error.message}</Text>;
  return <Text>{user.name}</Text>;
}
```

### Why Hooks Are Better

✅ **Automatic refetching** on dependency change (queryKey)  
✅ **Built-in caching** strategy  
✅ **Automatic deduplication** - prevents duplicate requests  
✅ **Less boilerplate** code  
✅ **Background refetching** keeps data fresh  
✅ **Easy retry logic**  
✅ **Devtools integration** for debugging

---

## 💡 Real-World Examples

### Example 1: Simple Data Fetching

```javascript
import { useQuery } from "@tanstack/react-query";
import { View, Text, ActivityIndicator } from "react-native";

function ProductList() {
  const {
    data: products,
    isLoading,
    error,
  } = useQuery({
    queryKey: ["products"],
    queryFn: async () => {
      const response = await fetch("https://api.example.com/products");
      return response.json();
    },
  });

  if (isLoading) return <ActivityIndicator size="large" />;
  if (error) return <Text>Error: {error.message}</Text>;

  return (
    <View>
      {products.map((product) => (
        <Text key={product.id}>
          {product.name} - ${product.price}
        </Text>
      ))}
    </View>
  );
}
```

### Example 2: Dependent Queries with Stale Time

```javascript
import { useQuery } from "@tanstack/react-query";

function UserDetails({ userId }) {
  // Fetch user data - cache for 5 minutes
  const { data: user } = useQuery({
    queryKey: ["user", userId],
    queryFn: () => fetch(`/api/users/${userId}`).then((r) => r.json()),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });

  // Only fetch posts when user exists
  const { data: posts } = useQuery({
    queryKey: ["posts", user?.id],
    queryFn: () => fetch(`/api/users/${user.id}/posts`).then((r) => r.json()),
    enabled: !!user, // Only run when user exists
  });

  if (!user) return <Text>Loading user...</Text>;

  return (
    <View>
      <Text>{user.name}</Text>
      {posts?.map((post) => (
        <Text key={post.id}>{post.title}</Text>
      ))}
    </View>
  );
}
```

### Example 3: Manual Refetch Control

```javascript
import { useQuery } from "@tanstack/react-query";
import { TouchableOpacity, Text } from "react-native";

function PostsList() {
  const {
    data: posts,
    refetch,
    isFetching,
  } = useQuery({
    queryKey: ["posts"],
    queryFn: () => fetch("/api/posts").then((r) => r.json()),
    enabled: false, // Don't auto-fetch
  });

  return (
    <View>
      <TouchableOpacity onPress={() => refetch()}>
        <Text>Refresh Posts {isFetching ? "(Loading...)" : ""}</Text>
      </TouchableOpacity>
      {posts?.map((post) => (
        <Text key={post.id}>{post.title}</Text>
      ))}
    </View>
  );
}
```

### Example 4: Error Boundary with Retry

```javascript
import { useQuery } from "@tanstack/react-query";

function ApiData() {
  const { data, error, isError, isFetching } = useQuery({
    queryKey: ["api-data"],
    queryFn: async () => {
      const response = await fetch("/api/data");
      if (!response.ok) throw new Error("API Error");
      return response.json();
    },
    retry: 3, // Retry 3 times on failure
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  });

  if (isError) {
    return <Text>Error: {error.message} - Please try again</Text>;
  }

  return (
    <View>
      <Text>Status: {isFetching ? "Fetching..." : "Ready"}</Text>
      <Text>{JSON.stringify(data)}</Text>
    </View>
  );
}
```

### Example 5: Query with Search Filter

```javascript
import { useQuery } from "@tanstack/react-query";
import { useState } from "react";
import { TextInput, View } from "react-native";

function SearchUsers() {
  const [search, setSearch] = useState("");

  // Changes queryKey when search changes = new request
  const { data: users } = useQuery({
    queryKey: ["users", "search", search],
    queryFn: () => fetch(`/api/users?search=${search}`).then((r) => r.json()),
    enabled: search.length > 2, // Only search if > 2 chars
  });

  return (
    <View>
      <TextInput
        value={search}
        onChangeText={setSearch}
        placeholder="Search users..."
      />
      {users?.map((user) => (
        <Text key={user.id}>{user.name}</Text>
      ))}
    </View>
  );
}
```

---

## ❓ Interview Q&A

**Q1: What's the difference between useQuery and useState + useEffect?**

A: useQuery provides automatic caching, deduplication, background refetching, and retry logic. With useState + useEffect, you manually handle all of this. useQuery is more declarative and handles edge cases automatically.

```javascript
// With useState + useEffect - you write all this:
useEffect(() => {
  if (!shouldFetch) return;
  let cancelled = false;
  fetchData().then((data) => {
    if (!cancelled) setState(data);
  });
  return () => {
    cancelled = true;
  };
}, [deps]);

// With useQuery - just one hook:
useQuery({ queryKey, queryFn });
```

**Q2: How does the queryKey work and why is it important?**

A: queryKey is an array that uniquely identifies the query. React Query uses it for:

- **Caching** - Same queryKey = same cached data
- **Refetching** - When key changes, automatically refetch
- **Deduplication** - Multiple queries with same key = 1 request
- **Invalidation** - Clear cache when key matches pattern

```javascript
// These are different queries:
useQuery({ queryKey: ['users', 1], ... })
useQuery({ queryKey: ['users', 2], ... })
useQuery({ queryKey: ['users', 1, 'posts'], ... })

// Invalidate all user queries:
queryClient.invalidateQueries({ queryKey: ['users'] })
```

**Q3: What does `enabled: false` do and when would you use it?**

A: `enabled: false` prevents the query from running automatically. Use it when:

- Data depends on user interaction
- Waiting for another query to complete
- Conditional fetching based on state

```javascript
// Only fetch comments when post is loaded
const { data: post } = useQuery({ queryKey: ['post', id], ... });
const { data: comments } = useQuery({
  queryKey: ['comments', post?.id],
  queryFn: () => fetch(`/posts/${post.id}/comments`),
  enabled: !!post, // Don't run until post exists
});
```

**Q4: How does caching work and what's `staleTime`?**

A: React Query keeps fetched data in cache. `staleTime` is how long data is considered fresh:

- **Fresh data** (staleTime not expired) = uses cache without fetching
- **Stale data** (staleTime expired) = uses cache but refetches in background

```javascript
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
  staleTime: 5 * 60 * 1000, // 5 minutes
  gcTime: 10 * 60 * 1000, // Keep in cache 10 min even if unused
});
```

**Q5: What's the difference between isLoading and isFetching?**

A:

- `isLoading` = true only when fetching AND no cached data exists
- `isFetching` = true whenever a request is in flight (even if we have cache)

```javascript
// First load: both true
// Background refetch: only isFetching true
const { isLoading, isFetching } = useQuery({ ... });

// Show spinner only on first load:
{isLoading && <Spinner />}

// Show "updating..." during background fetch:
{isFetching && <Text>Updating...</Text>}
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Creating New Function Reference Every Render

```javascript
// BAD - creates new function every render
useQuery({
  queryKey: ["data"],
  queryFn: () => fetch("/api/data").then((r) => r.json()),
});

// GOOD - memoize function
const fetchData = useCallback(
  () => fetch("/api/data").then((r) => r.json()),
  [],
);
useQuery({ queryKey: ["data"], queryFn: fetchData });
```

### ❌ Mistake 2: Not Including All Dependencies in queryKey

```javascript
// BAD - doesn't refetch when userId changes
useQuery({
  queryKey: ["user"],
  queryFn: () => fetch(`/api/users/${userId}`),
});

// GOOD - includes userId in key
useQuery({
  queryKey: ["user", userId],
  queryFn: () => fetch(`/api/users/${userId}`),
});
```

### ❌ Mistake 3: Not Handling Error States

```javascript
// BAD - ignores errors
const { data } = useQuery({ queryKey: ['data'], queryFn });
return <Text>{data.name}</Text> // Crashes if error

// GOOD - handle error state
const { data, error, isError } = useQuery({ ... });
if (isError) return <Text>Error: {error.message}</Text>;
return <Text>{data.name}</Text>;
```

---

## 📝 Practice Exercise

**Task:** Create a user search component that:

1. Fetches users from `https://api.example.com/users?search=query`
2. Only searches when input has 3+ characters
3. Shows loading state during fetch
4. Displays error messages
5. Has a manual refresh button
6. Caches results for 2 minutes

**Solution:**

```javascript
import { useQuery } from "@tanstack/react-query";
import { useState } from "react";
import {
  View,
  TextInput,
  TouchableOpacity,
  Text,
  ActivityIndicator,
} from "react-native";

function UserSearch() {
  const [search, setSearch] = useState("");

  const {
    data: users,
    isLoading,
    error,
    refetch,
  } = useQuery({
    queryKey: ["users", search],
    queryFn: () =>
      fetch(`https://api.example.com/users?search=${search}`).then((r) =>
        r.json(),
      ),
    enabled: search.length >= 3,
    staleTime: 2 * 60 * 1000,
  });

  return (
    <View style={{ flex: 1, padding: 10 }}>
      <View style={{ flexDirection: "row", marginBottom: 10 }}>
        <TextInput
          style={{ flex: 1, borderWidth: 1, padding: 10, marginRight: 10 }}
          value={search}
          onChangeText={setSearch}
          placeholder="Search users (3+ chars)..."
        />
        <TouchableOpacity onPress={() => refetch()}>
          <Text>Refresh</Text>
        </TouchableOpacity>
      </View>

      {isLoading && <ActivityIndicator size="large" />}
      {error && <Text>Error: {error.message}</Text>}
      {!search.length && <Text>Type to search...</Text>}

      {users?.map((user) => (
        <Text key={user.id}>
          {user.name} ({user.email})
        </Text>
      ))}
    </View>
  );
}

export default UserSearch;
```

---

## 🎓 Key Takeaways

✅ **useQuery manages server state** - fetching, caching, synchronization  
✅ **Automatic deduplication** - same request = 1 network call  
✅ **Built-in caching** - configurable via staleTime and gcTime  
✅ **Smart refetching** - automatic + manual control  
✅ **Error & loading states** - automatically managed  
✅ **Dependency tracking** - queryKey drives updates  
✅ **Dev tools** - React Query Devtools for debugging

---

## 🔗 Related Hooks

- **useMutation** - For POST/PUT/DELETE operations
- **useInfiniteQuery** - For paginated/infinite data
- **useQueries** - Multiple queries at once
- **queryClient.invalidateQueries()** - Force refetch

---

**Next:** Learn about useMutation for handling updates and mutations.
