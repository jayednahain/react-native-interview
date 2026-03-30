# 🏢 Redux Toolkit (RTK) - Enterprise State Management

## Overview

**Redux Toolkit** = Modern Redux with all the improvements.  
**Still the best choice for large enterprise teams.**

Think of it as: **Redux, but without all the pain.**

---

## Simple Definition

**Redux Toolkit** = Opinionated, batteries-included framework for predictable state management with standardized patterns, excellent DevTools, and enterprise-grade features.

---

## Redux Toolkit vs Original Redux

### The Original Redux Problem (Pre-RTK)

```javascript
// 😫 Original Redux: SO MUCH BOILERPLATE

// 1. Action types
export const FETCH_PRODUCTS_REQUEST = "FETCH_PRODUCTS_REQUEST";
export const FETCH_PRODUCTS_SUCCESS = "FETCH_PRODUCTS_SUCCESS";
export const FETCH_PRODUCTS_ERROR = "FETCH_PRODUCTS_ERROR";
export const ADD_TO_CART = "ADD_TO_CART";
export const REMOVE_FROM_CART = "REMOVE_FROM_CART";

// 2. Action creators
export const addToCart = (product) => ({
  type: ADD_TO_CART,
  payload: product,
});

// 3. Reducer with switch statement
const initialState = { items: [], total: 0 };

export const cartReducer = (state = initialState, action) => {
  switch (action.type) {
    case ADD_TO_CART:
      return {
        ...state,
        items: [...state.items, action.payload],
      };
    case REMOVE_FROM_CART:
      return {
        ...state,
        items: state.items.filter((item) => item.id !== action.payload),
      };
    default:
      return state;
  }
};

// 4. Middleware for async
export const fetchProducts = () => async (dispatch) => {
  dispatch({ type: FETCH_PRODUCTS_REQUEST });
  try {
    const data = await fetch("/api/products").then((r) => r.json());
    dispatch({ type: FETCH_PRODUCTS_SUCCESS, payload: data });
  } catch (error) {
    dispatch({ type: FETCH_PRODUCTS_ERROR, payload: error });
  }
};

// 5. Create store
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";

export const store = createStore(cartReducer, applyMiddleware(thunk));
```

### Redux Toolkit Solution

```javascript
// ✅ Redux Toolkit: Clean and modern

import {
  createSlice,
  configureStore,
  createAsyncThunk,
} from "@reduxjs/toolkit";

// 1. Async thunk (replaces middleware)
export const fetchProducts = createAsyncThunk("products/fetch", async () => {
  const response = await fetch("/api/products");
  return response.json();
});

// 2. Slice (combines actions + reducers)
const cartSlice = createSlice({
  name: "cart",
  initialState: { items: [], total: 0, loading: false },

  reducers: {
    // Synchronous actions (Immer handles immutability)
    addItem: (state, action) => {
      state.items.push(action.payload); // Looks like mutation!
      state.total += action.payload.price;
    },

    removeItem: (state, action) => {
      state.items = state.items.filter((i) => i.id !== action.payload);
    },
  },

  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state) => {
        state.loading = false;
      });
  },
});

// 3. Configure store (auto-sets up DevTools + middleware)
export const store = configureStore({
  reducer: {
    cart: cartSlice.reducer,
  },
});

// 4. Export actions
export const { addItem, removeItem } = cartSlice.actions;
```

**That's it! Much simpler, right?** ✅

---

## Architecture Diagram

```
┌──────────────────────────────────────┐
│      Redux Store                     │
│  ┌────────────────────────────────┐  │
│  │  State                         │  │
│  │  ├─ cart:                      │  │
│  │  │   ├─ items: []              │  │
│  │  │   └─ total: 0               │  │
│  │  ├─ products:                  │  │
│  │  │   ├─ list: []               │  │
│  │  │   └─ loading: false         │  │
│  │  └─ auth:                      │  │
│  │      ├─ user: null             │  │
│  │      └─ token: null            │  │
│  └────────────────────────────────┘  │
│                                      │
│  Actions & Async Thunks             │
│  ├─ cart/addItem                    │
│  ├─ cart/removeItem                 │
│  ├─ products/fetchProducts (async)  │
│  └─ auth/loginUser (async)          │
│                                      │
│  Middleware                          │
│  ├─ Redux Thunk (async)             │
│  ├─ Logger (logging)                │
│  ├─ Custom Middleware               │
│  └─ DevTools Middleware             │
│                                      │
│  Redux DevTools                      │
│  ├─ Action Timeline                 │
│  ├─ State Changes                   │
│  ├─ Time-Travel Debugging           │
│  ├─ Action Dispatch                 │
│  └─ State Comparison                │
│                                      │
│  Selectors                           │
│  ├─ selectCartItems                 │
│  ├─ selectCartTotal                 │
│  └─ selectIsLoading                 │
│                                      │
│  Components                          │
│  ├─ useSelector(selectCartItems)   │
│  ├─ useDispatch()                   │
│  └─ dispatch(actions)               │
└──────────────────────────────────────┘
```

---

## Installation

```bash
npm install @reduxjs/toolkit react-redux
# or
yarn add @reduxjs/toolkit react-redux
```

---

## Implementation: Shopping Cart with Redux Toolkit

### Step 1: Create Slices

```javascript
// store/cartSlice.js

import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

const API_URL = "https://api.example.com";

// Async thunk for fetching cart
export const fetchCart = createAsyncThunk(
  "cart/fetchCart",
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch(`${API_URL}/cart`);
      if (!response.ok) throw new Error("Failed to fetch cart");
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  },
);

// Async thunk for adding item
export const addToCartAsync = createAsyncThunk(
  "cart/addToCart",
  async (product, { rejectWithValue }) => {
    try {
      const response = await fetch(`${API_URL}/cart/add`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(product),
      });
      if (!response.ok) throw new Error("Failed to add to cart");
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  },
);

export const removeFromCartAsync = createAsyncThunk(
  "cart/removeFromCart",
  async (itemId, { rejectWithValue }) => {
    try {
      const response = await fetch(`${API_URL}/cart/remove/${itemId}`, {
        method: "DELETE",
      });
      if (!response.ok) throw new Error("Failed to remove from cart");
      return itemId;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  },
);

// Create slice
const cartSlice = createSlice({
  name: "cart",
  initialState: {
    items: [],
    total: 0,
    loading: false,
    error: null,
  },

  // Synchronous reducers
  reducers: {
    // Action: clear cart immediately
    clearCart: (state) => {
      state.items = [];
      state.total = 0;
      state.error = null;
    },

    // Action: update quantity
    updateQuantity: (state, action) => {
      const { itemId, quantity } = action.payload;
      const item = state.items.find((i) => i.id === itemId);

      if (item) {
        if (quantity <= 0) {
          state.items = state.items.filter((i) => i.id !== itemId);
        } else {
          item.quantity = quantity;
        }

        // Recalculate total
        state.total = state.items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0,
        );
      }
    },
  },

  // Extra reducers for async thunks
  extraReducers: (builder) => {
    // Fetch cart
    builder
      .addCase(fetchCart.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchCart.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload.items;
        state.total = action.payload.total;
      })
      .addCase(fetchCart.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });

    // Add to cart
    builder
      .addCase(addToCartAsync.pending, (state) => {
        state.error = null;
      })
      .addCase(addToCartAsync.fulfilled, (state, action) => {
        const newItem = action.payload;
        const existingItem = state.items.find((i) => i.id === newItem.id);

        if (existingItem) {
          existingItem.quantity += newItem.quantity;
        } else {
          state.items.push(newItem);
        }

        state.total = state.items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0,
        );
      })
      .addCase(addToCartAsync.rejected, (state, action) => {
        state.error = action.payload;
      });

    // Remove from cart
    builder
      .addCase(removeFromCartAsync.fulfilled, (state, action) => {
        state.items = state.items.filter((i) => i.id !== action.payload);
        state.total = state.items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0,
        );
      })
      .addCase(removeFromCartAsync.rejected, (state, action) => {
        state.error = action.payload;
      });
  },
});

export const { clearCart, updateQuantity } = cartSlice.actions;
export default cartSlice.reducer;
```

### Step 2: Create Store

```javascript
// store/store.js

import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './cartSlice';
import productsReducer from './productsSlice';
import authReducer from './authSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    products: productsReducer,
    auth: authReducer,
  },

  // DevTools configuration
  devTools: process.env.NODE_ENV !== 'production',

  // Middleware configuration
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // Ignore these action types in serialization check
        ignoredActions: ['cart/fetchCart/fulfilled'],
      },
    })
      .concat(customLogger)
      .concat(customErrorHandler),
});

// Custom middleware example
const customLogger = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  const result = next(action);
  console.log('New state:', store.getState());
  return result;
};

const customErrorHandler = (store) => (next) => (action) => {
  try {
    return next(action);
  } catch (error) {
    console.error('Error in action:', error);
    throw error;
  }
};

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Step 3: Create Selectors

```javascript
// store/selectors.js

// Selectors prevent re-renders when state doesn't change
export const selectCartItems = (state) => state.cart.items;
export const selectCartTotal = (state) => state.cart.total;
export const selectCartLoading = (state) => state.cart.loading;
export const selectCartError = (state) => state.cart.error;

// Derived selectors (computed values)
export const selectItemCount = (state) => state.cart.items.length;

export const selectCartSummary = (state) => ({
  itemCount: state.cart.items.length,
  total: state.cart.total,
  hasItems: state.cart.items.length > 0,
});

export const selectItemById = (state, itemId) =>
  state.cart.items.find((item) => item.id === itemId);
```

### Step 4: Wrap App with Provider

```javascript
// App.js

import { Provider } from "react-redux";
import { store } from "./store/store";

export default function App() {
  return (
    <Provider store={store}>
      <MainApp />
    </Provider>
  );
}
```

### Step 5: Use in Components

```javascript
// screens/ProductsScreen.js

import React, { useEffect } from "react";
import {
  View,
  ScrollView,
  TouchableOpacity,
  Text,
  StyleSheet,
  ActivityIndicator,
} from "react-native";
import { useDispatch, useSelector } from "react-redux";
import { addToCartAsync } from "../store/cartSlice";
import { selectCartItems } from "../store/selectors";

const PRODUCTS = [
  { id: 1, name: "Laptop", price: 999.99 },
  { id: 2, name: "Phone", price: 799.99 },
  { id: 3, name: "Tablet", price: 499.99 },
];

export default function ProductsScreen() {
  const dispatch = useDispatch();
  const cartItems = useSelector(selectCartItems);
  const cartItemCount = useSelector((state) => state.cart.items.length);
  const loading = useSelector((state) => state.cart.loading);

  const handleAddToCart = (product) => {
    dispatch(addToCartAsync(product));
  };

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Products</Text>
        <TouchableOpacity style={styles.cartBadge}>
          <Text style={styles.cartCount}>{cartItemCount}</Text>
        </TouchableOpacity>
      </View>

      <ScrollView>
        {PRODUCTS.map((product) => (
          <View key={product.id} style={styles.productCard}>
            <View>
              <Text style={styles.productName}>{product.name}</Text>
              <Text style={styles.productPrice}>${product.price}</Text>
            </View>

            <TouchableOpacity
              style={[styles.addButton, loading && styles.disabled]}
              onPress={() => handleAddToCart(product)}
              disabled={loading}
            >
              {loading ? (
                <ActivityIndicator color="#fff" size="small" />
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
  container: { flex: 1, backgroundColor: "#fff" },
  header: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
    padding: 20,
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
  productCard: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
    padding: 15,
    marginVertical: 5,
    marginHorizontal: 10,
    backgroundColor: "#f9f9f9",
    borderRadius: 8,
  },
  productName: { fontSize: 16, fontWeight: "600" },
  productPrice: { fontSize: 14, color: "#666" },
  addButton: {
    backgroundColor: "#007AFF",
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 5,
  },
  addButtonText: { color: "#fff", fontWeight: "600" },
  disabled: { opacity: 0.5 },
});
```

```javascript
// screens/CartScreen.js

import React, { useEffect } from "react";
import {
  View,
  FlatList,
  TouchableOpacity,
  Text,
  StyleSheet,
} from "react-native";
import { useDispatch, useSelector } from "react-redux";
import {
  clearCart,
  updateQuantity,
  removeFromCartAsync,
} from "../store/cartSlice";
import {
  selectCartItems,
  selectCartTotal,
  selectItemCount,
} from "../store/selectors";

export default function CartScreen() {
  const dispatch = useDispatch();
  const items = useSelector(selectCartItems);
  const total = useSelector(selectCartTotal);
  const itemCount = useSelector(selectItemCount);

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
                onPress={() =>
                  dispatch(
                    updateQuantity({
                      itemId: item.id,
                      quantity: Math.max(1, item.quantity - 1),
                    }),
                  )
                }
              >
                <Text style={styles.quantityButton}>-</Text>
              </TouchableOpacity>
              <Text style={styles.quantity}>{item.quantity}</Text>
              <TouchableOpacity
                onPress={() =>
                  dispatch(
                    updateQuantity({
                      itemId: item.id,
                      quantity: item.quantity + 1,
                    }),
                  )
                }
              >
                <Text style={styles.quantityButton}>+</Text>
              </TouchableOpacity>
            </View>

            <TouchableOpacity
              onPress={() => dispatch(removeFromCartAsync(item.id))}
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

      <TouchableOpacity
        style={styles.clearButton}
        onPress={() => dispatch(clearCart())}
      >
        <Text style={styles.buttonText}>Clear Cart</Text>
      </TouchableOpacity>
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
  },
  itemInfo: { flex: 1 },
  itemName: { fontSize: 16, fontWeight: "600" },
  itemPrice: { fontSize: 14, color: "#666" },
  quantityControl: { flexDirection: "row", alignItems: "center", gap: 8 },
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
});
```

---

## Interview Q&A: Redux Toolkit

### Q1: When should you use Redux vs Zustand?

**Answer:**

```
REDUX BEST FOR:
✓ Teams with 20+ developers
✓ Requiring standardization
✓ Complex business logic audit trail needed
✓ Time-travel debugging critical
✓ Large ecosystem dependencies

ZUSTAND BEST FOR:
✓ Teams < 20 developers
✓ Speed matters
✓ Less boilerplate preference
✓ Bundle size concerns
✓ DX over standardization

MODERN RECOMMENDATION (2026):
For most companies: Zustand + React Query
For large enterprises: Redux Toolkit + RTK Query

If team knows Redux:
→ Use Redux Toolkit (modernized)

If team doesn't know Redux:
→ Use Zustand (easier)
```

### Q2: What's the advantage of createSlice?

**Answer:**

```
createSlice solves Redux boilerplate by:

1. Auto-generating action creators
2. Combining actions + reducers
3. Using Immer internally (mutations look like mutations)
4. Setting up good defaults

BEFORE createSlice:
- Write action types (constants)
- Write action creators
- Write reducer with switch
- Wire up in store

AFTER createSlice:
- Define in one place
- Everything auto-generated

Code reduction: ~80% ✅
```

### Q3: How do you handle async with Redux Toolkit?

**Answer:**

```
THREE APPROACHES:

1. createAsyncThunk (Recommended)
   export const fetchProducts = createAsyncThunk(
     'products/fetch',
     async () => {
       const response = await fetch('/api/products');
       return response.json();
     }
   );

   extraReducers: (builder) => {
     builder
       .addCase(fetchProducts.pending, ...)
       .addCase(fetchProducts.fulfilled, ...)
       .addCase(fetchProducts.rejected, ...);
   }

2. Manual async in reducer + custom middleware
   (rarely used, more control)

3. Redux Saga (advanced, for complex async)
   (separate saga file)

Best practice: createAsyncThunk for 95% of cases
```

---

## Key Takeaways

1. ✅ Redux Toolkit modernizes Redux significantly
2. ✅ createSlice removes 80% of boilerplate
3. ✅ Immer middleware handles immutability
4. ✅ createAsyncThunk simplifies async
5. ✅ Better DevTools than any other solution
6. ✅ Best for large enterprise teams
7. ✅ Pair with RTK Query for API handling
8. ✅ Still necessary when standardization critical

---

**Zustand is more popular in 2026, but Redux Toolkit still dominates enterprise.**
