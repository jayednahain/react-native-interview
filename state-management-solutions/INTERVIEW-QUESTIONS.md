# ❓ State Management Interview Questions (2026)

## Table of Contents

1. [Beginner Questions](#beginner-questions)
2. [Intermediate Questions](#intermediate-questions)
3. [Advanced Questions](#advanced-questions)
4. [System Design Questions](#system-design-questions)
5. [Tricky Follow-up Questions](#tricky-follow-up-questions)

---

## Beginner Questions

### Q1: What's state management and why do we need it?

**Expected Answer:**

```
State management = How an app stores, updates, and shares data across components.

Why we need it:
1. Avoid prop drilling (passing props through 10 levels)
2. Single source of truth (no duplicate data)
3. Predictable state updates (know how state changes)
4. Easier debugging (see state history)
5. Better performance (optimize re-renders)

Simple example:
Without: useState in each component → duplicated, out-of-sync
With: Central store → consistent, synchronized
```

**Follow-up:** "What problems does state management solve?"

```
- Prop drilling: State directly accessible anywhere
- Duplicate data: Single source of truth
- Out-of-sync: Changes reflected everywhere
- Performance: Only components using state re-render
- Debugging: Middleware logs all changes
```

---

### Q2: What's the difference between client state and server state?

**Expected Answer:**

```
CLIENT STATE:
- Owned by frontend
- UI state (modal open/closed, theme, language)
- Form data (temporary values)
- User preferences (dark mode)
- Always fresh
- No sync needed
- Examples: theme, language, isModalOpen

SERVER STATE:
- Owned by backend
- User data, products, orders
- Shared across sessions
- May become stale
- Needs background sync
- Need cache management
- Examples: users, products, comments

KEY DIFFERENCE:
❌ DON'T use Redux for server state (hard to manage)
✅ DO use React Query for server state (automatic)
✅ DO use Zustand for client state (simple)
```

---

### Q3: What are the pros and cons of Context API?

**Expected Answer:**

```
PROS:
✅ Built into React (no dependency)
✅ Good for learning
✅ Perfect for small apps
✅ No providers needed (actually you do need it, but simpler than Redux)
✅ No bundle bloat

CONS:
❌ Performance: ALL consumers re-render on ANY value change
❌ No DevTools
❌ Manual async handling
❌ No middleware
❌ Boilerplate for complex state (switch statements)
❌ Not scalable to large apps

USE IT WHEN:
- Learning React
- Simple apps (< 5 global values)
- UI state (theme, language)
- Small teams

DON'T USE IT WHEN:
- Server state management needed
- Complex business logic
- Large apps
- Performance critical
```

---

### Q4: Explain this code - what's wrong?

```javascript
const AppContext = createContext();

export function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");

  const value = {
    // <-- What's wrong here?
    user,
    setUser,
    theme,
    setTheme,
  };

  return (
    <AppContext.Provider value={value}>
      <Children />
    </AppContext.Provider>
  );
}
```

**Answer:**

```
PROBLEM: New object created every render

Every render:
1. App component re-renders
2. New value object created: {} !== {}
3. Context thinks value changed
4. ALL consumers re-render
5. Causes unnecessary re-renders

SOLUTION: Memoize value

const value = useMemo(() => ({
  user,
  setUser,
  theme,
  setTheme,
}), [user, theme]);

Now:
- value only changes when user or theme changes
- Prevents unnecessary consumer re-renders
- Performance improved!
```

---

### Q5: When would you use useReducer instead of useState?

**Expected Answer:**

```
USE useState WHEN:
✅ Single value state
✅ Simple updates
✅ No complex logic

Example:
const [count, setCount] = useState(0);
const [name, setName] = useState('');

USE useReducer WHEN:
✅ Multiple related states
✅ Complex update logic
✅ Many dispatch points
✅ Predictable state transitions

Example:
const [formState, dispatch] = useReducer(formReducer, initialState);
- Multiple fields: name, email, phone, address
- Complex validation
- Dispatch from many places

BENEFITS OF useReducer:
1. Centralize complex logic
2. Easier to test
3. Pass single dispatch instead of 10 setters
4. Easier to reason about state
```

---

## Intermediate Questions

### Q6: Compare Redux vs Zustand - when would you use each?

**Expected Answer:**

```
REDUX:
✓ Large enterprise apps (20+ developers)
✓ Team already knows Redux
✓ Complex business logic
✓ Audit trail needed
✓ Time-travel debugging critical
✓ Middleware-heavy

ZUSTAND:
✓ Startup (< 20 developers)
✓ Fast development speed
✓ Less boilerplate
✓ Simple state needs
✓ Bundle size matters
✓ DX important

MODERN TREND (2026):
Most companies: Zustand + React Query
Large enterprises: Redux Toolkit + RTK Query

If I had to choose for a NEW project:
→ Zustand (better DX, less boilerplate, faster)

If I had to choose for EXISTING Redux codebase:
→ Stick with Redux Toolkit (consistency)

If starting ENTERPRISE project:
→ Redux Toolkit (standardization matters)
```

---

### Q7: How do you handle derived state (computed values)?

**Expected Answer:**

```
DERIVED STATE = Values calculated from other state

BAD: Store everything
const [items, setItems] = useState([]);
const [itemCount, setItemCount] = useState(0);
const [total, setTotal] = useState(0);
// Need to update all when items change 😫

GOOD APPROACH 1: Calculate in component
function CartComponent() {
  const items = useStore((state) => state.items);
  const total = items.reduce((sum, item) =>
    sum + item.price * item.quantity, 0);
  return <Text>${total}</Text>;
}

GOOD APPROACH 2: Use selector
const selectCartTotal = (state) =>
  state.items.reduce((sum, item) =>
    sum + item.price * item.quantity, 0);

function CartComponent() {
  const total = useStore(selectCartTotal);
  return <Text>${total}</Text>;
}

GOOD APPROACH 3: Memoize selector
const selectCartTotal = (state) =>
  useMemo(() =>
    state.items.reduce((sum, item) =>
      sum + item.price * item.quantity, 0),
    [state.items]
  );

BEST PRACTICE:
- Calculate derived values in components (recompute is fast)
- OR use memoized selectors (if expensive)
- DON'T store derived values in state
```

---

### Q8: Explain how React Query's deduplication works

**Expected Answer:**

```
DEDUPLICATION = Multiple requests for same data = Only ONE fetch

HOW IT WORKS:

1. Component A requests useQuery({queryKey: ['products']})
2. Component B requests useQuery({queryKey: ['products']})
3. React Query sees same queryKey
4. Recognizes duplicate request
5. First request starts
6. Second request waits for same response
7. Response returns
8. Both components get data
9. Only ONE fetch happened! ✅

CODE EXAMPLE:

function ProductPage() {
  const { data } = useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });
}

function ProductList() {
  const { data } = useQuery({
    queryKey: ['products'],  // SAME KEY
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });
}

RESULT:
✅ Both get products
✅ Only one fetch
✅ Same data for both
✅ Automatic deduplication

DEDUP TIME WINDOW: 0ms default
Can configure: queryClient.setQueryDefaults()
```

---

### Q9: How do you prevent unnecessary re-renders with selectors?

**Expected Answer:**

```
PROBLEM: Without selectors, component re-renders on ANY state change

BAD:
function Header() {
  const state = useStore();  // Gets entire store
  // Re-renders when items, total, loading, error change
  // Even if Header only uses loading?
}

GOOD:
function Header() {
  const loading = useStore((state) => state.loading);
  // Only re-renders when loading changes
  // Ignores items, total, error changes
  // ✅ Prevents wasted re-renders
}

HOW SELECTORS WORK:

1. Pass selector function: (state) => state.loading
2. Store evaluates selector
3. Compares result with previous: oldValue !== newValue
4. If different: notify subscriber (re-render)
5. If same: don't notify (no re-render)

ADVANCED SELECTOR PATTERNS:

// Memoized selector (for expensive computations)
const selectTotalPrice = (state) =>
  useMemo(() =>
    state.items.reduce((sum, item) =>
      sum + item.price * item.quantity, 0),
    [state.items]
  );

function Total() {
  const total = useStore(selectTotalPrice);
  return <Text>${total}</Text>;
}

// Multiple values but optimized
function Cart() {
  // Only re-render if items OR total changed
  const { items, total } = useStore((state) => ({
    items: state.items,
    total: state.total,
  }));
}

RULE: More specific selector = fewer re-renders = better performance
```

---

### Q10: Explain optimistic updates and when to use them

**Expected Answer:**

```
OPTIMISTIC UPDATE = Update UI before server confirms

FLOW:

User clicks "Add to Cart"
  ↓
UPDATE UI IMMEDIATELY (optimistic)
  ↓
Show "Added!"
  ↓
Send request to server
  ↓
If success: ✅ Keep optimistic update
If error: ❌ Rollback to previous state

CODE EXAMPLE:

useMutation({
  mutationFn: (product) =>
    fetch('/api/cart/add', { method: 'POST', body: JSON.stringify(product) }),

  onMutate: async (newProduct) => {
    // Cancel ongoing queries
    await queryClient.cancelQueries({ queryKey: ['cart'] });

    // Save previous state (for rollback)
    const previousCart = queryClient.getQueryData(['cart']);

    // Optimistically update cache
    queryClient.setQueryData(['cart'], (old) => ({
      ...old,
      items: [...old.items, newProduct],
    }));

    // Return rollback function
    return { previousCart };
  },

  onError: (error, newProduct, context) => {
    // Rollback on error
    queryClient.setQueryData(['cart'], context.previousCart);
  },

  onSuccess: () => {
    // Refetch to confirm
    queryClient.invalidateQueries({ queryKey: ['cart'] });
  },
});

UX IMPROVEMENT:
❌ Without: Wait 1-2 seconds for response → feels slow
✅ With: Instant feedback → feels fast

WHEN TO USE:
✅ Read operations (safe if error)
✅ Cart operations
✅ Favorites/likes
✗ Payment (use confirmation instead)
✗ Destructive operations (delete)
```

---

## Advanced Questions

### Q11: Design a state management solution for a complex app

**Expected Answer (be detailed):**

```
ARCHITECTURE:

Server State (React Query):
├─ Products
├─ Orders
├─ User profile
├─ Messages
└─ Notifications

Client State (Zustand):
├─ Theme
├─ Language
├─ Modal states
├─ Form values
└─ UI state

Real-time (WebSocket + React Query):
├─ Invalidate on new messages
├─ Subscribe to order updates
└─ Invalidate notifications

STORE STRUCTURE (Zustand):

export const useUIStore = create((set) => ({
  // UI State
  isDrawerOpen: false,
  isModalOpen: false,
  activeTab: 'home',

  // Theme
  isDarkMode: false,
  language: 'en',

  // Form
  formData: {},

  // Actions
  toggleDrawer: () => set(state => ({ isDrawerOpen: !state.isDrawerOpen })),
  openModal: () => set({ isModalOpen: true }),
  closeModal: () => set({ isModalOpen: false }),
  toggleTheme: () => set(state => ({ isDarkMode: !state.isDarkMode })),
}));

COMPONENTS:

// Server state
function ProductList() {
  const { data: products } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  });
}

// Client state
function Header() {
  const { isDarkMode, toggleTheme } = useUIStore();
  return <Button onPress={toggleTheme} />;
}

// Mixed
function ProductDetail({ id }) {
  const { data: product } = useQuery({
    queryKey: ['products', id],
    queryFn: () => fetchProduct(id),
  });

  const { isModalOpen, openModal } = useUIStore();

  return <View>...</View>;
}

BENEFITS:
✅ Clear separation: Server vs Client
✅ Easy to test: Mock React Query, use store
✅ Performance: Granular selectors
✅ Scalable: Modular stores
✅ Maintainable: Clear structure
```

---

### Q12: How would you migrate from Redux to Zustand?

**Expected Answer:**

```
MIGRATION STRATEGY:

PHASE 1: Coexist both (2-3 days)
├─ Add Zustand to project
├─ Keep Redux running
├─ Move UI state first (theme, modals)
└─ Test thoroughly

PHASE 2: Move non-critical state (1-2 weeks)
├─ Move cart state to Zustand
├─ Move user preferences
├─ Move form values
└─ Keep async operations in Redux

PHASE 3: Move critical state (1-2 weeks)
├─ Move user auth state
├─ Move app state
├─ Move settings
└─ Thoroughly test

PHASE 4: Remove Redux (2-3 days)
├─ Delete Redux slices
├─ Delete Redux selectors
├─ Delete Redux store
└─ Clean imports

TIMELINE: ~4-6 weeks for medium app

CODE EXAMPLE:

// Redux slice
const userSlice = createSlice({
  name: 'user',
  initialState: { user: null, loading: false },
  reducers: {
    setUser: (state, action) => {
      state.user = action.payload;
    },
  },
});

// Convert to Zustand
const useUserStore = create((set) => ({
  user: null,
  loading: false,
  setUser: (user) => set({ user }),
}));

// Usage stays mostly same
// Just update selectors/dispatches

RISKS TO MITIGATE:
⚠️ Bugs during migration
⚠️ Performance regressions
⚠️ Team confusion
→ Run both systems in parallel
→ Extensive testing
→ Good communication
```

---

### Q13: How do you handle offline state management?

**Expected Answer:**

```
OFFLINE STRATEGY:

1. DETECT OFFLINE STATE
import NetInfo from '@react-native-community/netinfo';

useEffect(() => {
  const unsubscribe = NetInfo.addEventListener(state => {
    if (!state.isConnected) {
      console.log('Offline');
      useStore.getState().setOffline(true);
    } else {
      console.log('Online');
      useStore.getState().setOffline(false);
      useStore.getState().syncOfflineChanges(); // Sync data
    }
  });
}, []);

2. QUEUE OPERATIONS OFFLINE
const useOfflineStore = create((set) => ({
  offline: false,
  queue: [],

  addToQueue: (action, payload) =>
    set(state => ({
      queue: [...state.queue, { action, payload, timestamp: Date.now() }],
    })),

  clearQueue: () => set({ queue: [] }),
}));

3. SYNC WHEN ONLINE
onSuccess: () => {
  if (isOnline()) {
    queryClient.invalidateQueries(); // Fresh sync
  }
}

4. CONFLICT RESOLUTION
// Server data is source of truth
function resolveConflict(localData, serverData) {
  if (localData.version > serverData.version) {
    return localData; // Newer local
  }
  return serverData; // Newer server
}

FULL EXAMPLE:

export const useCart = create((set) => ({
  items: [],
  synced: true,

  addItem: (product) =>
    set((state) => {
      const newItems = [...state.items, product];

      if (isOnline()) {
        // Sync immediately
        fetch('/api/cart/add', { method: 'POST', body: JSON.stringify(product) });
        return { items: newItems, synced: true };
      } else {
        // Queue for later
        useOfflineStore.getState().addToQueue('ADD_ITEM', product);
        return { items: newItems, synced: false };
      }
    }),

  syncOfflineChanges: async () => {
    const queue = useOfflineStore.getState().queue;

    for (const { action, payload } of queue) {
      if (action === 'ADD_ITEM') {
        await fetch('/api/cart/add', { method: 'POST', body: JSON.stringify(payload) });
      }
    }

    useOfflineStore.getState().clearQueue();
  },
}));
```

---

### Q14: How do you test state management?

**Expected Answer:**

```
TESTING REDUX:

// Test reducer
test('add item to cart', () => {
  const state = reducer(initialState, addItem(product));
  expect(state.items).toContain(product);
});

// Test selector
test('select cart total', () => {
  const state = { cart: { items: [...], total: 100 } };
  expect(selectCartTotal(state)).toBe(100);
});

TESTING ZUSTAND:

// Test store actions
test('add item to cart', () => {
  const { getState } = useStore;
  useStore.setState({ items: [] });
  useStore.getState().addItem(product);
  expect(getState().items).toContain(product);
});

TESTING REACT QUERY:

import { renderHook, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

test('fetch products', async () => {
  const wrapper = ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );

  const { result } = renderHook(() => useProducts(), { wrapper });

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });

  expect(result.current.data).toBeDefined();
});

TESTING ASYNC:

test('add to cart async', async () => {
  const { result } = renderHook(() => useAddToCart());

  result.current.mutate(product);

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });

  expect(cart.items).toContain(product);
});

BEST PRACTICES:
✅ Test actions separately from components
✅ Mock API calls
✅ Test selectors
✅ Test async operations
✅ Use test utilities
```

---

## System Design Questions

### Q15: Design an e-commerce app state management

**Expected Answer:**

```
STORE STRUCTURE:

SERVER STATE (React Query):
├─ useProducts() → products list
├─ useProductDetail(id) → single product
├─ useCart() → cart from server
├─ useOrders() → user's orders
├─ useReviews(productId) → product reviews
└─ useSearchResults(query) → search results

CLIENT STATE (Zustand):
├─ useUIStore
│  ├─ isDrawerOpen
│  ├─ isFilterOpen
│  ├─ searchTerm
│  └─ sortBy
├─ useThemeStore
│  ├─ isDarkMode
│  └─ language
└─ useFormStore
   ├─ checkoutForm
   └─ paymentInfo

FEATURES:

1. Product Discovery
   - useQuery(['products', { page, sort }]) → search
   - Infinite query for pagination
   - Local filters in Zustand

2. Shopping Cart
   - Server-side cart (persists)
   - React Query for sync
   - Local optimistic updates
   - Persist with Zustand

3. Checkout
   - Form state in Zustand
   - Server mutation for order
   - Optimistic update
   - Rollback on error

4. Real-time Features
   - WebSocket for inventory
   - Invalidate on order updates
   - Push notifications

5. Search & Filter
   - Debounced search
   - Local filter state
   - React Query for results
   - Infinite scroll

ESTIMATED STATE SIZE:
- Products: 100+ items → Use pagination
- Orders: 10-100 items → Paginate
- Cart: 5-50 items → Keep in state
- UI state: 10-20 values → Zustand
- Form data: 5-10 values → Zustand

PERFORMANCE OPTIMIZATIONS:
✅ Memoized selectors
✅ Pagination for large lists
✅ Infinite queries for scroll
✅ Debounced search
✅ Selective re-renders
✅ Image lazy loading
```

---

## Tricky Follow-up Questions

### Q16: What's the difference between cacheTime and staleTime in React Query?

**Expected Answer:**

```
STALE TIME: How long until data is considered "stale"
- Default: 0 (always stale)
- After staleTime: Background refetch triggered
- Component still shows old data while refetching
- User sees instant data + fresh data in background

CACHE TIME (now called gcTime): How long to keep unused data
- Default: 5 minutes
- After gcTime: Remove from cache if not used
- Frees up memory
- Next query = fresh fetch

ANALOGY:

staleTime = "Sell by date" on milk
- After staleTime: Milk might be old
- But still edible (use it)
- Check if newer available

cacheTime = "Expiration date"
- After cacheTime: Throw out the milk
- Can't use anymore

TYPICAL SETUP:

staleTime: 1000 * 60 * 5  // 5 min (fresh)
cacheTime: 1000 * 60 * 10  // 10 min (keep in cache)

TIMELINE:

00:00 - Fetch products
00:00-05:00 - Fresh (staleTime)
05:01 - Stale, background refetch starts
05:01 - Still show old data + refetch
05:05 - Refetch complete, new data
10:00 - Garbage collect, remove from cache

EXAMPLES:

useQuery({
  queryKey: ['products'],
  queryFn: () => fetch('/api/products').then(r => r.json()),
  staleTime: 0,        // Always refetch
  gcTime: 1000 * 60 * 60 * 24, // Keep 24 hours
});

// vs

useQuery({
  queryKey: ['user'],
  queryFn: () => fetch('/api/user').then(r => r.json()),
  staleTime: Infinity, // Never refetch (manual only)
  gcTime: 1000 * 60 * 10, // 10 min cache
});
```

---

### Q17: When would you choose Redux over Zustand?

**Expected Answer:**

```
REDUX IS BETTER WHEN:

1. LARGE TEAM (20+ developers)
   Why: Standardization, consistency
   Zustand: Everyone codes differently

2. AUDIT TRAIL REQUIRED
   Why: Redux devtools show action history
   Zustand: No central log

3. COMPLEX BUSINESS LOGIC
   Why: Middleware can intercept, validate, transform
   Zustand: Direct mutations, no middleware

4. TIME-TRAVEL DEBUGGING CRITICAL
   Why: Redux DevTools has perfect replay
   Zustand: Limited devtools

5. EXISTING REDUX ECOSYSTEM
   Why: Don't rewrite, improve incrementally
   Zustand: Would need rewrite

6. REGULATORY REQUIREMENTS
   Why: Audit trail for compliance
   Zustand: Can't prove what changed when

7. MICRO-FRONTEND ARCHITECTURE
   Why: Each MFE can have own Redux
   Zustand: Harder to isolate

EXAMPLE COMPANY:

Banking app (high regulation):
- Need audit trail for all transactions
- Compliance requirements
- Large team (50+ developers)
- Complex business logic
→ Redux Toolkit is right choice

vs

Startup app:
- Move fast, iterate
- Small team (3 developers)
- Simple state needs
- No audit requirements
→ Zustand + React Query is right choice

HONEST TAKE (2026):
Most new projects choose Zustand
Redux still used in:
- Enterprise apps
- Large teams
- Existing codebases
- Projects where standardization critical

If I had to start fresh: Zustand 🚀
If existing Redux: Improve with RTK ✅
```

---

### Q18: How do you handle global error state?

**Expected Answer:**

```
APPROACH 1: Centralized error store

const useErrorStore = create((set) => ({
  errors: {},  // { [errorId]: error }

  addError: (error, id = Date.now()) =>
    set(state => ({
      errors: { ...state.errors, [id]: error },
    })),

  removeError: (id) =>
    set(state => {
      const { [id]: _, ...rest } = state.errors;
      return { errors: rest };
    }),

  clearErrors: () => set({ errors: {} }),
}));

// Use in component
function App() {
  const errors = useErrorStore((state) => state.errors);

  return (
    <>
      {Object.values(errors).map(error => (
        <ErrorToast key={error.id} error={error} />
      ))}
    </>
  );
}

// Use in mutation
const mutation = useMutation({
  mutationFn: ...,
  onError: (error) => {
    useErrorStore.getState().addError({
      id: Date.now(),
      message: error.message,
      type: 'error',
    });
  },
});

APPROACH 2: Redux with middleware

const errorSlice = createSlice({
  name: 'errors',
  initialState: [],
  reducers: {
    addError: (state, action) => {
      state.push(action.payload);
    },
    removeError: (state, action) => {
      return state.filter(e => e.id !== action.payload);
    },
  },
});

// Global error middleware
const errorMiddleware = (store) => (next) => (action) => {
  try {
    return next(action);
  } catch (error) {
    store.dispatch(addError({
      message: error.message,
      timestamp: Date.now(),
    }));
  }
};

APPROACH 3: React Query + Toast

const useAddToCart = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ...,
    onError: (error) => {
      // Show toast
      Toast.show({ text: error.message, duration: 3000 });
      // Also log to store
      useErrorStore.getState().addError(error);
    },
  });
};
```

---

**Good luck with your interviews! You now understand state management deeply. 🚀**
