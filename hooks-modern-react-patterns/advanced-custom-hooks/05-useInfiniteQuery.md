# useInfiniteQuery Hook - Infinite Scrolling & Pagination

## 📌 Definition

**useInfiniteQuery** (from React Query / TanStack Query) is a hook designed for paginated and infinite-scrolling data. It handles fetching data in chunks, automatically manages page state, and provides methods to fetch the next/previous page seamlessly.

---

## 🎯 Purpose & What It Does

useInfiniteQuery solves infinite scroll problems:

- **Fetches data in pages** - not all at once
- **Manages pagination state** - current page tracking
- **Smart caching** - each page cached separately
- **Merge pages** - displays all loaded pages together
- **Load more** - easy pagination controls
- **Infinite scroll** - auto-load more on scroll

---

## 🔄 Class Component Equivalent & Reasoning

### Class Component Approach

```javascript
class InfiniteUserList extends React.Component {
  state = {
    users: [],
    page: 1,
    hasMore: true,
    loading: false,
    error: null,
  };

  componentDidMount() {
    this.loadMore();
  }

  loadMore = async () => {
    this.setState({ loading: true });
    try {
      const response = await fetch(`/api/users?page=${this.state.page}`);
      const newUsers = await response.json();
      this.setState({
        users: [...this.state.users, ...newUsers.data],
        page: this.state.page + 1,
        hasMore: newUsers.hasMore,
        loading: false,
      });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  };

  render() {
    return (
      <View>
        <FlatList data={this.state.users} renderItem={...} />
        {this.state.hasMore && (
          <TouchableOpacity onPress={this.loadMore}>
            <Text>{this.state.loading ? 'Loading...' : 'Load More'}</Text>
          </TouchableOpacity>
        )}
      </View>
    );
  }
}
```

### Modern Hook Approach

```javascript
function InfiniteUserList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetching,
  } = useInfiniteQuery({
    queryKey: ['users'],
    queryFn: ({ pageParam = 1 }) =>
      fetch(`/api/users?page=${pageParam}`).then(r => r.json()),
    getNextPageParam: (lastPage) =>
      lastPage.hasMore ? lastPage.nextPage : null,
  });

  const users = data?.pages.flatMap(page => page.data) ?? [];

  return (
    <View>
      <FlatList data={users} renderItem={...} />
      {hasNextPage && (
        <TouchableOpacity onPress={() => fetchNextPage()}>
          <Text>{isFetching ? 'Loading...' : 'Load More'}</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}
```

### Why Hooks Are Better

✅ **Built-in pagination** - no manual page tracking  
✅ **Page merging** - automatically combines pages  
✅ **Smart caching** - each page cached separately  
✅ **Less boilerplate** code  
✅ **Infinite scroll** - easy to implement

---

## 💡 Real-World Examples

### Example 1: Simple Infinite Scroll

```javascript
import { useInfiniteQuery } from "@tanstack/react-query";
import {
  FlatList,
  TouchableOpacity,
  Text,
  ActivityIndicator,
  View,
} from "react-native";

function InfinitePostList() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useInfiniteQuery({
      queryKey: ["posts"],
      queryFn: ({ pageParam = 1 }) =>
        fetch(`https://api.example.com/posts?page=${pageParam}`).then((r) =>
          r.json(),
        ),
      getNextPageParam: (lastPage, pages) =>
        lastPage.hasMore ? pages.length + 1 : null,
    });

  const posts = data?.pages.flatMap((page) => page.posts) ?? [];

  return (
    <View>
      <FlatList
        data={posts}
        renderItem={({ item }) => <Text>{item.title}</Text>}
        keyExtractor={(item) => item.id.toString()}
        onEndReached={() => hasNextPage && fetchNextPage()}
        onEndReachedThreshold={0.5}
        ListFooterComponent={
          isFetchingNextPage ? <ActivityIndicator size="large" /> : null
        }
      />
    </View>
  );
}
```

### Example 2: Pagination with Buttons

```javascript
import { useInfiniteQuery } from "@tanstack/react-query";

function PaginatedUserList() {
  const {
    data,
    fetchNextPage,
    fetchPreviousPage,
    hasNextPage,
    hasPreviousPage,
    isFetching,
  } = useInfiniteQuery({
    queryKey: ["users"],
    queryFn: ({ pageParam = 1 }) =>
      fetch(`/api/users?page=${pageParam}&limit=10`).then((r) => r.json()),
    getNextPageParam: (lastPage) => lastPage.nextPage ?? null,
    getPreviousPageParam: (firstPage) => firstPage.previousPage ?? null,
  });

  const users = data?.pages.flatMap((page) => page.users) ?? [];

  return (
    <View>
      {users.map((user) => (
        <Text key={user.id}>{user.name}</Text>
      ))}

      <View style={{ flexDirection: "row" }}>
        <TouchableOpacity
          onPress={() => fetchPreviousPage()}
          disabled={!hasPreviousPage || isFetching}
        >
          <Text>← Previous</Text>
        </TouchableOpacity>
        <TouchableOpacity
          onPress={() => fetchNextPage()}
          disabled={!hasNextPage || isFetching}
        >
          <Text>Next →</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}
```

### Example 3: With Error Handling

```javascript
import { useInfiniteQuery } from "@tanstack/react-query";
import { useState } from "react";

function RobustInfiniteList() {
  const [retryCount, setRetryCount] = useState(0);

  const {
    data,
    error,
    isError,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ["items"],
    queryFn: ({ pageParam = 1 }) =>
      fetch(`/api/items?page=${pageParam}`).then((r) => {
        if (!r.ok) throw new Error("Failed to load");
        return r.json();
      }),
    getNextPageParam: (lastPage) => lastPage.nextPage,
    retry: 2,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  });

  const items = data?.pages.flatMap((p) => p.items) ?? [];

  return (
    <View>
      {isError && <Text style={{ color: "red" }}>Error: {error.message}</Text>}
      {items.map((item) => (
        <Text key={item.id}>{item.name}</Text>
      ))}
      {hasNextPage && (
        <TouchableOpacity onPress={() => fetchNextPage()}>
          <Text>{isFetchingNextPage ? "Loading..." : "Load More"}</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}
```

### Example 4: Search with Infinite Scroll

```javascript
import { useInfiniteQuery } from "@tanstack/react-query";
import { useState } from "react";

function SearchResults() {
  const [query, setQuery] = useState("");

  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useInfiniteQuery({
      queryKey: ["search", query],
      queryFn: ({ pageParam = 1 }) =>
        fetch(`/api/search?q=${query}&page=${pageParam}`).then((r) => r.json()),
      getNextPageParam: (lastPage) => lastPage.nextPage,
      enabled: query.length > 2, // Only search if > 2 chars
    });

  const results = data?.pages.flatMap((p) => p.results) ?? [];

  return (
    <View>
      <TextInput
        value={query}
        onChangeText={setQuery}
        placeholder="Search..."
      />
      <FlatList
        data={results}
        renderItem={({ item }) => <Text>{item.title}</Text>}
        onEndReached={() => hasNextPage && fetchNextPage()}
        ListFooterComponent={isFetchingNextPage ? <ActivityIndicator /> : null}
      />
    </View>
  );
}
```

### Example 5: Custom Page Merge Logic

```javascript
import { useInfiniteQuery } from "@tanstack/react-query";

function CustomInfiniteList() {
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
    queryKey: ["items"],
    queryFn: ({ pageParam = 0 }) =>
      fetch(`/api/items?offset=${pageParam}&limit=20`).then((r) => r.json()),
    getNextPageParam: (lastPage, allPages) => {
      // Offset-based pagination
      if (lastPage.items.length < 20) return null;
      return allPages.length * 20;
    },
    initialPageParam: 0,
  });

  const allItems = data?.pages.flatMap((page) => page.items) ?? [];

  return (
    <View>
      {allItems.map((item) => (
        <Text key={item.id}>{item.name}</Text>
      ))}
      <TouchableOpacity onPress={() => fetchNextPage()}>
        <Text>{hasNextPage ? "Load More" : "No More Items"}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## ❓ Interview Q&A

**Q1: What's the difference between pageParam and getNextPageParam?**

A:

- `pageParam` - Current page number/offset passed to queryFn
- `getNextPageParam` - Function that calculates the next pageParam

```javascript
useInfiniteQuery({
  queryFn: ({ pageParam = 1 }) => fetch(`/api?page=${pageParam}`),
  getNextPageParam: (lastPage, allPages) => {
    // Calculate next page from lastPage data
    if (lastPage.hasMore) return lastPage.page + 1;
    return null; // No more pages
  },
});
```

**Q2: How do you merge pages from different queries?**

A: Use the `select` option to customize how pages are merged.

```javascript
const { data } = useInfiniteQuery({
  queryKey: ["items"],
  queryFn: ({ pageParam }) => fetch(`/api?page=${pageParam}`),
  select: (data) => ({
    // Custom merge logic
    pages: data.pages.map((p) => p.items).flat(),
    pageParams: data.pageParams,
  }),
});
```

**Q3: How do you handle cursor-based pagination vs offset-based?**

A: Both work, depends on API design:

```javascript
// Offset-based (page number)
getNextPageParam: (lastPage, pages) =>
  pages.length < lastPage.totalPages ? pages.length + 1 : null,

// Cursor-based (using ID cursor)
getNextPageParam: (lastPage) =>
  lastPage.cursor ? lastPage.cursor : null,
```

**Q4: How do you refresh all pages?**

A: Use `queryClient.invalidateQueries()` or `refetch()`.

```javascript
const { refetch } = useInfiniteQuery({ ... });

// Refetch all pages
const handleRefresh = () => refetch();
```

**Q5: What's the difference between isFetching and isFetchingNextPage?**

A:

- `isFetching` - Any fetch including first load
- `isFetchingNextPage` - Only when fetching additional pages

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Not Flattening Pages

```javascript
// BAD - returns nested structure
const { data } = useInfiniteQuery({ ... });
data.pages // [[item1, item2], [item3, item4]]

// GOOD - flatten to single array
const items = data?.pages.flatMap(page => page.items) ?? [];
// [item1, item2, item3, item4]
```

### ❌ Mistake 2: Incorrect getNextPageParam Logic

```javascript
// BAD - always returns next page
getNextPageParam: (lastPage) => lastPage.page + 1,

// GOOD - check if more data exists
getNextPageParam: (lastPage) =>
  lastPage.hasMore ? lastPage.page + 1 : null,
```

### ❌ Mistake 3: Not Handling Empty Results

```javascript
// BAD - crashes if no pages
data.pages.flatMap((p) => p.items);

// GOOD - provide default
data?.pages.flatMap((p) => p.items) ?? [];
```

---

## 📝 Practice Exercise

**Task:** Create an infinite scrolling image gallery that:

1. Loads images 20 at a time
2. Auto-loads more when scrolling near end
3. Shows loading indicator while fetching
4. Displays error if loading fails
5. Has manual refresh button

**Solution:**

```javascript
import { useInfiniteQuery } from "@tanstack/react-query";
import {
  FlatList,
  Image,
  TouchableOpacity,
  Text,
  View,
  ActivityIndicator,
} from "react-native";

function ImageGallery() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    refetch,
    error,
  } = useInfiniteQuery({
    queryKey: ["images"],
    queryFn: ({ pageParam = 1 }) =>
      fetch(`https://api.example.com/images?page=${pageParam}&limit=20`).then(
        (r) => r.json(),
      ),
    getNextPageParam: (lastPage) =>
      lastPage.hasMore ? lastPage.page + 1 : null,
  });

  const images = data?.pages.flatMap((p) => p.images) ?? [];

  return (
    <View style={{ flex: 1 }}>
      <TouchableOpacity onPress={() => refetch()}>
        <Text>Refresh</Text>
      </TouchableOpacity>

      {error && <Text style={{ color: "red" }}>Error: {error.message}</Text>}

      <FlatList
        data={images}
        numColumns={2}
        renderItem={({ item }) => (
          <Image
            source={{ uri: item.url }}
            style={{ width: "50%", height: 200 }}
          />
        )}
        keyExtractor={(item) => item.id.toString()}
        onEndReached={() => hasNextPage && fetchNextPage()}
        onEndReachedThreshold={0.5}
        ListFooterComponent={
          isFetchingNextPage ? <ActivityIndicator size="large" /> : null
        }
      />
    </View>
  );
}

export default ImageGallery;
```

---

## 🎓 Key Takeaways

✅ **useInfiniteQuery handles pagination** - one hook for all pagination  
✅ **Automatic page management** - tracks pages and merges them  
✅ **Smart caching** - each page cached independently  
✅ **Easy infinite scroll** - works with FlatList onEndReached  
✅ **Flexible pagination** - supports offset, cursor, page-based  
✅ **Cursor-based** - efficient for large datasets

---

**Next:** Learn about useThrottle for throttling function calls.
