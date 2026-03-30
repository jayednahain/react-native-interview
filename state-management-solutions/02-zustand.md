# ⭐ Zustand - Shopping Cart Example (Most Recommended 2026)

## Overview

**Zustand** is a lightweight state management library that's MUCH simpler than Redux but more powerful than Context API.

Think of it as: **Redux's power + Zustand's simplicity = Perfect 80/20 solution**

---

## Simple Definition

**Zustand** = A tiny library that lets you create global state stores with minimal boilerplate, no providers, and excellent performance.

Key feature: **Direct mutations** (looks like you're mutating state, but it's actually immutable under the hood)

---

## Why Zustand Exists

### The Redux Problem

```javascript
// Redux: 100 lines of boilerplate for one feature
// 1. Action types
export const FETCH_PRODUCTS_REQUEST = "FETCH_PRODUCTS_REQUEST";
export const FETCH_PRODUCTS_SUCCESS = "FETCH_PRODUCTS_SUCCESS";
export const FETCH_PRODUCTS_ERROR = "FETCH_PRODUCTS_ERROR";
export const ADD_PRODUCT = "ADD_PRODUCT";

// 2. Action creators
export const fetchProductsRequest = () => ({
  type: FETCH_PRODUCTS_REQUEST,
});
// ... more boilerplate

// 3. Reducer
const initialState = { products: [], loading: false, error: null };
export function productsReducer(state = initialState, action) {
  switch (action.type) {
    case FETCH_PRODUCTS_REQUEST:
      return { ...state, loading: true };
    // ... 20 more cases
  }
}

// All this for just one feature! 😫
```

### The Zustand Solution

```javascript
// Zustand: 10 lines
import create from "zustand";

const useProductStore = create((set) => ({
  products: [],
  loading: false,
  error: null,

  addProduct: (product) =>
    set((state) => ({
      products: [...state.products, product],
    })),
}));

// That's it! 🎉
```

---

## Architecture Diagram

```
┌──────────────────────────────────────┐
│      Zustand Store                   │
│  ┌────────────────────────────────┐  │
│  │  State                         │  │
│  │  ├─ items: []                  │  │
│  │  ├─ total: 0                   │  │
│  │  ├─ loading: false             │  │
│  │  └─ error: null                │  │
│  │                                │  │
│  │  Actions (Direct mutations)    │  │
│  │  ├─ addItem(product)           │  │
│  │  ├─ removeItem(id)             │  │
│  │  ├─ updateQuantity(id, qty)    │  │
│  │  └─ clearCart()                │  │
│  └────────────────────────────────┘  │
│                                      │
│  Subscribers                         │
│  (Components using useStore)         │
│  ├─ ProductList (watches items)     │
│  ├─ CartIcon (watches total)        │
│  └─ CheckoutButton (reads state)    │
│                                      │
│  ONLY subscribed components          │
│  re-render when THEIR part changes   │
└──────────────────────────────────────┘
```

---

## Installation

```bash
npm install zustand
# or
yarn add zustand
```

---

## Implementation: Shopping Cart

### Step 1: Create Store

```javascript
// store/cartStore.js

import create from "zustand";
import { devtools, persist } from "zustand/middleware";

export const useCartStore = create(
  devtools(
    persist(
      (set, get) => ({
        // STATE
        items: [],
        total: 0,
        loading: false,
        error: null,

        // ACTIONS - Direct mutations (Immer-like)
        addItem: (product) =>
          set(
            (state) => {
              const existingItem = state.items.find(
                (item) => item.id === product.id,
              );

              let newItems;
              if (existingItem) {
                newItems = state.items.map((item) =>
                  item.id === product.id
                    ? { ...item, quantity: item.quantity + 1 }
                    : item,
                );
              } else {
                newItems = [...state.items, { ...product, quantity: 1 }];
              }

              const newTotal = newItems.reduce(
                (sum, item) => sum + item.price * item.quantity,
                0,
              );

              return {
                items: newItems,
                total: parseFloat(newTotal.toFixed(2)),
                error: null,
              };
            },
            false,
            "cart/addItem",
          ),

        removeItem: (itemId) =>
          set(
            (state) => {
              const newItems = state.items.filter((item) => item.id !== itemId);
              const newTotal = newItems.reduce(
                (sum, item) => sum + item.price * item.quantity,
                0,
              );

              return {
                items: newItems,
                total: parseFloat(newTotal.toFixed(2)),
              };
            },
            false,
            "cart/removeItem",
          ),

        updateQuantity: (itemId, quantity) =>
          set(
            (state) => {
              if (quantity <= 0) {
                // Delegate to removeItem
                return useCartStore.getState().removeItem(itemId);
              }

              const newItems = state.items.map((item) =>
                item.id === itemId ? { ...item, quantity } : item,
              );

              const newTotal = newItems.reduce(
                (sum, item) => sum + item.price * item.quantity,
                0,
              );

              return {
                items: newItems,
                total: parseFloat(newTotal.toFixed(2)),
              };
            },
            false,
            "cart/updateQuantity",
          ),

        clearCart: () =>
          set(
            {
              items: [],
              total: 0,
              error: null,
            },
            false,
            "cart/clearCart",
          ),

        setLoading: (loading) => set({ loading }, false, "cart/setLoading"),

        setError: (error) => set({ error }, false, "cart/setError"),

        // SELECTORS - Derived state
        getItemCount: () => get().items.length,

        getItemById: (itemId) => get().items.find((item) => item.id === itemId),

        hasItem: (itemId) => get().items.some((item) => item.id === itemId),
      }),
      {
        name: "shopping-cart-store", // localStorage key
      },
    ),
    { name: "CartStore" }, // DevTools name
  ),
);
```

### Step 2: Create Selectors for Optimization

```javascript
// store/cartSelectors.js

// Selectors prevent unnecessary re-renders
// Component only re-renders when ITS selected value changes

export const selectCartItems = (state) => state.items;
export const selectCartTotal = (state) => state.total;
export const selectCartLoading = (state) => state.loading;
export const selectCartError = (state) => state.error;
export const selectItemCount = (state) => state.items.length;

// Derived selectors
export const selectCartSubtotal = (state) =>
  state.items.reduce((sum, item) => sum + item.price * item.quantity, 0);

export const selectTaxAmount = (state) => {
  const subtotal = selectCartSubtotal(state);
  return parseFloat((subtotal * 0.1).toFixed(2)); // 10% tax
};

export const selectCartTotal = (state) => {
  const subtotal = selectCartSubtotal(state);
  const tax = selectTaxAmount(state);
  return parseFloat((subtotal + tax).toFixed(2));
};
```

### Step 3: Use Store in Components (No Provider!)

```javascript
// screens/ProductsScreen.js

import React from "react";
import {
  View,
  ScrollView,
  TouchableOpacity,
  Text,
  StyleSheet,
} from "react-native";
import { useCartStore } from "../store/cartStore";

const PRODUCTS = [
  { id: 1, name: "Laptop", price: 999.99 },
  { id: 2, name: "Phone", price: 799.99 },
  { id: 3, name: "Tablet", price: 499.99 },
  { id: 4, name: "Headphones", price: 199.99 },
];

export default function ProductsScreen() {
  // Select only what you need - only re-renders when itemCount changes!
  const itemCount = useCartStore((state) => state.items.length);
  const addItem = useCartStore((state) => state.addItem);

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Products</Text>
        <TouchableOpacity style={styles.cartBadge}>
          <Text style={styles.cartCount}>{itemCount}</Text>
        </TouchableOpacity>
      </View>

      <ScrollView style={styles.productList}>
        {PRODUCTS.map((product) => (
          <View key={product.id} style={styles.productCard}>
            <View>
              <Text style={styles.productName}>{product.name}</Text>
              <Text style={styles.productPrice}>${product.price}</Text>
            </View>
            <TouchableOpacity
              style={styles.addButton}
              onPress={() => addItem(product)}
            >
              <Text style={styles.addButtonText}>Add to Cart</Text>
            </TouchableOpacity>
          </View>
        ))}
      </ScrollView>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: "#fff" },
  header: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
    padding: 20,
    borderBottomWidth: 1,
    borderBottomColor: "#eee",
  },
  title: { fontSize: 24, fontWeight: "bold" },
  cartBadge: {
    backgroundColor: "#ff6b6b",
    width: 30,
    height: 30,
    borderRadius: 15,
    justifyContent: "center",
    alignItems: "center",
  },
  cartCount: { color: "#fff", fontWeight: "bold" },
  productList: { padding: 10 },
  productCard: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
    padding: 15,
    marginVertical: 5,
    backgroundColor: "#f9f9f9",
    borderRadius: 8,
  },
  productName: { fontSize: 16, fontWeight: "600" },
  productPrice: { fontSize: 14, color: "#666", marginTop: 5 },
  addButton: {
    backgroundColor: "#007AFF",
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 5,
  },
  addButtonText: { color: "#fff", fontWeight: "600" },
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
} from "react-native";
import { useCartStore } from "../store/cartStore";

export default function CartScreen() {
  // Each selector = fine-grained subscription
  const items = useCartStore((state) => state.items);
  const total = useCartStore((state) => state.total);
  const removeItem = useCartStore((state) => state.removeItem);
  const updateQuantity = useCartStore((state) => state.updateQuantity);
  const clearCart = useCartStore((state) => state.clearCart);

  // Re-render only when items change, not when total changes (optimization!)
  const itemCount = useCartStore((state) => state.items.length);

  if (itemCount === 0) {
    return (
      <View style={styles.emptyContainer}>
        <Text style={styles.emptyText}>Your cart is empty</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={items}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.cartItem}>
            <View style={styles.itemInfo}>
              <Text style={styles.itemName}>{item.name}</Text>
              <Text style={styles.itemPrice}>${item.price}</Text>
            </View>

            <View style={styles.quantityControl}>
              <TouchableOpacity
                onPress={() => updateQuantity(item.id, item.quantity - 1)}
                style={styles.quantityButton}
              >
                <Text style={styles.buttonText}>-</Text>
              </TouchableOpacity>

              <Text style={styles.quantity}>{item.quantity}</Text>

              <TouchableOpacity
                onPress={() => updateQuantity(item.id, item.quantity + 1)}
                style={styles.quantityButton}
              >
                <Text style={styles.buttonText}>+</Text>
              </TouchableOpacity>
            </View>

            <Text style={styles.subtotal}>
              ${(item.price * item.quantity).toFixed(2)}
            </Text>

            <TouchableOpacity
              onPress={() => removeItem(item.id)}
              style={styles.removeButton}
            >
              <Text style={styles.removeText}>Remove</Text>
            </TouchableOpacity>
          </View>
        )}
      />

      <View style={styles.totalSection}>
        <Text style={styles.totalLabel}>Total:</Text>
        <Text style={styles.totalAmount}>${total.toFixed(2)}</Text>
      </View>

      <View style={styles.actionButtons}>
        <TouchableOpacity
          style={[styles.button, styles.clearButton]}
          onPress={clearCart}
        >
          <Text style={styles.buttonTextWhite}>Clear Cart</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.button, styles.checkoutButton]}>
          <Text style={styles.buttonTextWhite}>Checkout</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: "#fff" },
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
  itemPrice: { fontSize: 14, color: "#666", marginTop: 5 },
  quantityControl: {
    flexDirection: "row",
    alignItems: "center",
    marginHorizontal: 10,
  },
  quantityButton: {
    width: 30,
    height: 30,
    justifyContent: "center",
    alignItems: "center",
    backgroundColor: "#f0f0f0",
    borderRadius: 5,
  },
  buttonText: { fontWeight: "bold" },
  quantity: { marginHorizontal: 8, fontWeight: "600" },
  subtotal: { fontWeight: "600", minWidth: 70 },
  removeButton: { marginLeft: 10 },
  removeText: { color: "#ff6b6b", fontWeight: "600" },
  totalSection: {
    flexDirection: "row",
    justifyContent: "space-between",
    padding: 20,
    backgroundColor: "#f9f9f9",
    borderTopWidth: 1,
    borderTopColor: "#eee",
  },
  totalLabel: { fontSize: 18, fontWeight: "600" },
  totalAmount: { fontSize: 18, fontWeight: "bold", color: "#007AFF" },
  actionButtons: { flexDirection: "row", padding: 15, gap: 10 },
  button: {
    flex: 1,
    paddingVertical: 12,
    borderRadius: 8,
    justifyContent: "center",
    alignItems: "center",
  },
  clearButton: { backgroundColor: "#999" },
  checkoutButton: { backgroundColor: "#007AFF" },
  buttonTextWhite: { color: "#fff", fontWeight: "600", fontSize: 16 },
});
```

### Step 4: Advanced: Combine Multiple Stores

```javascript
// store/useAppStore.js

import create from "zustand";
import { useCartStore } from "./cartStore";
import { useAuthStore } from "./authStore";
import { useThemeStore } from "./themeStore";

// Combine stores for convenience
export function useAppStore() {
  return {
    cart: useCartStore(),
    auth: useAuthStore(),
    theme: useThemeStore(),
  };
}

// OR use composition pattern
export const useShoppingApp = () => ({
  // Cart
  items: useCartStore((state) => state.items),
  total: useCartStore((state) => state.total),
  addItem: useCartStore((state) => state.addItem),

  // Auth
  user: useAuthStore((state) => state.user),
  isLoggedIn: useAuthStore((state) => state.isLoggedIn),

  // Theme
  isDarkMode: useThemeStore((state) => state.isDarkMode),
  toggleTheme: useThemeStore((state) => state.toggleTheme),
});
```

---

## Interview Q&A: Zustand

### Q1: Why Zustand over Redux?

**Answer:**

```
ZUSTAND ADVANTAGES:

1. Less Boilerplate
   Redux: 100 lines (actions, reducers, types)
   Zustand: 10 lines (just state + actions)

2. No Providers
   Redux: Wrap entire app with <Provider store={store}>
   Zustand: Just import and use

3. Faster Development
   Redux: Write reducers, actions, selectors
   Zustand: Direct mutations

4. Better Bundle Size
   Redux: 26KB
   Zustand: 2.3KB

5. Performance
   Zustand: Fine-grained subscriptions (only affected components re-render)
   Redux: Selector pattern (need knowledge to optimize)

WHEN TO USE REDUX INSTEAD:
- Large enterprise teams (20+ developers)
- Standardization is critical
- Time-travel debugging is essential
- Complex middleware needs

RECOMMENDATION:
For 95% of apps: Zustand + React Query is better than Redux
```

### Q2: How do Zustand selectors prevent re-renders?

**Answer:**

```
KEY INSIGHT: Zustand automatically optimizes subscriptions

// ❌ BAD - Component re-renders when ANY state changes
const state = useCartStore();
// Changed items? Re-render
// Changed loading? Re-render
// Changed error? Re-render

// ✅ GOOD - Component only re-renders when itemCount changes
const itemCount = useCartStore((state) => state.items.length);

INSIDE ZUSTAND:
1. You pass a selector function: (state) => state.items.length
2. Zustand tracks what you selected
3. It compares: oldValue vs newValue (using Object.is)
4. If different: component re-renders
5. If same: component doesn't re-render

EXAMPLE:

// Component A - re-renders when items change
function CartIcon() {
  const count = useCartStore((state) => state.items.length);
  return <Badge count={count} />;
}

// Component B - re-renders when total changes
function TotalDisplay() {
  const total = useCartStore((state) => state.total);
  return <Text>${total}</Text>;
}

// Change items array: Component A re-renders, B doesn't
// Change total: Component B re-renders, A doesn't
// Perfect optimization! ✅
```

### Q3: How do you handle async operations in Zustand?

**Answer:**

```
APPROACH 1: Async directly in actions
export const useUserStore = create((set) => ({
  users: [],
  loading: false,
  error: null,

  fetchUsers: async () => {
    set({ loading: true, error: null });
    try {
      const response = await fetch('/api/users');
      const users = await response.json();
      set({ users, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
}));

// Use it
function UserList() {
  const { users, loading, fetchUsers } = useUserStore();

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  return <View>{/* render */}</View>;
}

APPROACH 2: Immer middleware (mutations look like mutations)
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

export const useStore = create(
  immer((set) => ({
    todos: [],

    addTodo: async (text) => {
      const todo = await fetchTodo(text);
      set((state) => {
        state.todos.push(todo); // Looks like mutation!
      });
    },
  }))
);

APPROACH 3: Use with React Query
// React Query for server state
const { data: users } = useQuery(['users'], fetchUsers);

// Zustand for client state
const filters = useFilterStore((state) => state.filters);
const sorting = useSortingStore((state) => state.sorting);

// Perfect separation!

BEST PRACTICE:
- Use React Query for server/API state
- Use Zustand for client/UI state
- Rarely mix async in Zustand unless simple
```

### Q4: What middleware does Zustand support?

**Answer:**

```
BUILT-IN MIDDLEWARE:

1. devtools - Redux DevTools support
   create(devtools((set) => ({...})))

2. persist - localStorage persistence
   create(persist((set) => ({...}), { name: 'my-store' }))

3. immer - Produce-like mutations
   create(immer((set) => ({...})))

4. combine - Combine multiple stores
   combine({ store1, store2 }, ...)

5. subscribeWithSelector - Improved subscriptions
   create(subscribeWithSelector((set) => ({...})))

COMPOSING MIDDLEWARE:
export const useStore = create(
  devtools(
    persist(
      immer((set) => ({
        // store implementation
      })),
      { name: 'my-store' }
    ),
    { name: 'MyStore' }
  )
);

// Stacking multiple middlewares for powerful stores!
```

### Q5: Zustand vs React Context API?

**Answer:**

```
CONTEXT API:
✓ Built-in (no dependency)
✓ Good for learning
✓ Fine for small state
✗ Performance: all consumers re-render on any change
✗ No DevTools
✗ Need providers
✗ More boilerplate for complex state

ZUSTAND:
✓ Fine-grained subscriptions (optimal performance)
✓ No providers needed
✓ DevTools support
✓ Less boilerplate
✓ Better DX
✗ External dependency
✗ Less ecosystem than Redux

WHEN TO USE CONTEXT:
- Learning React state management
- Very simple state (< 3 values)
- No performance concerns

WHEN TO USE ZUSTAND:
- Production apps
- Performance matters
- Developer experience matters
- Most modern apps in 2026

MODERN RECOMMENDATION:
Context API: Learning + simple UI state (theme, language)
Zustand: Production apps
React Query: Server state
```

---

## Common Mistakes

### ❌ Mistake 1: Mutating State Directly

```javascript
// WRONG - Direct mutation won't trigger re-render
export const useStore = create((set) => ({
  user: { name: "John" },

  updateName: (name) => {
    const state = get();
    state.user.name = name; // 🔴 Won't work!
  },
}));

// RIGHT - Create new objects
export const useStore = create((set) => ({
  user: { name: "John" },

  updateName: (name) =>
    set((state) => ({
      user: { ...state.user, name }, // ✅ Creates new object
    })),
}));

// ALTERNATIVE - Use immer middleware
export const useStore = create(
  immer((set) => ({
    user: { name: "John" },

    updateName: (name) =>
      set((state) => {
        state.user.name = name; // ✅ Immer handles immutability
      }),
  })),
);
```

### ❌ Mistake 2: Not Using Selectors

```javascript
// WRONG - Re-renders on ANY state change
function MyComponent() {
  const state = useStore(); // 🔴 Gets entire store
  // Component re-renders when items, total, loading, error change
}

// RIGHT - Select only what you need
function MyComponent() {
  const items = useStore((state) => state.items); // ✅ Only re-renders when items change
  const total = useStore((state) => state.total); // ✅ Only re-renders when total changes
}
```

### ❌ Mistake 3: Forgetting to Memoize Selectors

```javascript
// WRONG - New function every render
function MyComponent() {
  const items = useStore((state) => ({
    all: state.items,
    count: state.items.length,
  })); // 🔴 New object every render!
}

// RIGHT - Memoize selector
const selectItemsInfo = (state) => ({
  all: state.items,
  count: state.items.length,
});

function MyComponent() {
  const items = useStore(selectItemsInfo); // ✅ Same function reference
}
```

---

## Practice Exercise

Build an auth store with:

- Login/logout
- Token management
- User profile
- Persist to localStorage

```javascript
// Solution: See exercises folder
```

---

## Key Takeaways

1. ✅ Zustand is the 80/20 solution for most apps
2. ✅ No providers needed (just import and use)
3. ✅ Direct mutations look cleaner than reducers
4. ✅ Selectors optimize performance automatically
5. ✅ DevTools support helps debugging
6. ✅ Middleware system is powerful and composable
7. ✅ Pair with React Query for complete solution
8. ✅ Much smaller learning curve than Redux

---

**Next: Learn React Query for server state management →**
