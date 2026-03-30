# 🔥 React Query (TanStack Query) - Server State Management

## Overview

**React Query** is NOT a general state management library.  
**React Query** = The BEST solution for managing SERVER STATE (API data).

Key insight: **Most apps treat server state same as client state. HUGE MISTAKE.**

---

## Simple Definition

**React Query** = Library that handles fetching, caching, synchronizing, and updating server state with near-zero boilerplate.

It's the answer to: "How do I manage API data properly?"

---

## The Server State Problem

### Problem 1: Duplicate API Calls

```javascript
// ❌ BAD: Calling same API multiple times
function ProductsPage() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch("/api/products")
      .then((r) => r.json())
      .then(setProducts);
  }, []);

  return <ProductList products={products} />;
}

function ProductHeader() {
  const [productCount, setProductCount] = useState(0);

  useEffect(() => {
    fetch("/api/products")
      .then((r) => r.json())
      .then((p) => setProductCount(p.length));
  }, []);

  return <Header count={productCount} />;
}

// Both components fetch same data! Waste, redundant requests 😫
```

### Problem 2: Stale Data

```javascript
// ❌ BAD: Data becomes stale
function Dashboard() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch("/api/data")
      .then((r) => r.json())
      .then(setData);
  }, []);

  // What if server data changed?
  // Component still shows old data!
  // How do you know when to refetch?
  // Manual polling? 🔄 Gross!
}
```

### Problem 3: Race Conditions

```javascript
// ❌ BAD: Race conditions
function SearchProducts({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    fetch(`/api/search?q=${query}`)
      .then((r) => r.json())
      .then(setResults);
  }, [query]);

  // RACE CONDITION:
  // User types 'l' → fetch for 'l'
  // User types 'la' → fetch for 'la'
  // 'la' request responds first
  // 'l' request responds second → overwrites 'la' with 'l' results!
  // 😱 Shows wrong results!
}
```

### Problem 4: Loading & Error States

```javascript
// ❌ BAD: Manual state management
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [retryCount, setRetryCount] = useState(0);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then((user) => {
        setUser(user);
        setLoading(false);
        setError(null);
      })
      .catch((err) => {
        setError(err);
        setLoading(false);
      });
  }, [userId, retryCount]);

  // 😫 So much state for one thing!
  // What about retry logic? Exponential backoff? Timeout?
}
```

---

## React Query Solution

### How It Solves All Problems

```javascript
// ✅ REACT QUERY: Simple and powerful
import { useQuery } from "@tanstack/react-query";

function ProductsPage() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["products"],
    queryFn: () => fetch("/api/products").then((r) => r.json()),
  });

  return <ProductList products={data} />;
}

function ProductHeader() {
  const { data } = useQuery({
    queryKey: ["products"],
    queryFn: () => fetch("/api/products").then((r) => r.json()),
  });

  // MAGIC: Both components call this
  // But React Query recognizes same queryKey
  // Deduplicates request!
  // Only ONE fetch happens! ✅
  return <Header count={data?.length} />;
}

// And it handles:
// ✅ Deduplication
// ✅ Caching
// ✅ Background refetch
// ✅ Stale data detection
// ✅ Error handling
// ✅ Retry logic
// ✅ Race condition prevention
// All automatically! 🎉
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────┐
│      React Query Cache                      │
│  ┌───────────────────────────────────────┐  │
│  │ queryKey: ['products']                │  │
│  │ ├─ data: [...]                        │  │
│  │ ├─ status: 'success'                  │  │
│  │ ├─ dataUpdatedAt: 1234567890          │  │
│  │ ├─ isStale: false                     │  │
│  │ └─ staleTime: 5 min                   │  │
│  │                                       │  │
│  │ queryKey: ['products', 1]             │  │
│  │ ├─ data: {...}                        │  │
│  │ ├─ status: 'loading'                  │  │
│  │ └─ ...                                │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  Background Tasks                           │
│  ├─ Refetch when window focused             │
│  ├─ Refetch periodically                    │
│  ├─ Refetch on reconnect                    │
│  └─ Garbage collect old queries             │
│                                             │
│  Components Subscribed                      │
│  ├─ ProductsPage (watching ['products'])   │
│  ├─ ProductHeader (watching ['products'])  │
│  ├─ ProductDetail (watching ['products', 1])
│  └─ Dashboard (watching ['products', 2])   │
│                                             │
│  When data updates:                         │
│  → All subscribed components notified      │
│  → Only changed components re-render       │
│  → Perfect sync across app! ✅             │
└─────────────────────────────────────────────┘
```

---

## Installation

```bash
npm install @tanstack/react-query
# or
yarn add @tanstack/react-query
```

---

## Implementation: Shopping Cart with API

### Step 1: Setup QueryClient

```javascript
// queryClient.js

import { QueryClient } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 10, // garbage collect after 10 min (cache time)
      retry: 1,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
  },
});
```

### Step 2: Wrap App with Provider

```javascript
// App.js

import { QueryClientProvider } from "@tanstack/react-query";
import { queryClient } from "./queryClient";

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MainApp />
    </QueryClientProvider>
  );
}
```

### Step 3: Create Query Hooks

```javascript
// hooks/useProducts.js

import { useQuery, useInfiniteQuery } from "@tanstack/react-query";

const API_URL = "https://api.example.com";

// Fetch all products
export function useProducts() {
  return useQuery({
    queryKey: ["products"],
    queryFn: async () => {
      const response = await fetch(`${API_URL}/products`);
      if (!response.ok) throw new Error("Failed to fetch products");
      return response.json();
    },
    staleTime: 1000 * 60 * 5, // 5 minutes
  });
}

// Fetch single product
export function useProduct(productId) {
  return useQuery({
    queryKey: ["products", productId],
    queryFn: async () => {
      const response = await fetch(`${API_URL}/products/${productId}`);
      if (!response.ok) throw new Error("Failed to fetch product");
      return response.json();
    },
    enabled: !!productId, // Only fetch if productId exists
  });
}

// Fetch with search
export function useSearchProducts(searchTerm) {
  return useQuery({
    queryKey: ["products", "search", searchTerm],
    queryFn: async () => {
      const response = await fetch(
        `${API_URL}/products/search?q=${searchTerm}`,
      );
      if (!response.ok) throw new Error("Search failed");
      return response.json();
    },
    enabled: searchTerm.length > 0, // Only search if term > 0
  });
}

// Infinite query for pagination
export function useProductsInfinite() {
  return useInfiniteQuery({
    queryKey: ["products", "infinite"],
    queryFn: async ({ pageParam = 1 }) => {
      const response = await fetch(
        `${API_URL}/products?page=${pageParam}&limit=10`,
      );
      if (!response.ok) throw new Error("Fetch failed");
      return response.json();
    },
    getNextPageParam: (lastPage) =>
      lastPage.hasMore ? lastPage.nextPage : null,
  });
}
```

### Step 4: Create Mutation Hooks

```javascript
// hooks/useCart.js

import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";

export function useAddToCart() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (product) => {
      const response = await fetch(`${API_URL}/cart/add`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(product),
      });
      if (!response.ok) throw new Error("Failed to add to cart");
      return response.json();
    },

    // Optimistic update
    onMutate: async (newProduct) => {
      // Cancel ongoing fetches
      await queryClient.cancelQueries({ queryKey: ["cart"] });

      // Get previous data
      const previousCart = queryClient.getQueryData(["cart"]);

      // Optimistically update cache
      queryClient.setQueryData(["cart"], (old) => ({
        ...old,
        items: [...(old?.items || []), newProduct],
      }));

      // Return rollback function
      return { previousCart };
    },

    // On error, rollback
    onError: (error, newProduct, context) => {
      queryClient.setQueryData(["cart"], context.previousCart);
    },

    // Refetch after mutation
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["cart"] });
    },
  });
}

export function useRemoveFromCart() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (itemId) => {
      const response = await fetch(`${API_URL}/cart/remove/${itemId}`, {
        method: "DELETE",
      });
      if (!response.ok) throw new Error("Failed to remove from cart");
      return response.json();
    },

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["cart"] });
    },
  });
}

export function useUpdateCartQuantity() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ itemId, quantity }) => {
      const response = await fetch(`${API_URL}/cart/update/${itemId}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ quantity }),
      });
      if (!response.ok) throw new Error("Failed to update quantity");
      return response.json();
    },

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["cart"] });
    },
  });
}

export function useClearCart() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async () => {
      const response = await fetch(`${API_URL}/cart/clear`, {
        method: "DELETE",
      });
      if (!response.ok) throw new Error("Failed to clear cart");
      return response.json();
    },

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["cart"] });
    },
  });
}

// Get cart data
export function useCart() {
  return useQuery({
    queryKey: ["cart"],
    queryFn: async () => {
      const response = await fetch(`${API_URL}/cart`);
      if (!response.ok) throw new Error("Failed to fetch cart");
      return response.json();
    },
  });
}
```

### Step 5: Use in Components

```javascript
// screens/ProductsScreen.js

import React, { useState } from "react";
import {
  View,
  ScrollView,
  TouchableOpacity,
  Text,
  StyleSheet,
  ActivityIndicator,
} from "react-native";
import { useProducts } from "../hooks/useProducts";
import { useAddToCart } from "../hooks/useCart";

export default function ProductsScreen() {
  const { data: products, isLoading, error } = useProducts();
  const addToCartMutation = useAddToCart();

  if (isLoading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.errorText}>Error: {error.message}</Text>
      </View>
    );
  }

  const handleAddToCart = (product) => {
    addToCartMutation.mutate(product, {
      onSuccess: () => {
        alert("Added to cart!");
      },
      onError: (error) => {
        alert(`Error: ${error.message}`);
      },
    });
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Products</Text>

      <ScrollView>
        {products?.map((product) => (
          <View key={product.id} style={styles.productCard}>
            <View>
              <Text style={styles.productName}>{product.name}</Text>
              <Text style={styles.productPrice}>${product.price}</Text>
              <Text style={styles.productDesc}>{product.description}</Text>
            </View>

            <TouchableOpacity
              style={[
                styles.addButton,
                addToCartMutation.isPending && styles.addButtonLoading,
              ]}
              onPress={() => handleAddToCart(product)}
              disabled={addToCartMutation.isPending}
            >
              {addToCartMutation.isPending ? (
                <ActivityIndicator size="small" color="#fff" />
              ) : (
                <Text style={styles.addButtonText}>Add to Cart</Text>
              )}
            </TouchableOpacity>
          </View>
        ))}
      </ScrollView>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: "#fff", padding: 10 },
  centerContainer: { flex: 1, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 10 },
  productCard: {
    padding: 15,
    marginVertical: 5,
    backgroundColor: "#f9f9f9",
    borderRadius: 8,
    borderWidth: 1,
    borderColor: "#eee",
  },
  productName: { fontSize: 16, fontWeight: "600" },
  productPrice: { fontSize: 14, color: "#ff6b6b", fontWeight: "bold" },
  productDesc: { fontSize: 12, color: "#666", marginTop: 5 },
  addButton: {
    backgroundColor: "#007AFF",
    padding: 10,
    borderRadius: 5,
    marginTop: 10,
    alignItems: "center",
  },
  addButtonLoading: { opacity: 0.7 },
  addButtonText: { color: "#fff", fontWeight: "600" },
  errorText: { color: "#ff6b6b", fontSize: 16 },
});
```

```javascript
// screens/CartScreen.js

import React from "react";
import {
  View,
  ScrollView,
  TouchableOpacity,
  Text,
  StyleSheet,
  FlatList,
  ActivityIndicator,
} from "react-native";
import {
  useCart,
  useRemoveFromCart,
  useUpdateCartQuantity,
  useClearCart,
} from "../hooks/useCart";

export default function CartScreen() {
  const { data: cart, isLoading, error } = useCart();
  const removeFromCartMutation = useRemoveFromCart();
  const updateQuantityMutation = useUpdateCartQuantity();
  const clearCartMutation = useClearCart();

  if (isLoading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.errorText}>Error loading cart</Text>
      </View>
    );
  }

  if (!cart?.items || cart.items.length === 0) {
    return (
      <View style={styles.emptyContainer}>
        <Text style={styles.emptyText}>Your cart is empty</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={cart.items}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.cartItem}>
            <View style={styles.itemInfo}>
              <Text style={styles.itemName}>{item.name}</Text>
              <Text style={styles.itemPrice}>${item.price}</Text>
            </View>

            <View style={styles.quantityControl}>
              <TouchableOpacity
                onPress={() =>
                  updateQuantityMutation.mutate({
                    itemId: item.id,
                    quantity: Math.max(1, item.quantity - 1),
                  })
                }
                disabled={updateQuantityMutation.isPending}
              >
                <Text style={styles.quantityButton}>-</Text>
              </TouchableOpacity>
              <Text style={styles.quantity}>{item.quantity}</Text>
              <TouchableOpacity
                onPress={() =>
                  updateQuantityMutation.mutate({
                    itemId: item.id,
                    quantity: item.quantity + 1,
                  })
                }
                disabled={updateQuantityMutation.isPending}
              >
                <Text style={styles.quantityButton}>+</Text>
              </TouchableOpacity>
            </View>

            <TouchableOpacity
              onPress={() => removeFromCartMutation.mutate(item.id)}
              disabled={removeFromCartMutation.isPending}
            >
              <Text style={styles.removeText}>Remove</Text>
            </TouchableOpacity>
          </View>
        )}
      />

      <View style={styles.totalSection}>
        <Text style={styles.totalLabel}>Total:</Text>
        <Text style={styles.totalAmount}>${cart.total?.toFixed(2)}</Text>
      </View>

      <TouchableOpacity
        style={[
          styles.clearButton,
          clearCartMutation.isPending && styles.disabled,
        ]}
        onPress={() => clearCartMutation.mutate()}
        disabled={clearCartMutation.isPending}
      >
        <Text style={styles.buttonText}>Clear Cart</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: "#fff" },
  centerContainer: { flex: 1, justifyContent: "center", alignItems: "center" },
  emptyContainer: { flex: 1, justifyContent: "center", alignItems: "center" },
  emptyText: { fontSize: 18, color: "#999" },
  cartItem: {
    flexDirection: "row",
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: "#eee",
    alignItems: "center",
  },
  itemInfo: { flex: 1 },
  itemName: { fontSize: 16, fontWeight: "600" },
  itemPrice: { fontSize: 14, color: "#666" },
  quantityControl: {
    flexDirection: "row",
    alignItems: "center",
    gap: 8,
  },
  quantityButton: { paddingHorizontal: 8, fontWeight: "bold" },
  quantity: { marginHorizontal: 8, fontWeight: "600" },
  removeText: { color: "#ff6b6b", fontWeight: "600" },
  totalSection: {
    flexDirection: "row",
    justifyContent: "space-between",
    padding: 20,
    backgroundColor: "#f9f9f9",
  },
  totalLabel: { fontSize: 16, fontWeight: "600" },
  totalAmount: { fontSize: 16, fontWeight: "bold", color: "#007AFF" },
  clearButton: {
    backgroundColor: "#ff6b6b",
    margin: 10,
    padding: 12,
    borderRadius: 8,
    alignItems: "center",
  },
  buttonText: { color: "#fff", fontWeight: "600" },
  disabled: { opacity: 0.6 },
  errorText: { color: "#ff6b6b", fontSize: 16 },
  centerContainer: { flex: 1, justifyContent: "center", alignItems: "center" },
});
```

---

## Interview Q&A: React Query

### Q1: What's the difference between React Query and Redux?

**Answer:**

```
REACT QUERY:
✓ ONLY for server state (API data)
✓ Automatic caching
✓ Automatic background sync
✓ Handles duplicates and race conditions
✓ Minimal boilerplate
✓ Great for API interactions

REDUX:
✓ For BOTH server and client state
✓ Manual state management
✓ Manual caching logic
✓ More control
✓ More boilerplate
✓ Better for complex business logic

KEY INSIGHT:
React Query + Zustand = Better than Redux for most apps

React Query handles:
- Products from /api/products
- User from /api/user
- Orders from /api/orders

Zustand handles:
- Theme (light/dark)
- Language (en/es)
- UI state (modal open)
```

### Q2: How do you prevent duplicate API calls?

**Answer:**

```
React Query automatically deduplicates using queryKey!

// Two components request same data
function ProductList() {
  const { data } = useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });
}

function ProductHeader() {
  const { data } = useQuery({
    queryKey: ['products'],  // SAME queryKey
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });
}

RESULT:
✓ Both components request ['products']
✓ React Query recognizes same queryKey
✓ First request starts
✓ Second request waits for same response
✓ Only ONE fetch happens
✓ Both components get data
✓ Perfect deduplication!

CONFIG:
queryClient.setQueryDefaults(['products'], {
  staleTime: 1000 * 60 * 5,  // Cache 5 minutes
  gcTime: 1000 * 60 * 10,    // Garbage collect after 10 min
  retry: 1,                   // Retry once on failure
});
```

### Q3: What's caching vs stale time vs gc time?

**Answer:**

```
THREE KEY CONCEPTS:

1. CACHING
   Data stored in memory
   When: After first successful request
   How long: Until removed from cache
   Examples:
   ✓ Load products page
   ✓ Data cached in React Query
   ✓ Navigate to profile
   ✓ Come back to products
   ✓ Still in cache!

2. STALE TIME
   How long until data is considered "stale"
   When: After staleTime, data is considered old
   What happens: Background refetch triggered

   Example:
   staleTime: 5 minutes
   ├─ Load products at 10:00 AM
   ├─ Data fresh until 10:05 AM
   ├─ At 10:06 AM, automatically refetch
   └─ Show cached data while refetching

3. GC TIME (Garbage Collection)
   How long to keep unused data in cache
   When: After gcTime, remove from memory if not used

   Example:
   gcTime: 10 minutes
   ├─ Load products
   ├─ Don't use for 10 minutes
   ├─ Data removed from cache (free memory)
   └─ Next request = fresh fetch

TYPICAL SETUP:
staleTime: 5 min   (5 min fresh, then background refetch)
gcTime: 10 min     (keep cache 10 min, then remove)

SPECIAL CASES:
staleTime: 0       (always stale, always background refetch)
staleTime: Infinity (never stale, never refetch unless invalidated)
gcTime: Infinity   (keep in cache forever)
```

### Q4: How does optimistic update work?

**Answer:**

```
OPTIMISTIC UPDATE = Update UI before server confirms

FLOW:
1. User clicks "Add to Cart"
2. Optimistically update local cache
3. Show "Added!" immediately (great UX)
4. Send request to server
5. If success: Keep optimistic update
6. If error: Rollback to previous state

IMPLEMENTATION:

useMutation({
  mutationFn: (product) =>
    fetch('/api/cart/add', { method: 'POST', body: JSON.stringify(product) }),

  // Called BEFORE mutation
  onMutate: async (newProduct) => {
    // Step 1: Cancel any ongoing fetches
    await queryClient.cancelQueries({ queryKey: ['cart'] });

    // Step 2: Save previous state (for rollback)
    const previousCart = queryClient.getQueryData(['cart']);

    // Step 3: Optimistically update cache
    queryClient.setQueryData(['cart'], (old) => ({
      ...old,
      items: [...old.items, newProduct],
    }));

    // Step 4: Return context for rollback
    return { previousCart };
  },

  // Called if mutation fails
  onError: (error, newProduct, context) => {
    // Rollback to previous state
    queryClient.setQueryData(['cart'], context.previousCart);
  },

  // Called on success
  onSuccess: () => {
    // Refetch cart to confirm server state
    queryClient.invalidateQueries({ queryKey: ['cart'] });
  },
});

USER EXPERIENCE:
✅ Click "Add to Cart"
✅ See item immediately (optimistic)
✅ No waiting for server
✅ If error: Item removed automatically
✅ If success: Data stays and syncs

Perfect UX! 🎉
```

### Q5: When do you use useQuery vs useMutation?

**Answer:**

```
USE useQuery FOR:
✓ Fetching data
✓ GET requests
✓ Read-only operations
✓ Automatic refetching

USE useMutation FOR:
✓ Creating/updating/deleting data
✓ POST/PUT/DELETE requests
✓ Side effects
✓ Manual control needed

EXAMPLES:

useQuery:
- const { data: products } = useQuery(['products'])     // GET
- const { data: user } = useQuery(['user', userId])    // GET
- const { data: orders } = useQuery(['orders'])        // GET

useMutation:
- useMutation(() => fetch('/add', { method: 'POST' }))  // POST
- useMutation(({ id, data }) => fetch(`/update/${id}`, { method: 'PUT' }))  // PUT
- useMutation((id) => fetch(`/delete/${id}`, { method: 'DELETE' }))  // DELETE

NEVER:
❌ useQuery for mutations (not designed for it)
❌ useMutation for fetching (won't auto-refetch)
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Cache Key

```javascript
// WRONG - Different keys = different cache entries
const page1 = useQuery({
  queryKey: ["products"], // 🔴 Generic key
  queryFn: () => fetch("/api/products?page=1").then((r) => r.json()),
});

const page2 = useQuery({
  queryKey: ["products"], // 🔴 SAME key but different data!
  queryFn: () => fetch("/api/products?page=2").then((r) => r.json()),
});

// RIGHT - Include all parameters in key
const page1 = useQuery({
  queryKey: ["products", { page: 1 }], // ✅ Unique key
  queryFn: () => fetch("/api/products?page=1").then((r) => r.json()),
});

const page2 = useQuery({
  queryKey: ["products", { page: 2 }], // ✅ Different key
  queryFn: () => fetch("/api/products?page=2").then((r) => r.json()),
});
```

### ❌ Mistake 2: Not Invalidating After Mutations

```javascript
// WRONG - Cart doesn't update after adding
useMutation({
  mutationFn: (product) =>
    fetch("/api/cart/add", { method: "POST", body: JSON.stringify(product) }),
  // 🔴 No invalidation = cart data doesn't refresh
});

// RIGHT - Invalidate to refetch
useMutation({
  mutationFn: (product) =>
    fetch("/api/cart/add", { method: "POST", body: JSON.stringify(product) }),

  onSuccess: () => {
    // ✅ Refetch cart after adding
    queryClient.invalidateQueries({ queryKey: ["cart"] });
  },
});
```

---

## Key Takeaways

1. ✅ React Query is the STANDARD for server state
2. ✅ Solves deduplication, caching, stale data, race conditions
3. ✅ Pair with Zustand for complete solution
4. ✅ Use proper cache keys including all parameters
5. ✅ Invalidate queries after mutations
6. ✅ Use optimistic updates for better UX
7. ✅ Set appropriate staleTime and gcTime

---

**Next: Learn Redux Toolkit for enterprise apps →**
