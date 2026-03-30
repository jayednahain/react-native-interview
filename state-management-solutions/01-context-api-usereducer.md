# 📌 Context API + useReducer - Shopping Cart Example

## Overview

**Context API** is React's built-in solution for sharing state without prop drilling.  
**useReducer** is used for complex state logic.

Combined, they provide a lightweight alternative to external libraries.

---

## Simple Definition

**Context API + useReducer** = Sharing global state with local components without passing props through every level.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────┐
│         App Component                       │
│  ┌───────────────────────────────────────┐  │
│  │   ShoppingCartProvider (Context)      │  │
│  │  ┌─────────────────────────────────┐  │  │
│  │  │  State (useReducer)             │  │  │
│  │  │  ├─ items: []                   │  │  │
│  │  │  ├─ total: 0                    │  │  │
│  │  │  ├─ loading: false              │  │  │
│  │  │  └─ error: null                 │  │  │
│  │  │                                 │  │  │
│  │  │  Reducer Function               │  │  │
│  │  │  ├─ ADD_ITEM                    │  │  │
│  │  │  ├─ REMOVE_ITEM                 │  │  │
│  │  │  ├─ UPDATE_QUANTITY             │  │  │
│  │  │  └─ CLEAR_CART                  │  │  │
│  │  └─────────────────────────────────┘  │  │
│  │                                       │  │
│  │   Child Components (useContext)      │  │
│  │  ┌──────────┬──────────┬──────────┐  │  │
│  │  │ Header  │  Products│  Footer  │  │  │
│  │  │cart icon│ display  │ checkout │  │  │
│  │  └──────────┴──────────┴──────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

---

## Implementation: Shopping Cart

### Step 1: Define Initial State

```javascript
// cartContext.js

export const initialState = {
  items: [],
  total: 0,
  loading: false,
  error: null,
};
```

### Step 2: Define Actions

```javascript
// Action Types
export const CART_ACTIONS = {
  ADD_ITEM: "ADD_ITEM",
  REMOVE_ITEM: "REMOVE_ITEM",
  UPDATE_QUANTITY: "UPDATE_QUANTITY",
  CLEAR_CART: "CLEAR_CART",
  SET_LOADING: "SET_LOADING",
  SET_ERROR: "SET_ERROR",
};
```

### Step 3: Create Reducer Function

```javascript
// cartReducer.js

import { CART_ACTIONS } from "./cartContext";

export function cartReducer(state, action) {
  switch (action.type) {
    // Add item to cart
    case CART_ACTIONS.ADD_ITEM: {
      const existingItem = state.items.find(
        (item) => item.id === action.payload.id,
      );

      let updatedItems;
      if (existingItem) {
        // Item already in cart, increase quantity
        updatedItems = state.items.map((item) =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + 1 }
            : item,
        );
      } else {
        // New item, add to cart
        updatedItems = [...state.items, { ...action.payload, quantity: 1 }];
      }

      const newTotal = updatedItems.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0,
      );

      return {
        ...state,
        items: updatedItems,
        total: parseFloat(newTotal.toFixed(2)),
        error: null,
      };
    }

    // Remove item from cart
    case CART_ACTIONS.REMOVE_ITEM: {
      const filteredItems = state.items.filter(
        (item) => item.id !== action.payload,
      );

      const newTotal = filteredItems.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0,
      );

      return {
        ...state,
        items: filteredItems,
        total: parseFloat(newTotal.toFixed(2)),
        error: null,
      };
    }

    // Update quantity of item
    case CART_ACTIONS.UPDATE_QUANTITY: {
      const { itemId, quantity } = action.payload;

      if (quantity <= 0) {
        // If quantity is 0, remove item
        return cartReducer(state, {
          type: CART_ACTIONS.REMOVE_ITEM,
          payload: itemId,
        });
      }

      const updatedItems = state.items.map((item) =>
        item.id === itemId ? { ...item, quantity } : item,
      );

      const newTotal = updatedItems.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0,
      );

      return {
        ...state,
        items: updatedItems,
        total: parseFloat(newTotal.toFixed(2)),
        error: null,
      };
    }

    // Clear entire cart
    case CART_ACTIONS.CLEAR_CART: {
      return {
        ...state,
        items: [],
        total: 0,
        error: null,
      };
    }

    // Set loading state
    case CART_ACTIONS.SET_LOADING: {
      return {
        ...state,
        loading: action.payload,
      };
    }

    // Set error
    case CART_ACTIONS.SET_ERROR: {
      return {
        ...state,
        error: action.payload,
      };
    }

    default:
      return state;
  }
}
```

### Step 4: Create Context and Provider

```javascript
// cartContext.js

import React, { createContext, useReducer } from "react";
import { cartReducer, initialState } from "./cartReducer";
import { CART_ACTIONS } from "./constants";

export const CartContext = createContext();

export function ShoppingCartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  // Action creators for clean dispatch calls
  const value = {
    // State
    items: state.items,
    total: state.total,
    loading: state.loading,
    error: state.error,

    // Actions
    addItem: (product) => {
      dispatch({
        type: CART_ACTIONS.ADD_ITEM,
        payload: product,
      });
    },

    removeItem: (itemId) => {
      dispatch({
        type: CART_ACTIONS.REMOVE_ITEM,
        payload: itemId,
      });
    },

    updateQuantity: (itemId, quantity) => {
      dispatch({
        type: CART_ACTIONS.UPDATE_QUANTITY,
        payload: { itemId, quantity },
      });
    },

    clearCart: () => {
      dispatch({ type: CART_ACTIONS.CLEAR_CART });
    },

    setLoading: (loading) => {
      dispatch({
        type: CART_ACTIONS.SET_LOADING,
        payload: loading,
      });
    },

    setError: (error) => {
      dispatch({
        type: CART_ACTIONS.SET_ERROR,
        payload: error,
      });
    },
  };

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}
```

### Step 5: Create Custom Hook

```javascript
// useCart.js

import { useContext } from "react";
import { CartContext } from "./cartContext";

export function useCart() {
  const context = useContext(CartContext);

  if (!context) {
    throw new Error("useCart must be used within ShoppingCartProvider");
  }

  return context;
}
```

### Step 6: Wrap App with Provider

```javascript
// App.js

import React from "react";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { ShoppingCartProvider } from "./context/cartContext";
import ProductsScreen from "./screens/ProductsScreen";
import CartScreen from "./screens/CartScreen";

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <ShoppingCartProvider>
      <NavigationContainer>
        <Stack.Navigator>
          <Stack.Screen name="Products" component={ProductsScreen} />
          <Stack.Screen name="Cart" component={CartScreen} />
        </Stack.Navigator>
      </NavigationContainer>
    </ShoppingCartProvider>
  );
}
```

### Step 7: Use in Components

```javascript
// ProductsScreen.js

import React from "react";
import {
  View,
  ScrollView,
  TouchableOpacity,
  Text,
  StyleSheet,
} from "react-native";
import { useCart } from "../hooks/useCart";

const PRODUCTS = [
  { id: 1, name: "Laptop", price: 999.99 },
  { id: 2, name: "Phone", price: 799.99 },
  { id: 3, name: "Tablet", price: 499.99 },
  { id: 4, name: "Headphones", price: 199.99 },
];

export default function ProductsScreen() {
  const { addItem, items } = useCart();

  const itemCount = items.length;

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
// CartScreen.js

import React from "react";
import {
  View,
  ScrollView,
  TouchableOpacity,
  Text,
  StyleSheet,
  FlatList,
} from "react-native";
import { useCart } from "../hooks/useCart";

export default function CartScreen() {
  const { items, total, removeItem, updateQuantity, clearCart } = useCart();

  if (items.length === 0) {
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

---

## Interview Q&A: Context API + useReducer

### Q1: Why use Context + useReducer instead of just useState?

**Answer:**

```
useState is fine for simple state (single value).
useReducer is better when:

1. Multiple related states
   ✓ useState: 3 separate state variables (messy)
   ✓ useReducer: 1 cohesive state object (clean)

2. Complex state logic
   ✓ useState: Complex update logic scattered
   ✓ useReducer: All logic in one reducer (centralized)

3. Multiple dispatch points
   ✓ useState: Pass 10 setters down
   ✓ useReducer: Pass 1 dispatch function

Example - Form with multiple fields:

// BAD - With useState
const [name, setName] = useState('');
const [email, setEmail] = useState('');
const [phone, setPhone] = useState('');
const [address, setAddress] = useState('');
const [errors, setErrors] = useState({});
const [loading, setLoading] = useState(false);
// Now pass 8 functions down! 😱

// GOOD - With useReducer
const [formState, dispatch] = useReducer(formReducer, initialState);
// Pass dispatch to children, much cleaner
```

### Q2: What's the difference between Context API and Redux?

**Answer:**

```
CONTEXT API:
✓ Built into React (no dependency)
✓ Good for small-medium apps
✓ Simple learning curve
✗ No DevTools
✗ Performance: re-renders all subscribers
✗ No middleware system
✗ Limited to local state

REDUX:
✓ Excellent DevTools (time-travel debugging)
✓ Middleware system (logging, analytics)
✓ Better performance (selectors)
✓ Enterprise-ready
✗ Much more boilerplate
✗ Steeper learning curve
✗ Overkill for simple apps

Rule of thumb:
- < 5 state values → Context API
- > 5 state values → Redux or Zustand
- Server state → React Query
```

### Q3: How do you prevent re-rendering all components?

**Answer:**

```
Context API problem:
When ANY value in context changes, ALL consumers re-render!

const value = {
  user,      // If this changes
  theme,     // Everything re-renders
  cart,      // Even if component only uses theme!
};

Solutions:

1. SPLIT CONTEXTS (Best for Context API)
   ✓ UserContext (just user data)
   ✓ ThemeContext (just theme)
   ✓ CartContext (just cart)

   Now each component only re-renders when ITS context changes

2. USE useMemo (Optimization)
   const value = useMemo(() => ({
     user,
     theme,
     cart,
   }), [user, theme, cart]);

3. USE A LIBRARY (Zustand, Redux)
   ✓ Selectors allow fine-grained subscriptions
   ✓ Only components reading changed value re-render

Example - Split Contexts:

// UserContext
export const UserContext = createContext();
export function useUser() {
  return useContext(UserContext);
}

// ThemeContext
export const ThemeContext = createContext();
export function useTheme() {
  return useContext(ThemeContext);
}

// In App
<UserProvider>
  <ThemeProvider>
    <MainApp />
  </ThemeProvider>
</UserProvider>

// In Component
function MyComponent() {
  const user = useUser();        // Only re-renders when user changes
  const theme = useTheme();      // Only re-renders when theme changes
}
```

### Q4: How do you handle async operations with useReducer?

**Answer:**

```
Three approaches:

APPROACH 1: Handle async in component (Simplest)
function ProductsScreen() {
  const [state, dispatch] = useReducer(productsReducer, initialState);

  useEffect(() => {
    dispatch({ type: 'SET_LOADING', payload: true });

    fetch('/api/products')
      .then(res => res.json())
      .then(products => {
        dispatch({ type: 'SET_SUCCESS', payload: products });
      })
      .catch(error => {
        dispatch({ type: 'SET_ERROR', payload: error.message });
      });
  }, []);

  return <View>{/* render */}</View>;
}

APPROACH 2: Use custom async hook
function useAsync(asyncFunction) {
  const [state, dispatch] = useReducer(asyncReducer, initialState);

  useEffect(() => {
    dispatch({ type: 'PENDING' });
    asyncFunction()
      .then(data => dispatch({ type: 'SUCCESS', payload: data }))
      .catch(error => dispatch({ type: 'ERROR', payload: error }));
  }, [asyncFunction]);

  return state;
}

// Use it
function ProductsScreen() {
  const { data, loading, error } = useAsync(() =>
    fetch('/api/products').then(res => res.json())
  );
}

APPROACH 3: Async middleware pattern (Advanced)
function asyncReducer(state, action) {
  if (action.type.endsWith('_REQUEST')) {
    return { ...state, loading: true, error: null };
  }
  if (action.type.endsWith('_SUCCESS')) {
    return { ...state, loading: false, data: action.payload };
  }
  if (action.type.endsWith('_ERROR')) {
    return { ...state, loading: false, error: action.payload };
  }
  return state;
}

BEST PRACTICE FOR CONTEXT:
Move async to component level, dispatch results to context.
React Query handles this better automatically.
```

### Q5: Can Context API replace Redux?

**Answer:**

```
Short answer: For 80% of apps, YES.
For enterprise apps with 50+ developers: NO.

CONTEXT API WORKS WELL WHEN:
✓ < 5 developers
✓ Relatively simple state
✓ No need for time-travel debugging
✓ Performance is acceptable
✓ Team knows React fundamentals

REDUX IS STILL BETTER FOR:
✓ 20+ developers (standardization)
✓ Complex business logic
✓ Regulatory audit requirements
✓ Middleware-heavy apps
✓ DevTools critical for debugging

MODERN TREND IN 2026:
Context API + Zustand > Redux for most cases

Stack:
- Context API: Learning, small projects
- Zustand: Production apps (faster, less boilerplate)
- Redux: Large enterprise teams
- React Query: Always for server state
```

---

## Common Mistakes

### ❌ Mistake 1: Creating Context in Component

```javascript
// WRONG - Context created on every render!
function App() {
  const CartContext = createContext(); // 🔴 WRONG
  // ...
}

// RIGHT - Create context outside component
const CartContext = createContext(); // ✅ CORRECT
function App() {
  // ...
}
```

### ❌ Mistake 2: Not Memoizing Context Value

```javascript
// WRONG - New object every render, causes re-renders
export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  const value = {
    // 🔴 New object every render
    state,
    dispatch,
  };

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

// RIGHT - Memoize value object
export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  const value = useMemo(
    () => ({
      // ✅ Only changes when state/dispatch changes
      state,
      dispatch,
    }),
    [state, dispatch],
  );

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}
```

### ❌ Mistake 3: Storing Too Much in One Context

```javascript
// WRONG - All state in one context, everything re-renders
const value = {
  user,
  products,
  orders,
  theme,
  language,
  notifications,
  // Change ANY = entire app re-renders
};

// RIGHT - Split by concern
<UserProvider>
  <ProductsProvider>
    <ThemeProvider>
      <LanguageProvider>
        <App />
      </LanguageProvider>
    </ThemeProvider>
  </ProductsProvider>
</UserProvider>;
```

---

## Practice Exercise: Authentication Context

Build an auth context with:

- Login/logout functionality
- Token management
- Protected routes
- User profile

```javascript
// Solution below

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth must be used within AuthProvider");
  }
  return context;
}

// See full implementation in exercises folder
```

---

## Key Takeaways

1. ✅ Context API solves prop drilling
2. ✅ useReducer handles complex state logic
3. ✅ Together they're powerful for small-medium apps
4. ✅ Always memoize context value
5. ✅ Split contexts by concern for performance
6. ✅ For server state, use React Query
7. ✅ For large apps, consider Zustand or Redux
8. ✅ DevTools are limited (use Redux if critical)

---

**Next: Learn Zustand (most recommended 2026) →**
