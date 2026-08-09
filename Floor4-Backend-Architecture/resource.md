# Floor 4 — Backend Architecture

> **Prerequisites:** Floor 3 — High-Level System Design
> **Focus:** How the server side of your app actually works — the code that runs when your React Native app hits an API.

---

## 4.1 REST API Design

A well-designed REST API follows predictable conventions your app can rely on.

### Resource-Based URLs

URLs represent **things (nouns)**, not actions (verbs). The HTTP method is the action.

```
✅ Good — noun-based resources
GET    /users              → List all users
GET    /users/:id          → Get one user
POST   /users              → Create a user
PUT    /users/:id          → Replace a user entirely
PATCH  /users/:id          → Partially update a user
DELETE /users/:id          → Delete a user

❌ Bad — verb-based URLs (RPC style)
POST  /getUser
POST  /createUser
POST  /deleteUser
```

### Nested Resources

```
GET  /users/:userId/orders              → All orders for a user
GET  /users/:userId/orders/:orderId     → One specific order for a user
POST /users/:userId/orders              → Create an order for a user

GET  /posts/:postId/comments            → All comments on a post
POST /posts/:postId/comments            → Add a comment to a post
DELETE /posts/:postId/comments/:id      → Delete a specific comment
```

### Query Parameters

Use query params for filtering, sorting, searching, and pagination — not for identifying resources.

```javascript
// Filtering
GET /products?category=electronics&inStock=true

// Sorting
GET /products?sortBy=price&order=asc

// Pagination
GET /products?page=2&limit=20
// OR cursor-based (better for real-time feeds)
GET /posts?cursor=eyJpZCI6MTIzfQ&limit=20

// Search
GET /users?q=alice@example.com

// Field selection (sparse fieldsets)
GET /users/1?fields=id,name,email
```

### Versioning

APIs change. Versioning lets you evolve without breaking existing clients (like your production React Native app).

```
# URL versioning — most common
GET /v1/users
GET /v2/users    ← v2 might have a different response shape

# Header versioning
GET /users
Accept-Version: 2.0

# In React Native, always include version in your base URL:
const API_BASE = 'https://api.myapp.com/v1';
```

### Response Envelope Pattern

Consistent response shape across all endpoints.

```javascript
// Success response
{
  "success": true,
  "data": {
    "id": "user_123",
    "name": "Alice",
    "email": "alice@example.com"
  },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2025-07-01T12:00:00Z"
  }
}

// Paginated list response
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 450,
    "totalPages": 23,
    "hasNext": true,
    "nextCursor": "eyJpZCI6NDUwfQ"
  }
}

// Error response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "fields": {
      "email": ["Email is required", "Must be a valid email"]
    }
  }
}
```

### React Native API Client Example

```javascript
// A typed API client implementing the patterns above
const BASE_URL = 'https://api.myapp.com/v1';

class ApiClient {
  #token = null;

  setToken(token) { this.#token = token; }
  clearToken() { this.#token = null; }

  async #request(method, path, body = null) {
    const headers = {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
    };

    if (this.#token) {
      headers['Authorization'] = `Bearer ${this.#token}`;
    }

    const response = await fetch(`${BASE_URL}${path}`, {
      method,
      headers,
      body: body ? JSON.stringify(body) : undefined,
    });

    const json = await response.json();

    if (!response.ok) {
      throw new ApiError(json.error.code, json.error.message, response.status);
    }

    return json.data;
  }

  get(path) { return this.#request('GET', path); }
  post(path, body) { return this.#request('POST', path, body); }
  patch(path, body) { return this.#request('PATCH', path, body); }
  delete(path) { return this.#request('DELETE', path); }
}

class ApiError extends Error {
  constructor(code, message, status) {
    super(message);
    this.code = code;
    this.status = status;
  }
}

export const apiClient = new ApiClient();
```

---

## 4.2 Authentication and Authorization

**Authentication** — Who are you? Prove your identity.
**Authorization** — What are you allowed to do? Check your permissions.

These are separate concepts that work together.

### JWT — JSON Web Token

The standard for stateless authentication in mobile apps.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← Header (base64)
.
eyJ1c2VySWQiOiJ1c2VyXzEyMyIsInJvbGUiOiJ1c2VyIiwiZXhwIjoxNjg4MjEyODAwfQ  ← Payload (base64)
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature (HMAC-SHA256)
```

**Payload contains:**
```json
{
  "userId": "user_123",
  "role": "user",
  "email": "alice@example.com",
  "iat": 1688126400,   // Issued at
  "exp": 1688212800    // Expires at (24 hours later)
}
```

The server signs the token with a secret key. When the server receives a token, it verifies the signature — if valid, it trusts the payload **without querying the database**. This is what makes JWTs stateless.

> ### The JWT trade-off nobody mentions until it bites
>
> "Stateless" and "revocable" are opposites. Because the server doesn't check a
> database, it also **cannot un-issue a token**. A user taps "Log out of all devices",
> or you ban an account, or a token leaks — that token stays valid until it expires,
> no matter what your database says.
>
> The standard mitigations:
> - **Short access token lifetime** (5–15 min), so the damage window is small. This is
>   the main reason for the access/refresh split — not convenience.
> - **A revocation list** for refresh tokens (they're few and long-lived, so checking
>   them is cheap). Revoking the refresh token means the session dies at the next
>   access-token expiry.
> - **A `tokenVersion` claim** stored on the user row. Bump it to invalidate every
>   token that user holds — but this costs you a DB read per request, which trades
>   away the statelessness you chose JWTs for.
>
> Being able to say *"JWTs buy scale and cost you instant revocation, and here's how
> I'd manage that"* is worth more than reciting the three-part structure.

Two more things the JWT payload must never do:
- **Never store secrets in it.** The payload is base64, not encrypted. Anyone holding
  the token can read every claim. It is signed (tamper-proof), not hidden.
- **Never trust an unverified token on the client.** Your app can decode the payload
  to show a username or check expiry for UX, but authorisation decisions belong on
  the server, every time.

### Token Flow in React Native

```javascript
// 1. Login — exchange credentials for tokens
const login = async (email, password) => {
  const response = await fetch(`${API_BASE}/auth/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });

  const { data } = await response.json();
  // data = { accessToken, refreshToken, expiresIn }

  // 2. Store securely — NEVER in AsyncStorage for sensitive tokens
  await SecureStore.setItemAsync('access_token', data.accessToken);
  await SecureStore.setItemAsync('refresh_token', data.refreshToken);

  apiClient.setToken(data.accessToken);
};

// 3. Every API request includes the token automatically
// (handled inside ApiClient above)

// 4. Refresh access token when it expires
const refreshSession = async () => {
  const refreshToken = await SecureStore.getItemAsync('refresh_token');

  const response = await fetch(`${API_BASE}/auth/refresh`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ refreshToken }),
  });

  if (!response.ok) {
    // Refresh token expired → force logout
    logout();
    return;
  }

  const { data } = await response.json();
  await SecureStore.setItemAsync('access_token', data.accessToken);
  apiClient.setToken(data.accessToken);
};

// 5. Auto-refresh: intercept 401 responses
const requestWithAutoRefresh = async (requestFn) => {
  try {
    return await requestFn();
  } catch (error) {
    if (error.status === 401) {
      await refreshSession();
      return requestFn(); // Retry with new token
    }
    throw error;
  }
};

// 6. Logout — clear tokens
const logout = async () => {
  await SecureStore.deleteItemAsync('access_token');
  await SecureStore.deleteItemAsync('refresh_token');
  apiClient.clearToken();
  navigateToLogin();
};
```

### Access Token vs Refresh Token

| Token | Lifetime | Stored where | Purpose |
|-------|----------|-------------|---------|
| **Access Token** | Short (15 min — 1 hour) | Memory or SecureStore | Sent on every API request |
| **Refresh Token** | Long (7 — 30 days) | SecureStore only | Exchange for a new access token when it expires |

Short-lived access tokens limit damage if stolen. The refresh token is the long-lived secret.

### OAuth 2.0 — Sign in with Google/Apple

The protocol behind "Sign in with Google" and "Sign in with Apple".

```javascript
// Expo AuthSession handles the OAuth flow
import * as AuthSession from 'expo-auth-session';
import * as Google from 'expo-auth-session/providers/google';

const [request, response, promptAsync] = Google.useAuthRequest({
  clientId: 'YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com',
});

// Trigger OAuth flow
const handleGoogleSignIn = () => promptAsync();

useEffect(() => {
  if (response?.type === 'success') {
    const { authentication } = response;
    // Send the Google token to YOUR server
    // Your server validates with Google, creates/finds user, issues your own JWT
    sendGoogleTokenToServer(authentication.accessToken);
  }
}, [response]);
```

OAuth flow:
1. App opens Google login in a browser
2. User logs into Google
3. Google redirects back with an authorization code
4. App exchanges code for a Google access token
5. App sends Google token to YOUR server
6. Your server validates with Google, creates the user if new, issues your own JWT
7. App stores your JWT and uses it for all future requests

### Role-Based Access Control (RBAC)

Authorization — what each authenticated user can do.

```javascript
// JWT payload includes role
// { userId: "user_123", role: "admin" }

// Server middleware checks role before allowing access
const requireRole = (requiredRole) => (req, res, next) => {
  const user = req.user; // Set by JWT verification middleware

  if (!user) {
    return res.status(401).json({ error: { code: 'UNAUTHENTICATED' } });
  }

  if (user.role !== requiredRole && user.role !== 'admin') {
    return res.status(403).json({ error: { code: 'FORBIDDEN' } });
  }

  next();
};

// Route protection
app.delete('/users/:id', requireAuth, requireRole('admin'), deleteUserHandler);
app.get('/users/me', requireAuth, getUserProfileHandler);
```

---

## 4.3 Databases

### Relational Databases (SQL)

Data stored in tables with rows and columns. Relationships between tables via foreign keys.

```sql
-- Users table
CREATE TABLE users (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email       VARCHAR(255) UNIQUE NOT NULL,
  name        VARCHAR(255) NOT NULL,
  created_at  TIMESTAMP DEFAULT NOW()
);

-- Orders table — foreign key to users
CREATE TABLE orders (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id),
  total       DECIMAL(10, 2) NOT NULL,
  status      VARCHAR(50) DEFAULT 'pending',
  created_at  TIMESTAMP DEFAULT NOW()
);

-- Query with JOIN — get all orders for a user
SELECT
  o.id,
  o.total,
  o.status,
  u.name AS customer_name
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.id = 'user_123'
ORDER BY o.created_at DESC;
```

**ACID Properties** — what makes relational databases reliable:
- **Atomicity** — a transaction fully succeeds or fully fails (no half-written data)
- **Consistency** — the database always moves from one valid state to another
- **Isolation** — concurrent transactions don't interfere with each other
- **Durability** — committed transactions survive crashes

### Non-Relational Databases (NoSQL)

Optimized for specific access patterns that relational databases handle poorly.

```javascript
// MongoDB — document store (closest to JSON)
// Documents don't need a fixed schema
const user = {
  _id: 'user_123',
  name: 'Alice',
  email: 'alice@example.com',
  preferences: {           // Nested object — no JOIN needed
    theme: 'dark',
    notifications: { push: true, email: false }
  },
  recentSearches: ['react native', 'system design'], // Array field
};

// Firestore — real-time document database (very common in React Native)
import firestore from '@react-native-firebase/firestore';

// Read a document
const userDoc = await firestore().collection('users').doc('user_123').get();
const userData = userDoc.data();

// Real-time listener — onSnapshot is like subscribing to an observable
const unsubscribe = firestore()
  .collection('messages')
  .where('roomId', '==', 'room_456')
  .orderBy('createdAt', 'desc')
  .limit(50)
  .onSnapshot(snapshot => {
    const messages = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    setMessages(messages);
  });

// Clean up
useEffect(() => unsubscribe, []);
```

### SQL vs NoSQL — When to Use Which

| Situation | Choose |
|-----------|--------|
| Structured data with clear relationships (users, orders, products) | SQL (PostgreSQL) |
| Flexible schema that changes often | NoSQL (MongoDB) |
| Real-time updates pushed to clients | Firestore / Supabase Realtime |
| Session storage, caching, leaderboards | Redis |
| Write-heavy time-series data (logs, metrics) | Cassandra, InfluxDB |
| Full-text search | Elasticsearch |
| High-speed key-value lookups | DynamoDB, Redis |

### Indexes

Without an index, every query scans the entire table (O(n)). An index creates a sorted copy of specific columns, enabling O(log n) lookups.

```sql
-- Without index: scans ALL users to find by email
SELECT * FROM users WHERE email = 'alice@example.com'; -- Full table scan

-- Add an index on email
CREATE INDEX idx_users_email ON users(email);

-- Now the same query uses the index: O(log n) instead of O(n)
```

**When to index:**
- Columns used in WHERE clauses frequently
- Columns used in JOIN conditions
- Columns used in ORDER BY

**Trade-off:** Indexes speed up reads but slow down writes (the index must be updated on every insert/update/delete). Don't index every column.

### The N+1 Query Problem

The single most common backend performance bug, and a frequent interview question.

```javascript
// ❌ N+1 — 1 query for the orders, then 1 MORE per order for its user.
// 100 orders = 101 round trips to the database.
const orders = await db.query('SELECT * FROM orders LIMIT 100');

for (const order of orders) {
  order.user = await db.query('SELECT * FROM users WHERE id = $1', [order.user_id]);
}
```

Each query might take only 2ms, but 101 × 2ms = 200ms of pure waiting — and it gets linearly worse as data grows. Two fixes:

```javascript
// ✅ Fix 1 — JOIN. One query, the database does the work.
const orders = await db.query(`
  SELECT o.*, u.name AS user_name, u.email AS user_email
  FROM orders o
  JOIN users u ON u.id = o.user_id
  LIMIT 100
`);

// ✅ Fix 2 — batch load. Two queries total, regardless of how many orders.
const orders = await db.query('SELECT * FROM orders LIMIT 100');
const userIds = [...new Set(orders.map(o => o.user_id))];

const users = await db.query('SELECT * FROM users WHERE id = ANY($1)', [userIds]);
const usersById = new Map(users.map(u => [u.id, u]));

orders.forEach(o => { o.user = usersById.get(o.user_id); });
```

> **Where it hides:** ORMs (Prisma, TypeORM, Sequelize) and GraphQL resolvers cause
> this constantly, because the lazy `order.user` lookup looks like a property access,
> not a database call. GraphQL's standard answer is **DataLoader**, which batches all
> the per-item lookups in one tick into a single query — Fix 2, automated.
>
> As a mobile dev you often *notice* this first: an endpoint that's fast with test
> data and crawls in production is N+1 until proven otherwise.

### Connection Pooling

Opening a database connection costs ~10–50ms (TCP handshake, TLS, authentication). Doing that per request is wasteful, and databases cap total connections (Postgres defaults to ~100).

A **pool** keeps a set of open connections and lends them out:

```javascript
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                      // Max connections THIS instance holds
  idleTimeoutMillis: 30000,     // Return idle connections after 30s
  connectionTimeoutMillis: 5000, // Fail fast if the pool is exhausted
});

// Borrow → use → always return
const client = await pool.connect();
try {
  await client.query('SELECT 1');
} finally {
  client.release();   // Forgetting this leaks a connection until the pool is dry
}
```

> **The horizontal scaling trap (ties Floor 3 to Floor 5):** `max: 20` is *per
> instance*. Scale to 10 pods and you've asked for 200 connections from a database
> that allows 100 — your deployment succeeds and the database starts refusing
> connections. Either lower `max` per instance, or put a pooler like **PgBouncer**
> in front. This is a genuinely common production incident.

---

## 4.4 Transactions and Deadlocks

### Transactions

Groups multiple database operations into one **atomic unit** — all succeed or all fail.

```javascript
// Node.js + PostgreSQL — transfer money between accounts
const transferMoney = async (fromId, toId, amount) => {
  const client = await pool.connect();

  try {
    await client.query('BEGIN');

    // Check balance
    const result = await client.query(
      'SELECT balance FROM accounts WHERE id = $1 FOR UPDATE', // Lock this row
      [fromId]
    );
    const balance = result.rows[0].balance;

    if (balance < amount) {
      throw new Error('Insufficient funds');
    }

    // Debit
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromId]
    );

    // Credit
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toId]
    );

    await client.query('COMMIT'); // All or nothing
  } catch (error) {
    await client.query('ROLLBACK'); // Undo everything
    throw error;
  } finally {
    client.release();
  }
};
```

### Deadlocks

Two transactions each waiting for the other to release a lock → neither can proceed.

```
Transaction A:
  1. Locks Account 1 (to debit)
  2. Tries to lock Account 2 (to credit) → WAITING

Transaction B (concurrent, opposite direction):
  1. Locks Account 2 (to debit)
  2. Tries to lock Account 1 (to credit) → WAITING

Both wait forever. DEADLOCK.
```

**Solutions:**

```javascript
// Solution 1: Consistent lock ordering — always lock accounts in ID order
const transferMoney = async (fromId, toId, amount) => {
  // Always lock the lower ID first — prevents circular waiting
  const [firstId, secondId] = fromId < toId
    ? [fromId, toId]
    : [toId, fromId];

  await client.query('SELECT * FROM accounts WHERE id IN ($1, $2) ORDER BY id FOR UPDATE',
    [firstId, secondId]
  );
  // Now proceed with the transfer
};

// Solution 2: Timeout — if waiting too long, abort and retry
await client.query('SET lock_timeout = 5000'); // Abort if waiting > 5 seconds

// Solution 3: Optimistic locking — don't lock at all; detect conflicts after the fact
const updateWithVersion = async (id, newData, expectedVersion) => {
  const result = await client.query(
    'UPDATE users SET data = $1, version = version + 1 WHERE id = $2 AND version = $3',
    [newData, id, expectedVersion]
  );

  if (result.rowCount === 0) {
    throw new Error('Conflict — data was modified by another request. Retry.');
  }
};
```

---

## 4.5 Caching Strategies

### Cache-Aside (Lazy Loading)

The application checks the cache first. On a miss, it loads from the database and populates the cache.

```javascript
const getUser = async (userId) => {
  // 1. Check cache
  const cached = await redis.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached); // Cache hit — fast
  }

  // 2. Cache miss — load from database
  const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

  // 3. Populate cache with TTL
  await redis.setEx(`user:${userId}`, 300, JSON.stringify(user)); // 5 min TTL

  return user;
};

const updateUser = async (userId, data) => {
  await db.query('UPDATE users SET ... WHERE id = $1', [userId]);
  await redis.del(`user:${userId}`); // Invalidate cache on write
};
```

### Write-Through

Every write goes to the cache AND the database simultaneously.

```javascript
const updateUser = async (userId, data) => {
  const updatedUser = await db.query('UPDATE users SET ... WHERE id = $1 RETURNING *', [userId]);

  // Write to cache at the same time as DB
  await redis.setEx(`user:${userId}`, 300, JSON.stringify(updatedUser));

  return updatedUser;
};
// Benefit: Cache is always fresh. No stale reads after writes.
// Drawback: Every write has double latency (DB + cache).
```

### Write-Behind (Write-Back)

Write to cache immediately. Sync to database asynchronously in the background.

```javascript
const updateUserFast = async (userId, data) => {
  // Write to cache immediately — fast, returns instantly
  await redis.setEx(`user:${userId}`, 3600, JSON.stringify(data));

  // Queue a background job to persist to DB
  await queue.add('persist-user', { userId, data });

  return data;
};
// Benefit: Extremely fast writes — no DB latency.
// Risk: If the app crashes before the job runs, data is lost.
// Only use for data you can afford to lose (e.g. view counts, draft content).
```

### Cache Invalidation Strategies

| Strategy | How it works | Best for |
|----------|-------------|----------|
| **TTL expiry** | Cache entry expires after N seconds | Data that's OK to be slightly stale |
| **Event-based invalidation** | On write, explicitly delete cache keys | Data that must be fresh after changes |
| **Cache-busting** | Append version to cache key (`user:123:v2`) | Deploys where the schema changes |
| **LRU eviction** | Evict least-recently-used items when cache is full | General-purpose in-memory caches |

---

## 4.6 Message Queues

When Service A needs Service B to do work, but A shouldn't wait for B to finish.

### How It Works

```
1. Order Service processes an order.
2. Order Service publishes "order.confirmed" message to the queue.
3. Order Service returns 201 to the React Native app immediately — doesn't wait.
4. Notification Service (separately) reads the message from the queue.
5. Notification Service sends a push notification.
6. If Notification Service crashes, message stays in queue — will be processed after recovery.
```

```javascript
// Producer — Order Service publishes events
import { SQSClient, SendMessageCommand } from '@aws-sdk/client-sqs';

const sqs = new SQSClient({ region: 'us-east-1' });

const publishOrderConfirmed = async (order) => {
  await sqs.send(new SendMessageCommand({
    QueueUrl: process.env.ORDERS_QUEUE_URL,
    MessageBody: JSON.stringify({
      type: 'order.confirmed',
      payload: {
        orderId: order.id,
        userId: order.userId,
        total: order.total,
        items: order.items,
      },
    }),
    MessageGroupId: order.userId, // FIFO queue — process per user in order
  }));
};

// Consumer — Notification Service reads and processes
const processMessage = async (message) => {
  const { type, payload } = JSON.parse(message.Body);

  switch (type) {
    case 'order.confirmed':
      await pushNotificationService.send(payload.userId, {
        title: 'Order Confirmed!',
        body: `Your order for $${payload.total} has been confirmed.`,
        data: { orderId: payload.orderId },
      });
      break;
  }
};
```

### Dead Letter Queue (DLQ)

If a message fails to process after N retries, it goes to a DLQ for investigation.

```
Main Queue  →  Consumer fails 3x  →  Dead Letter Queue
                                          ↓
                                   Alert fires → Engineer investigates
```

This prevents bad messages from blocking the entire queue forever.

---

## 4.7 GraphQL — The React Native-Friendly API Pattern

GraphQL lets the client specify exactly what data it needs in a single request. Very popular in React Native apps (Apollo Client, urql, TanStack Query + graphql-request).

```javascript
// Problem with REST — over-fetching and under-fetching
// Over-fetching: GET /users/1 returns 50 fields, you only needed 3
// Under-fetching: You need user + their posts + post likes → 3 separate requests

// GraphQL solution — one request, exactly the fields you need
const GET_USER_PROFILE = gql`
  query GetUserProfile($userId: ID!) {
    user(id: $userId) {
      id
      name
      avatarUrl
      posts(first: 5, orderBy: { field: CREATED_AT, direction: DESC }) {
        edges {
          node {
            id
            title
            likesCount
            createdAt
          }
        }
      }
      followersCount
    }
  }
`;

// Apollo Client in React Native
const UserProfile = ({ userId }) => {
  const { data, loading, error } = useQuery(GET_USER_PROFILE, {
    variables: { userId },
  });

  if (loading) return <Skeleton />;
  if (error) return <ErrorView message={error.message} />;

  return (
    <View>
      <Text>{data.user.name}</Text>
      <Text>{data.user.followersCount} followers</Text>
      {data.user.posts.edges.map(({ node }) => (
        <PostCard key={node.id} post={node} />
      ))}
    </View>
  );
};

// Mutation — create or modify data
const FOLLOW_USER = gql`
  mutation FollowUser($targetUserId: ID!) {
    followUser(targetUserId: $targetUserId) {
      success
      followersCount
    }
  }
`;

const FollowButton = ({ targetUserId }) => {
  const [followUser, { loading }] = useMutation(FOLLOW_USER, {
    variables: { targetUserId },
    // Optimistic update — update UI before server confirms
    optimisticResponse: {
      followUser: { success: true, followersCount: currentCount + 1 }
    },
  });

  return <Button title="Follow" onPress={followUser} loading={loading} />;
};
```

### REST vs GraphQL

| | REST | GraphQL |
|--|------|---------|
| Multiple resources | Multiple requests | One request |
| Data shape | Server decides | Client decides |
| Over-fetching | Common | Eliminated |
| Caching | HTTP cache (easy) | Query-level cache (more work) |
| Learning curve | Low | Medium |
| Tooling | Universal | Apollo, urql, codegen |

---

## 4.8 Rate Limiting

Prevents any single client from overwhelming your API.

```
Rule: Maximum 100 requests per minute per user.

Request 1-100 within a minute:  → 200 OK
Request 101 within a minute:    → 429 Too Many Requests
                                   Retry-After: 45 (seconds until limit resets)
```

```javascript
// React Native — handle rate limiting gracefully
const fetchWithRateLimitHandling = async (url, options) => {
  const response = await fetch(url, options);

  if (response.status === 429) {
    const retryAfter = parseInt(response.headers.get('Retry-After') ?? '60', 10);
    console.warn(`Rate limited. Retrying after ${retryAfter}s`);

    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
    return fetch(url, options); // Retry after the wait
  }

  return response;
};

// Always expose rate limit info to your UI
// So users understand why they're waiting, not confused by errors
```

---

## 4.9 WebSockets and Real-Time

For features where the server needs to push data to the client without the client polling.

```javascript
// Socket.io in React Native (most popular library for real-time)
import io from 'socket.io-client';

const socket = io('https://api.myapp.com', {
  auth: { token: await SecureStore.getItemAsync('access_token') },
  transports: ['websocket'], // Force WebSocket, skip long-polling fallback
});

// Connect / disconnect lifecycle
useEffect(() => {
  socket.on('connect', () => {
    console.log('Connected. Socket ID:', socket.id);
    socket.emit('join_room', { roomId: 'chat_123' });
  });

  socket.on('disconnect', (reason) => {
    if (reason === 'io server disconnect') {
      socket.connect(); // Server disconnected us — reconnect
    }
    // Otherwise socket.io auto-reconnects
  });

  socket.on('new_message', (message) => {
    setMessages(prev => [message, ...prev]);
  });

  socket.on('user_typing', ({ userId, isTyping }) => {
    updateTypingIndicator(userId, isTyping);
  });

  return () => {
    socket.off('new_message');
    socket.off('user_typing');
    socket.disconnect();
  };
}, []);

// Send events
const sendMessage = (text) => {
  socket.emit('send_message', {
    roomId: 'chat_123',
    text,
    clientTempId: Date.now().toString(), // Optimistic ID before server assigns one
  });
};
```

### Polling vs WebSockets vs Server-Sent Events

| Method | How it works | Best for |
|--------|-------------|----------|
| **Short polling** | App calls API every N seconds | Simple, infrequent updates |
| **Long polling** | Server holds connection open until data ready | Moderate frequency |
| **Server-Sent Events (SSE)** | Server pushes, one direction only | Live feeds, notifications |
| **WebSockets** | Full bidirectional persistent connection | Chat, collaboration, live updates |

---

## 4.10 API Security

### Password Storage

Passwords are **never** stored, and never encrypted — they are **hashed with a slow, salted algorithm**. Encryption is reversible; that's exactly what you don't want.

```javascript
import argon2 from 'argon2';   // Or bcrypt — both are correct choices

// Registration — hash before storing. The salt is generated and embedded automatically.
const register = async (email, password) => {
  const passwordHash = await argon2.hash(password);
  await db.query(
    'INSERT INTO users (email, password_hash) VALUES ($1, $2)',
    [email, passwordHash],
  );
};

// Login — verify against the stored hash. You never decrypt anything.
const login = async (email, password) => {
  const user = await db.oneOrNone('SELECT * FROM users WHERE email = $1', [email]);

  // Always run a verify even when the user doesn't exist, and always return the
  // same error — otherwise response timing and wording tell an attacker which
  // emails are registered (user enumeration).
  const valid = user
    ? await argon2.verify(user.password_hash, password)
    : await argon2.verify(DUMMY_HASH, password).catch(() => false);

  if (!user || !valid) throw new AuthError('Invalid email or password');

  return issueTokens(user);
};
```

| Use | Never use |
|-----|-----------|
| Argon2id (current recommendation), bcrypt, scrypt | MD5, SHA-1, SHA-256 — fast hashes, brute-forceable at billions/sec |
| A per-password salt (these libraries do it for you) | A single global salt, or no salt |
| Rate limiting + lockout on the login endpoint | Unlimited login attempts |

> **Why "slow" is the point.** A GPU computes billions of SHA-256 hashes per second.
> Argon2 and bcrypt are deliberately expensive in time *and memory*, so an attacker
> who steals your database still can't feasibly reverse the hashes. If your login
> feels instant to compute, it's insecure.

### Input Validation

Never trust data from the client. Validate everything on the server.

```javascript
// Server-side validation (Express + Zod)
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(8).max(128),
  name: z.string().min(1).max(100).trim(),
});

app.post('/users', async (req, res) => {
  const result = CreateUserSchema.safeParse(req.body);

  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        fields: result.error.flatten().fieldErrors,
      }
    });
  }

  const { email, password, name } = result.data; // Safe, validated data
  // proceed
});
```

### SQL Injection Prevention

Never concatenate user input into SQL queries. Always use parameterized queries.

```javascript
// ❌ NEVER do this — SQL injection vulnerability
const userId = req.params.id; // Could be "1; DROP TABLE users; --"
await db.query(`SELECT * FROM users WHERE id = ${userId}`); // DANGEROUS

// ✅ Always use parameterized queries
await db.query('SELECT * FROM users WHERE id = $1', [userId]); // Safe
```

### CORS — Cross-Origin Resource Sharing

Browsers block cross-origin requests by default. Your server must explicitly allow them.

```javascript
// Node.js + Express
import cors from 'cors';

app.use(cors({
  origin: ['https://myapp.com', 'https://staging.myapp.com'],
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Authorization', 'Content-Type'],
}));

// React Native doesn't have CORS restrictions (it's not a browser)
// But your web version does. Set CORS on the server.
```

---

## Summary

| Concept | Key Takeaway |
|---------|-------------|
| REST design | Noun-based URLs, HTTP method = action, consistent response shape |
| Versioning | `/v1/` in URL — never break existing clients |
| JWT | Signed, self-contained token. Access (short) + Refresh (long). |
| OAuth | Delegate auth to Google/Apple. Your server issues your own JWT after. |
| RBAC | JWT payload carries role. Middleware enforces permissions. |
| SQL | ACID, JOINs, transactions. Best for structured relational data. |
| NoSQL | Flexible schema. Each type optimized for a different access pattern. |
| Indexes | O(log n) lookups. Trade write speed for read speed. |
| N+1 queries | 1 query becomes 101. Fix with JOIN, batch loading, or DataLoader. |
| Connection pooling | Reuse connections. `max` is per instance — watch it when scaling out. |
| Password hashing | Argon2/bcrypt, salted and slow. Never MD5/SHA. Never reversible. |
| Transactions | Atomic. All or nothing. Prevents half-updated state. |
| Deadlocks | Circular lock waiting. Fix: consistent order, timeouts, optimistic locking. |
| Cache-aside | Check cache first. Miss → load DB → populate cache. |
| Write-through | Write to cache and DB simultaneously. Always fresh. |
| Message queues | Async, decoupled, fault-tolerant task processing. |
| DLQ | Failed messages land here for investigation. |
| GraphQL | Client-defined queries. One request. Eliminates over/under-fetching. |
| Rate limiting | 429 Too Many Requests. Handle with Retry-After. |
| WebSockets | Full duplex. Use for chat, real-time collaboration. |
| SQL injection | Always use parameterized queries. Never concatenate user input. |

---

*Previous: [Floor 3.5 — Mobile System Design](../Floor3.5-Mobile-System-Design/resource.md) · Next: [Floor 5 — Infrastructure and DevOps](../Floor5-Infrastructure/resource.md) · [Index](../README.md)*
