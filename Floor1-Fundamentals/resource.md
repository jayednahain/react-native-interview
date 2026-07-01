# Floor 1 — Fundamentals

> **Prerequisites:** None. This is the foundation everything else is built on.
> **You already know:** Most of this as a React Native developer. This floor names it properly and fills gaps.

---

## 1.1 How Data Moves — The Request/Response Cycle

Every time your React Native app calls an API, this exact sequence happens:

1. Your app builds an HTTP request (method, URL, headers, body)
2. The request travels over TCP/IP to a server
3. The server processes it and sends back an HTTP response (status code, headers, body)
4. Your app reads the response and updates the UI

### In React Native — using `fetch`

```javascript
// GET request — read data
const getUser = async (id) => {
  const response = await fetch(`https://api.myapp.com/users/${id}`, {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer eyJhbGciOiJIUzI1NiJ9...',
      'Content-Type': 'application/json',
    },
  });

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  const user = await response.json();
  return user;
};

// POST request — create data
const createUser = async (userData) => {
  const response = await fetch('https://api.myapp.com/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer eyJhbGciOiJIUzI1NiJ9...',
    },
    body: JSON.stringify(userData),
  });

  return response.json();
};

// PATCH request — partially update
const updateUserName = async (id, newName) => {
  const response = await fetch(`https://api.myapp.com/users/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name: newName }),
  });
  return response.json();
};

// DELETE request
const deleteUser = async (id) => {
  await fetch(`https://api.myapp.com/users/${id}`, { method: 'DELETE' });
};
```

### HTTP Methods

| Method | Purpose | Has Body? | Idempotent? |
|--------|---------|-----------|-------------|
| GET | Read data | No | Yes |
| POST | Create data | Yes | No |
| PUT | Replace entire resource | Yes | Yes |
| PATCH | Partially update resource | Yes | No |
| DELETE | Remove resource | No | Yes |

> **Idempotent** means calling it multiple times has the same effect as calling it once. DELETE /users/1 twice is the same as once — the user is gone.

### HTTP Status Codes

| Code | Meaning | When you see it |
|------|---------|----------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE (no body returned) |
| 400 | Bad Request | You sent invalid or malformed data |
| 401 | Unauthorized | Missing or invalid authentication token |
| 403 | Forbidden | Token is valid but you lack permission |
| 404 | Not Found | Resource does not exist |
| 409 | Conflict | Duplicate resource (e.g. email already taken) |
| 422 | Unprocessable Entity | Validation failed (field format wrong) |
| 429 | Too Many Requests | Rate limited — slow down |
| 500 | Internal Server Error | Bug on the server |
| 502 | Bad Gateway | Upstream server failed |
| 503 | Service Unavailable | Server is down or overloaded |

### Headers

Headers are metadata on every request and response.

```javascript
// Common request headers your app sends
const headers = {
  'Content-Type': 'application/json',      // Body format you're sending
  'Authorization': 'Bearer <token>',        // Who you are
  'Accept': 'application/json',             // Format you expect back
  'Accept-Language': 'en-US',              // Language preference
  'Cache-Control': 'no-cache',             // Don't serve me stale cache
  'X-App-Version': '3.2.1',               // Custom: your app version
  'X-Request-ID': 'uuid-v4-here',         // Custom: trace this request
};

// Common response headers the server sends back
// Content-Type: application/json
// Cache-Control: max-age=3600
// X-RateLimit-Remaining: 99
// X-RateLimit-Reset: 1688212800
```

---

## 1.2 Networking Basics

### TCP/IP — How Data Actually Travels

**TCP (Transmission Control Protocol)** makes data travel reliably across the internet.

- Data is split into **packets** (small numbered chunks)
- Packets travel independently, potentially via different routes
- TCP guarantees they arrive **in order** and **without loss**
- If a packet is dropped, TCP automatically requests retransmission

**IP (Internet Protocol)** handles **addressing** — every device has an IP address.

```
React Native App (192.168.1.5 — private)
  → Your Router (public IP: 203.0.113.1)
    → Internet backbone
      → Server (54.201.45.23)
```

**UDP** is faster but unreliable (no guaranteed delivery). Used for video calls, games, and DNS where speed matters more than perfection.

### DNS — The Internet's Phone Book

DNS translates human-readable names into IP addresses.

```
api.myapp.com  →  DNS lookup  →  54.201.45.23
```

When your app calls `fetch('https://api.myapp.com/users')`:
1. OS checks the local DNS cache
2. If not cached, asks the configured DNS server (e.g. 8.8.8.8 — Google DNS)
3. DNS responds: `54.201.45.23`
4. Your device connects to that IP on port 443

DNS results are cached with a **TTL (Time To Live)**. When a company migrates servers, they update DNS and wait for TTL to expire before the old IP becomes unused.

### TLS/HTTPS — Encrypted Connections

TLS (Transport Layer Security) encrypts data in transit. HTTPS = HTTP + TLS.

**Without TLS:** Data travels as plain text. Anyone on your Wi-Fi network can intercept it.

**With TLS:**
1. Client and server perform a **TLS handshake** — agree on encryption keys
2. Server presents a **certificate** (issued by a Certificate Authority) proving its identity
3. All data is encrypted — unreadable in transit
4. Man-in-the-middle attacks are prevented

```javascript
// React Native certificate pinning — extra security for sensitive apps
// Prevents rogue CAs from issuing fake certificates for your domain
// Libraries: react-native-ssl-pinning

import { fetch } from 'react-native-ssl-pinning';

const response = await fetch('https://api.myapp.com/users', {
  method: 'GET',
  sslPinning: {
    certs: ['sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA='],
  },
});
```

Always use `https://` in production. Never `http://`.

### Latency vs Bandwidth

| Concept | Definition | Real Example |
|---------|-----------|-------------|
| **Latency** | Time for one request to complete (ms) | 50ms on LTE, 200ms+ on 3G |
| **Bandwidth** | How much data transfers per second (Mbps) | 100Mbps on 5G, 5Mbps on 3G |

For mobile UX:
- **Latency** dominates interactive requests (user taps → sees result)
- **Bandwidth** dominates bulk downloads (images, videos, large JSON)

A fast 100Mbps network with 500ms latency still feels slow for user interactions. This is why API response time matters more than payload size for most features.

### WebSockets — Persistent Two-Way Connections

HTTP is request/response — your app must initiate. WebSockets keep a persistent connection so the **server can push data** at any time.

```javascript
// WebSocket in React Native
const ws = new WebSocket('wss://api.myapp.com/chat');

ws.onopen = () => {
  console.log('Connection established');
  ws.send(JSON.stringify({ type: 'subscribe', channel: 'notifications' }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);

  if (data.type === 'new_message') {
    appendMessage(data.message);
  } else if (data.type === 'user_typing') {
    showTypingIndicator(data.userId);
  }
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error.message);
};

ws.onclose = (event) => {
  console.log('Disconnected. Code:', event.code);
  // Reconnect with exponential backoff
  setTimeout(reconnect, 2000);
};

// Clean up when component unmounts
useEffect(() => {
  return () => ws.close();
}, []);
```

Use WebSockets for: live chat, real-time notifications, collaborative features, multiplayer.

---

## 1.3 Data Structures You Must Know

These appear in system design discussions, interview questions, and daily JavaScript code.

### Array / List

Ordered collection. O(1) access by index. O(n) search.

```javascript
const users = ['Alice', 'Bob', 'Charlie'];

// Access by index — O(1)
const first = users[0]; // 'Alice'

// Add to end — O(1) amortized
users.push('Diana');

// Add to front — O(n) — every element shifts. Avoid in hot paths.
users.unshift('Zara');

// Remove from end — O(1)
users.pop();

// Remove from front — O(n)
users.shift();

// Search — O(n) — scans every element
const index = users.indexOf('Bob'); // 1

// Higher-order functions — very common in React Native
const names = users.map(u => u.toUpperCase());           // ['ALICE', 'BOB', ...]
const filtered = users.filter(u => u.startsWith('A'));    // ['Alice']
const found = users.find(u => u === 'Bob');               // 'Bob'
const hasAlice = users.some(u => u === 'Alice');          // true
const allStrings = users.every(u => typeof u === 'string'); // true
const total = [1, 2, 3].reduce((sum, n) => sum + n, 0);  // 6
```

### Hash Map / Object / Map

Key → value lookup. O(1) average for get/set/delete.

```javascript
// Plain object — most common for string keys
const userById = {
  'user_1': { name: 'Alice', age: 30 },
  'user_2': { name: 'Bob', age: 25 },
};

const alice = userById['user_1']; // O(1) — no scanning needed
userById['user_3'] = { name: 'Charlie', age: 28 };
delete userById['user_2'];
'user_1' in userById; // true

// Map — better when keys are not strings, or order matters
const requestCache = new Map();
requestCache.set('/api/users', { data: [], cachedAt: Date.now() });
requestCache.get('/api/users'); // O(1)
requestCache.has('/api/users'); // true
requestCache.delete('/api/users');

// Set — unique values only
const seen = new Set();
const deduped = items.filter(item => {
  if (seen.has(item.id)) return false;
  seen.add(item.id);
  return true;
});
```

**When to use which:**
- `{}` — simple string-keyed lookups, JSON data
- `Map` — non-string keys, need to track insertion order, frequent add/delete
- `Set` — deduplication, membership checks

### Queue — First In, First Out (FIFO)

```javascript
class Queue {
  #items = [];

  enqueue(item) { this.#items.push(item); }       // Add to back — O(1)
  dequeue() { return this.#items.shift(); }         // Remove from front — O(n)
  peek() { return this.#items[0]; }
  get size() { return this.#items.length; }
  isEmpty() { return this.#items.length === 0; }
}

// Real use case — offline request queue in React Native
const requestQueue = new Queue();

const makeRequest = async (config) => {
  if (!isOnline) {
    requestQueue.enqueue(config); // Save for later
    return;
  }
  return fetch(config.url, config.options);
};

// When connection restores, drain the queue
NetInfo.addEventListener(async (state) => {
  if (state.isConnected) {
    while (!requestQueue.isEmpty()) {
      const config = requestQueue.dequeue();
      await fetch(config.url, config.options);
    }
  }
});
```

### Stack — Last In, First Out (LIFO)

```javascript
class Stack {
  #items = [];

  push(item) { this.#items.push(item); }          // Add to top — O(1)
  pop() { return this.#items.pop(); }               // Remove from top — O(1)
  peek() { return this.#items[this.#items.length - 1]; }
  get size() { return this.#items.length; }
  isEmpty() { return this.#items.length === 0; }
}

// React Navigation uses a stack internally
const navHistory = new Stack();
navHistory.push('HomeScreen');     // Navigate to Home
navHistory.push('FeedScreen');     // Navigate to Feed
navHistory.push('PostDetailScreen'); // Navigate to Post

navHistory.pop();   // Back → FeedScreen
navHistory.peek();  // Current screen: FeedScreen

// Undo/redo pattern
const undoStack = new Stack();
const redoStack = new Stack();

const applyChange = (change) => {
  undoStack.push(change);
  redoStack = new Stack(); // Clear redo on new action
  performChange(change);
};

const undo = () => {
  if (!undoStack.isEmpty()) {
    const change = undoStack.pop();
    redoStack.push(change);
    revertChange(change);
  }
};
```

### Tree

Hierarchical structure. Each node has exactly one parent (except root) and zero or more children.

```javascript
// Building a component tree representation
const componentTree = {
  name: 'App',
  children: [
    {
      name: 'NavigationContainer',
      children: [
        {
          name: 'HomeScreen',
          children: [
            { name: 'Header', children: [] },
            { name: 'FlatList', children: [] },
          ]
        }
      ]
    }
  ]
};

// Depth-first traversal (pre-order) — visits parent before children
function traverseDFS(node, callback) {
  callback(node);
  node.children.forEach(child => traverseDFS(child, callback));
}

// Breadth-first traversal — visits level by level
function traverseBFS(root, callback) {
  const queue = [root];
  while (queue.length > 0) {
    const node = queue.shift();
    callback(node);
    queue.push(...node.children);
  }
}

// Binary Search Tree — sorted data, O(log n) search
class BSTNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
  }
}
```

React's virtual DOM is a tree. When state changes, React performs a diffing algorithm on two trees to find the minimal set of DOM updates. This is why component structure matters for performance.

### Graph

Nodes connected by edges. Can be directed (one-way) or undirected (two-way). Trees are special graphs.

```javascript
// Adjacency list — most common representation
const socialGraph = new Map([
  ['alice',   ['bob', 'charlie']],
  ['bob',     ['alice', 'diana']],
  ['charlie', ['alice']],
  ['diana',   ['bob']],
]);

// BFS — find shortest connection path ("6 degrees of separation")
function shortestPath(graph, start, target) {
  const queue = [[start]];
  const visited = new Set([start]);

  while (queue.length > 0) {
    const path = queue.shift();
    const current = path[path.length - 1];

    if (current === target) return path;

    for (const neighbor of graph.get(current) ?? []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push([...path, neighbor]);
      }
    }
  }
  return null;
}

shortestPath(socialGraph, 'alice', 'diana');
// Returns: ['alice', 'bob', 'diana']
```

Real-world graph use cases: social networks, route navigation, dependency resolution (npm package tree), recommendation engines.

---

## 1.4 What You Already Know as a React Native Developer

You already handle system design problems that backend engineers outsource to infrastructure.

### Local Storage

```javascript
// AsyncStorage — async key-value store (equivalent to browser localStorage)
import AsyncStorage from '@react-native-async-storage/async-storage';

// Write
await AsyncStorage.setItem('@user_prefs', JSON.stringify({ theme: 'dark' }));

// Read
const raw = await AsyncStorage.getItem('@user_prefs');
const prefs = raw ? JSON.parse(raw) : null;

// Delete
await AsyncStorage.removeItem('@user_prefs');

// MMKV — synchronous, 10x faster than AsyncStorage. Preferred for production.
import { MMKV } from 'react-native-mmkv';
const storage = new MMKV();
storage.set('auth.token', 'eyJhbGci...');
const token = storage.getString('auth.token');

// Secure Storage — for tokens and secrets (uses iOS Keychain / Android Keystore)
import * as SecureStore from 'expo-secure-store';
await SecureStore.setItemAsync('access_token', token);
const token = await SecureStore.getItemAsync('access_token');
```

### Network Connectivity & Resilience

```javascript
import NetInfo from '@react-native-community/netinfo';

// Subscribe to network changes
const unsubscribe = NetInfo.addEventListener(state => {
  console.log('Connected:', state.isConnected);
  console.log('Type:', state.type); // 'wifi', 'cellular', 'none'
  console.log('Is fast:', state.details?.isConnectionExpensive === false);
});

// Retry logic with exponential backoff
const fetchWithRetry = async (url, options, retries = 3) => {
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      const response = await fetch(url, options);
      if (!response.ok && response.status >= 500) throw new Error('Server error');
      return response;
    } catch (error) {
      if (attempt === retries - 1) throw error;
      const delay = Math.pow(2, attempt) * 1000; // 1s, 2s, 4s
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};
```

### App Lifecycle

```javascript
import { AppState } from 'react-native';

useEffect(() => {
  const subscription = AppState.addEventListener('change', nextState => {
    if (nextState === 'active') {
      // App came to foreground — refresh stale data, resume timers
      refetchUserData();
    } else if (nextState === 'background') {
      // App went to background — save state, pause timers
      saveCurrentState();
    }
  });

  return () => subscription.remove();
}, []);
```

### What This Means for System Design

| Mobile Challenge | System Design Concept |
|-----------------|----------------------|
| Offline support + request queue | Asynchronous message queuing |
| Local cache (MMKV/SQLite) | Caching layer |
| Retry logic + backoff | Fault tolerance and resilience |
| FlatList virtualization | Pagination and lazy loading |
| JWT in SecureStore | Secure authentication token storage |
| Network state detection | Circuit breaker pattern |

---

## 1.5 JavaScript-Specific Concepts Every React Native Dev Must Know

These are unique to JS and directly affect how your React Native app performs.

### The Event Loop

JavaScript is **single-threaded** — one piece of code runs at a time. The event loop manages what runs next.

```
┌─────────────────────────────────────┐
│           Call Stack                │  ← Synchronous code executes here
│  (currently executing function)     │
└─────────────────────────────────────┘
          ↓ when empty
┌─────────────────────────────────────┐
│        Microtask Queue              │  ← Promises, async/await, queueMicrotask()
│  (Promise.then, async/await)        │  ← Drains completely before next macro-task
└─────────────────────────────────────┘
          ↓ when empty
┌─────────────────────────────────────┐
│        Macro-task Queue             │  ← setTimeout, setInterval, I/O, UI events
│  (setTimeout, setInterval, fetch)   │  ← One task per event loop iteration
└─────────────────────────────────────┘
```

```javascript
console.log('1 — synchronous, runs immediately');

setTimeout(() => console.log('4 — macro-task'), 0);

Promise.resolve()
  .then(() => console.log('3 — microtask, runs before macro-tasks'));

console.log('2 — synchronous, runs immediately');

// Output order: 1 → 2 → 3 → 4
```

**Why this matters in React Native:**
- Synchronous loops with thousands of iterations **freeze** the JS thread → UI jank
- Use `requestAnimationFrame` for animation-timed work
- Use `InteractionManager.runAfterInteractions()` to defer heavy work until animations finish

```javascript
import { InteractionManager } from 'react-native';

// Don't process 10,000 items immediately — defer until after screen transition
InteractionManager.runAfterInteractions(() => {
  processMassiveDataset(data);
});
```

### Closures

A closure is a function that remembers variables from its outer scope even after the outer function has returned.

```javascript
// This is a closure — the timer callback captures 'count' from the enclosing scope
const makeCounter = (start = 0) => {
  let count = start; // This variable is "closed over"

  return {
    increment: () => ++count,
    decrement: () => --count,
    value: () => count,
  };
};

const counter = makeCounter(10);
counter.increment(); // 11
counter.increment(); // 12
counter.value();     // 12

// Common React Native closure gotcha — stale state in event listeners
const [count, setCount] = useState(0);

useEffect(() => {
  const ws = new WebSocket('wss://...');
  ws.onmessage = () => {
    // WARNING: 'count' here is stale — it's the value at the time this effect ran
    console.log(count); // Always logs initial value

    // FIX: use functional update or useRef
    setCount(prev => prev + 1);
  };
  return () => ws.close();
}, []); // Empty deps = runs once = stale closure
```

### Promises and Async/Await

```javascript
// Promise — represents a future value
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve('data'), 1000);
});

// Chain multiple async operations
fetch('/api/user/1')
  .then(response => {
    if (!response.ok) throw new Error(response.statusText);
    return response.json();
  })
  .then(user => setUser(user))
  .catch(error => showError(error.message))
  .finally(() => setLoading(false));

// Async/await — syntactic sugar over Promises, same behavior
const loadUserProfile = async (userId) => {
  setLoading(true);
  try {
    const [user, posts] = await Promise.all([
      fetch(`/api/users/${userId}`).then(r => r.json()),
      fetch(`/api/users/${userId}/posts`).then(r => r.json()),
    ]);
    setProfile({ user, posts });
  } catch (error) {
    setError(error.message);
  } finally {
    setLoading(false);
  }
};

// Promise combinators
await Promise.all([p1, p2, p3]);         // All must succeed; fails if any fail
await Promise.allSettled([p1, p2, p3]);  // All run; get success/failure for each
await Promise.race([p1, p2, p3]);        // Resolves/rejects with the first to finish
await Promise.any([p1, p2, p3]);         // Resolves with first success; fails only if all fail
```

### React Native Thread Model

React Native runs across three threads:

| Thread | Purpose |
|--------|---------|
| **JS Thread** | Your JavaScript code, business logic, React rendering logic |
| **UI Thread (Main)** | Native view rendering, system gestures, UIKit/Android View |
| **Shadow Thread** | Yoga layout calculations (Flexbox) |

The JS bridge (old arch) or JSI (new arch) handles communication between threads.

```javascript
// Animations on JS thread — can drop frames if JS thread is busy
Animated.timing(opacity, { toValue: 1, useNativeDriver: false }).start();

// Animations on UI thread — 60fps even if JS is busy
Animated.timing(opacity, { toValue: 1, useNativeDriver: true }).start();

// Reanimated 2/3 — worklets run entirely on the UI thread
import Animated, { useSharedValue, withTiming } from 'react-native-reanimated';
const opacity = useSharedValue(0);
opacity.value = withTiming(1); // Runs on UI thread, no JS involvement
```

---

## Summary

| Concept | Key Takeaway |
|---------|-------------|
| HTTP | Request → Server → Response. Know methods, status codes, headers. |
| TCP/IP | Packets travel reliably across the internet. Data arrives in order. |
| DNS | `api.myapp.com` → IP address. Cached with TTL. |
| TLS/HTTPS | Always encrypted. Certificate proves server identity. |
| Latency | Time per request. More UX impact than bandwidth. |
| Array | Ordered list. O(1) index access, O(n) search. |
| Map/Object | Key→value. O(1) lookup. Use for caches and indexes. |
| Queue | FIFO. Use for task queues, offline request queues. |
| Stack | LIFO. Use for navigation history, undo/redo. |
| Tree | Hierarchy. React's DOM is a tree. DFS vs BFS traversal. |
| Graph | Nodes + edges. BFS for shortest path. |
| Event loop | Single-threaded. Microtasks before macro-tasks. Avoid blocking. |
| Async/await | Promises under the hood. Use `Promise.all` for parallel requests. |
| RN threads | JS + UI + Shadow. Keep heavy work off JS thread. |

---

*Next: Floor 2 — OOP and Design Patterns*
