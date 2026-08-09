# Floor 3 — High-Level System Design

> **Prerequisites:** Floor 1 (Fundamentals) + Floor 2 (OOP/Patterns)
> **Focus:** Thinking in systems — entire services, not individual classes.

---

## 3.1 System Design vs System Architecture

These terms are related but distinct.

**System Architecture** is the blueprint — the big picture. It answers:
- What are the major components?
- How do they relate to each other?
- What are the boundaries between them?

**System Design** is the detailed decision-making. It answers:
- What exact protocols do components use to communicate?
- What happens when a component fails?
- How does each component scale?
- What are the latency, throughput, and storage requirements?

**Architecture is the noun. System design is the verb.**

```
Architecture: "We have a mobile client, an API gateway, a user service,
               an orders service, and a notification service."

System Design: "The mobile client sends JWT-authenticated REST requests
               to the API gateway. The gateway validates the token and
               routes to services. Services communicate async via SQS.
               The user service has a read replica for query performance.
               Failed messages go to a dead-letter queue with alerting."
```

As a React Native developer, you already think in architecture daily:
- Navigation stack architecture
- State management architecture (Redux, Zustand, Context)
- Feature folder vs screen folder vs domain-driven structure

Floor 3 extends this thinking to the full system — frontend + backend + infrastructure.

---

## 3.2 Monolith vs Microservices

### Monolith

The entire application is **one deployable unit** — one codebase, one process, one database.

```
┌─────────────────────────────────────────┐
│              MONOLITH                   │
│                                         │
│  ┌──────────┐  ┌──────────┐            │
│  │  Users   │  │  Orders  │            │
│  └──────────┘  └──────────┘            │
│  ┌──────────┐  ┌──────────┐            │
│  │Payments  │  │  Email   │            │
│  └──────────┘  └──────────┘            │
│                                         │
│           Single Database               │
└─────────────────────────────────────────┘
         ↕ Single deployment
```

**Advantages:**
- Simple to develop and run locally
- No network overhead between components (function calls, not HTTP)
- Easier to debug — one log stream, one process
- Easy transactions across multiple "services" (they share the DB)
- Fast to iterate at the start

**Disadvantages:**
- As it grows, a bug anywhere can crash everything
- Scaling requires scaling the entire app, even if only one part is under load
- Slow CI/CD builds — one change triggers a full rebuild
- Hard for multiple teams to work on without stepping on each other

### Microservices

The application is split into **small, independent services** that each do one thing.

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  User    │    │  Order   │    │ Payment  │
│ Service  │    │ Service  │    │ Service  │
│          │    │          │    │          │
│  DB      │    │  DB      │    │  DB      │
└────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │
     └───────────────┴───────────────┘
                     │
              Message Bus / API Gateway
                     │
              React Native App
```

**Advantages:**
- Each service deploys independently — change user service without touching orders
- Scale services independently — if orders are spiking, scale only the Orders service
- Teams own services autonomously
- One service crashing doesn't bring down the whole system
- Use different tech stacks per service if needed

**Disadvantages:**
- Much more complex to operate — orchestration, service discovery, monitoring
- Network calls between services add latency and failure points
- Distributed transactions are hard (no single DB to ACID-rollback across services)
- More services to monitor, debug, and maintain

### When to use which

| Stage | Recommended approach |
|-------|---------------------|
| Early startup / MVP | Monolith — ship fast, don't over-engineer |
| Growing team (5-20 devs) | Modular monolith — clean internal boundaries |
| Multiple independent teams | Microservices — team ownership per service |
| Performance-critical parts | Extract specific services (e.g., image processing) |

> Most successful companies (Netflix, Uber, Airbnb) started as monoliths and extracted services over time. Starting with microservices is usually premature optimization.

### Modular Monolith — The Middle Ground

A monolith with **strict internal module boundaries** — the best of both worlds during growth.

```javascript
// File structure of a well-bounded monolith
src/
  modules/
    users/
      UserRepository.js    // Only users/ touches this
      UserService.js
      userRoutes.js
    orders/
      OrderRepository.js   // Only orders/ touches this
      OrderService.js
      orderRoutes.js
    payments/
      PaymentService.js
      paymentRoutes.js
  shared/
    database.js            // Shared DB connection
    auth.js                // Shared auth middleware

// Rule: modules NEVER import from each other's internals
// orders/OrderService.js may call users/ only through a defined interface:
// ✅ import { getUserById } from '../users/UserService';
// ❌ import { UserRepository } from '../users/UserRepository';
```

---

## 3.3 How Services Communicate

### Synchronous — One Service Waits for Another

**REST over HTTP** — most common, simple, stateless. Your React Native app already does this.

```
React Native App  →  POST /orders  →  Order Service
                  ←  201 Created   ←  (waits for response)
```

**gRPC** — binary protocol, faster than JSON/REST, strongly typed. Used for internal service-to-service calls where performance matters.

```
Order Service  →  gRPC/Protobuf call  →  Inventory Service
              ←  response (binary)     ←  (faster than HTTP/JSON)
```

**GraphQL** — query language where the client specifies exactly what data it needs. Common in React Native apps (Apollo Client, urql).

```javascript
// REST — multiple round trips to get related data
const user = await fetch('/api/users/1');
const posts = await fetch('/api/users/1/posts');
const followers = await fetch('/api/users/1/followers');

// GraphQL — one request, exactly the data you need
const query = `
  query GetUserProfile($id: ID!) {
    user(id: $id) {
      id
      name
      email
      posts(limit: 10) {
        id
        title
        createdAt
      }
      followersCount
    }
  }
`;

const { data } = await apolloClient.query({ query, variables: { id: '1' } });
```

### Asynchronous — Fire and Forget via Message Queue

One service publishes a message. Another service picks it up when ready. Neither waits.

```
Order Service  →  "order.created" event  →  Message Queue (SQS/Kafka)
                                               ↓  (asynchronous — Order Service has moved on)
                                         Notification Service reads the event
                                               ↓
                                         Sends push notification
```

**Message Queue (SQS, RabbitMQ):**
- Guarantees delivery — message persists until consumed
- Point-to-point — one producer, one consumer
- Great for tasks: "process this image", "send this email"

**Event Streaming (Kafka):**
- Messages persist for a configurable time (days/weeks)
- Multiple consumers can independently read the same event stream
- Great for: audit logs, analytics, event sourcing
- "The Order service published an event; the Notification Service, Analytics Service, AND Loyalty Service all independently consumed it"

### Choosing Sync vs Async

| Use sync when | Use async when |
|--------------|---------------|
| The caller needs an immediate answer | The caller can continue without waiting |
| The operation is fast (< 100ms) | The operation is slow (email, image processing) |
| Strong consistency is required | Eventual consistency is acceptable |
| One service needs to confirm before proceeding | Services should be decoupled |

---

## 3.4 Scalability Patterns

### Vertical Scaling (Scale Up)

Add more resources to the **same server** — more CPU, more RAM, faster disk.

```
Before:  1 server × 4 CPU cores, 16GB RAM
After:   1 server × 32 CPU cores, 128GB RAM
```

**Pros:** Simple, no code changes, no distribution problems.
**Cons:** Hard ceiling (there's a largest machine). Single point of failure. Expensive.

### Horizontal Scaling (Scale Out)

Add **more servers** and distribute traffic between them.

```
                    ┌──────────────┐
                    │ Load Balancer │
                    └──────┬───────┘
              ┌─────────────┼─────────────┐
         ┌────┴────┐   ┌────┴────┐   ┌────┴────┐
         │ Server 1│   │ Server 2│   │ Server 3│
         └─────────┘   └─────────┘   └─────────┘
```

**Pros:** Theoretically unlimited scale. Survives individual server failure.
**Cons:** Requires **stateless** services. Sessions must be stored externally (Redis). More complex to operate.

**Stateless** means any server can handle any request — no user-specific data is stored in server memory. This is why JWT tokens (self-contained) are popular over server-side sessions.

### Load Balancer

Distributes incoming requests across multiple server instances.

**Algorithms:**
- **Round Robin** — requests rotate evenly across servers (simplest)
- **Least Connections** — sends to server with fewest active connections
- **IP Hash** — same IP always goes to the same server (sticky sessions)
- **Weighted** — sends more traffic to more powerful servers

```
User Request 1  →  Server 1
User Request 2  →  Server 2
User Request 3  →  Server 3
User Request 4  →  Server 1  (round robin repeats)
```

If Server 2 goes down, the load balancer stops sending traffic there. Users never notice. This is **high availability**.

### Caching

Store the result of expensive operations so you don't repeat them.

```
┌──────────────┐      Cache Miss         ┌───────────┐
│ React Native │ ──────────────────────► │   Cache   │
│     App      │ ◄──────────────────────  │  (Redis)  │
└──────────────┘  Cache Hit (fast)        └─────┬─────┘
                                                │ Cache Miss
                                          ┌─────▼─────┐
                                          │  Database  │
                                          └───────────┘
```

**Where to cache:**
- **CDN (Content Delivery Network)** — caches static files (images, fonts, JS bundles) on servers geographically close to users. Instead of requesting from a server in US-East, a user in Tokyo gets it from a CDN node in Tokyo. 10ms vs 200ms.
- **Redis (in-memory cache)** — microsecond reads. Cache API responses, user sessions, computed results. Data lives in RAM.
- **HTTP Cache headers** — server tells client/CDN how long to cache a response

```javascript
// Server sets cache headers on the API response
response.setHeader('Cache-Control', 'public, max-age=3600'); // Cache for 1 hour
response.setHeader('ETag', '"v1.2.3"'); // Fingerprint of the content

// React Native can respect cache headers via fetch cache option
const response = await fetch('/api/products', { cache: 'default' });
```

**Cache invalidation** — knowing when to expire stale data. This is one of the hardest problems in computer science because it requires the system to know when a cached value is no longer correct.

### Database Read Replicas

One **primary** database handles all writes. Multiple **replica** databases handle reads.

```
                    Write
App ────────────► Primary DB ──────► Replica 1 (read)
     Read ◄────── Replica 2 (read)         ↑ (async replication)
     Read ◄────── Replica 3 (read)
```

Most applications read far more than they write (100:1 ratio is common). Read replicas let you scale reads horizontally without touching write performance.

**Trade-off:** Replicas may lag slightly behind the primary (replication lag). A user who just posted something might not see it immediately if their read hits a lagging replica.

### Database Sharding

Split a large database across multiple machines. Each shard holds a **subset** of the data.

```
Users A-M  →  Shard 1 (users 1 - 50M)
Users N-Z  →  Shard 2 (users 50M - 100M)
```

**Shard key** — the field used to determine which shard holds a record (user ID, geographic region, etc.).

**Pros:** Horizontal scale for writes (each shard has its own primary).
**Cons:** Cross-shard queries are very complex. Hard to rebalance after the fact. Schema changes require coordinated migrations across all shards.

---

## 3.5 The CAP Theorem

In a distributed system, you can only **guarantee two of three** properties:

| Property | What it means |
|----------|--------------|
| **C — Consistency** | Every read returns the most recent write |
| **A — Availability** | Every request gets a response (even if not the latest data) |
| **P — Partition Tolerance** | System keeps working even if network links between nodes fail |

Network partitions **always happen** in real distributed systems (cables fail, switches reboot, data centers have issues). So **P is not optional** — you must tolerate partitions. The real choice is:

**When a partition occurs, do you sacrifice C or A?**

```
CP System (Consistency + Partition Tolerance):
  "If nodes can't agree on the latest data, return an error rather than stale data."
  Examples: HBase, Zookeeper, etcd
  Use when: financial transactions, inventory counts, anything where stale = wrong

AP System (Availability + Partition Tolerance):
  "Always return a response, even if the data might be slightly stale."
  Examples: Cassandra, DynamoDB, CouchDB
  Use when: social feeds, product catalogs, anything where slightly stale = acceptable
```

**Real example:** Your React Native app shows a user's follower count.
- CP: If the service has a partition, show an error ("couldn't load"). Correct but degraded UX.
- AP: Show the last known count (might be 5 seconds stale). Degraded accuracy but keeps working.

For follower counts, AP is the right choice. For bank balances, CP.

---

## 3.6 Consistency Models

Beyond CAP, it's useful to understand levels of consistency in distributed databases.

| Model | What it guarantees | Speed |
|-------|-------------------|-------|
| **Strong consistency** | Read always returns latest write. Like a single-node system. | Slowest |
| **Eventual consistency** | All nodes will agree eventually; reads may be stale for a short time. | Fastest |
| **Read-your-writes** | You always see your own writes immediately. Others may not yet. | Middle |
| **Causal consistency** | Operations causally related appear in order. Unrelated can be reordered. | Middle |

Most mobile apps are fine with eventual consistency for non-critical data (feeds, profiles, counts). Require strong consistency for critical data (payments, orders, medical records).

---

## 3.7 API Gateway

A single entry point that all clients (your React Native app) talk to. The gateway routes requests to the correct internal service.

```
React Native App
        │
        ▼
  ┌─────────────────────────────────────┐
  │           API Gateway               │
  │  - Authentication (validate JWT)    │
  │  - Rate limiting                    │
  │  - Request routing                  │
  │  - Response aggregation             │
  │  - Logging & monitoring             │
  │  - SSL termination                  │
  └──────────────┬──────────────────────┘
                 │
    ┌────────────┼────────────┐
    ▼            ▼            ▼
User Service  Order Service  Notification Service
```

Benefits:
- Your app talks to one URL, not 10 service URLs
- Auth logic lives in one place, not repeated across every service
- Can version APIs (`/v1/`, `/v2/`) without changing downstream services
- A/B testing, feature flags, canary deployments at the edge

---

## 3.8 Event-Driven Architecture

Services communicate exclusively through events. No service calls another directly.

```
User Service publishes: "user.registered" event
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
     Email Service      Analytics       Onboarding
   (sends welcome)    (tracks signup)   (creates tutorial)
```

Each consumer acts independently on the same event. The User Service doesn't know or care who's listening.

**Pros:**
- Extremely decoupled — services are unaware of each other
- Easy to add new consumers without changing producers
- Inherently asynchronous and scalable

**Cons:**
- Hard to trace what happened (event flows are less obvious than call stacks)
- Eventual consistency — the email might not send for a few seconds after registration
- Harder to debug — need good distributed tracing

---

## 3.9 Circuit Breaker Pattern

Prevents a failing service from cascading failures to other services.

```
Normal: App → Service A → Service B (working)

Service B fails:

Closed state:   App → Service A → Service B (failing, retrying)
                      After N failures:
Open state:     App → Service A →  ✗ (Circuit open, immediately returns error)
                      After timeout:
Half-open:      App → Service A → Service B (probe — did it recover?)
                      If success: back to Closed
                      If fail: back to Open
```

```javascript
class CircuitBreaker {
  #failures = 0;
  #lastFailureTime = null;
  #state = 'closed'; // closed | open | half-open
  #threshold = 5;
  #timeout = 60000; // 60 seconds

  async call(fn) {
    if (this.#state === 'open') {
      if (Date.now() - this.#lastFailureTime > this.#timeout) {
        this.#state = 'half-open';
      } else {
        throw new Error('Circuit is open — service unavailable');
      }
    }

    try {
      const result = await fn();
      this.#onSuccess();
      return result;
    } catch (error) {
      this.#onFailure();
      throw error;
    }
  }

  #onSuccess() {
    this.#failures = 0;
    this.#state = 'closed';
  }

  #onFailure() {
    this.#failures++;
    this.#lastFailureTime = Date.now();
    if (this.#failures >= this.#threshold) {
      this.#state = 'open';
    }
  }
}

// Usage in React Native — protect against flaky external services
const paymentCircuitBreaker = new CircuitBreaker();

const processPayment = async (order) => {
  return paymentCircuitBreaker.call(() =>
    fetch('/api/payments', { method: 'POST', body: JSON.stringify(order) })
  );
};
```

---

## 3.10 Service Discovery

In a dynamic environment where services start/stop and IP addresses change, services need to find each other.

**Client-side discovery:** The client queries a service registry (like Consul or Eureka) to find available instances.

**Server-side discovery:** A load balancer queries the registry and routes requests. The client only knows the load balancer's address.

In Kubernetes (Floor 5), service discovery is built-in — services find each other by **name** (`http://user-service/users`), not by IP.

---

## 3.11 Reliability Patterns You Will Be Asked About

Circuit breakers (3.9) are one of a family. These are the rest, and mobile clients make every one of them matter more, because phones retry on flaky networks constantly.

### Idempotency Keys — the one every mobile dev needs

Your app POSTs an order. The network dies before the response arrives. Did the order get created? The app doesn't know. If it retries, the user might be charged twice.

Idempotent *verbs* (GET, PUT, DELETE) don't help here — `POST /orders` is inherently non-idempotent. The fix is an **idempotency key**: the client generates a unique ID for the *attempt*, and the server guarantees that key can only ever produce one result.

```javascript
// Client — the key is generated ONCE per user intent, not per retry
const placeOrder = async (cart) => {
  const idempotencyKey = uuidv4();   // Generated when the user taps "Place Order"

  return fetchWithRetry('/v1/orders', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Idempotency-Key': idempotencyKey,   // SAME key on every retry
    },
    body: JSON.stringify(cart),
  });
};
```

```javascript
// Server — store the result against the key, replay it on repeat
const createOrder = async (req, res) => {
  const key = req.headers['idempotency-key'];
  if (!key) return res.status(400).json({ error: 'Idempotency-Key required' });

  const existing = await redis.get(`idem:${key}`);
  if (existing) {
    // This exact attempt already succeeded — replay the original response.
    // No second order. No second charge.
    return res.status(200).json(JSON.parse(existing));
  }

  const order = await orderService.create(req.user.id, req.body);
  await redis.setEx(`idem:${key}`, 24 * 60 * 60, JSON.stringify(order));

  res.status(201).json(order);
};
```

> **The critical detail interviewers listen for:** the key must be generated when the
> *user forms the intent*, not inside the retry loop. Generate it per-retry and you
> have implemented nothing at all. This is how Stripe's API works, and it's why
> `Idempotency-Key` is a header you'll meet in real payment integrations.

### Timeouts

A request with no timeout is a resource leak with extra steps. If a downstream service hangs, every caller's connection pool fills with requests waiting forever, and the failure spreads upward. **Every network call needs a timeout**, and the timeout budget must shrink as you go deeper:

```
Mobile client:  10s
  → API Gateway:  8s
    → Order Service:  5s
      → Inventory Service:  2s
```

If the inner call had the longer timeout, the outer one would give up first and the work would be wasted anyway.

### Retries — and when NOT to

| Retry | Don't retry |
|-------|------------|
| 500, 502, 503, 504 (server-side, likely transient) | 400, 401, 403, 404, 422 (your request is wrong — it will fail identically) |
| Network timeouts, connection resets | 409 Conflict (needs a real decision) |
| 429, but only after `Retry-After` | Anything non-idempotent without an idempotency key |

Always with exponential backoff **and jitter** (Floor 1). Always with a retry cap.

### Graceful Degradation and Load Shedding

When the system is overloaded, serving everyone slowly is worse than serving most people fast.

- **Graceful degradation** — the recommendations service is down, so the feed renders without the "For You" row instead of showing an error page. Core path survives; the garnish disappears.
- **Load shedding** — the server is at capacity, so it rejects low-priority requests (analytics, prefetch) immediately with 503 to protect high-priority ones (checkout).

For a mobile app this is a design decision you own: decide in advance which parts of a screen are essential and which can render empty. That decision is exactly what "designing for failure" means.

### Cache Stampede

A popular cache key expires. Ten thousand requests miss simultaneously and all hit the database at once — which then falls over.

Fixes: a short lock so only one request rebuilds the value while others serve stale; or randomised TTLs so keys don't all expire together; or refreshing the cache in the background before expiry.

### Consistent Hashing

If you have 4 cache servers and route with `hash(key) % 4`, adding a fifth server changes the destination of **almost every key** — the entire cache is instantly invalid, and the stampede above happens across the whole system.

**Consistent hashing** maps servers and keys onto the same ring, so adding or removing a node only remaps the keys immediately adjacent to it — roughly `1/n` of them instead of all of them. This is what Redis Cluster, Cassandra, and DynamoDB use to partition data.

---

## Summary

| Concept | One-line summary |
|---------|----------------|
| Monolith | Everything in one deployable unit. Simple. Gets hard to scale. |
| Microservices | Small, independent, separately deployable services. Complex to operate. |
| Modular monolith | Clean internal boundaries in one codebase. Best of both at mid-scale. |
| REST | Synchronous, stateless HTTP communication. Standard for client-server. |
| GraphQL | Client-defined queries. Solves over/under-fetching. Popular in RN. |
| Message Queue | Async communication. Decouples services. Guarantees delivery. |
| Event Streaming | Events persisted on a log. Many consumers. Kafka. |
| Vertical scaling | Bigger server. Simple. Hard ceiling. |
| Horizontal scaling | More servers + load balancer. Stateless required. |
| Caching | Store expensive results. Cache-aside, Redis, CDN. |
| Read replicas | Scale reads independently from writes. |
| Sharding | Split data horizontally across machines. |
| CAP theorem | Consistency vs Availability when network partitions occur. |
| Eventual consistency | All nodes agree eventually. Fine for feeds, counts. |
| API Gateway | Single entry point. Auth, routing, rate limiting. |
| Event-driven | Services react to events. Decoupled but hard to trace. |
| Circuit breaker | Stop cascading failures. Open/closed/half-open states. |
| Service discovery | Services find each other dynamically. |
| Idempotency key | Client-generated key per intent. Makes POST safe to retry. |
| Timeouts | Every call needs one. Budgets shrink as you go deeper. |
| Graceful degradation | Drop non-essential features instead of failing the whole screen. |
| Cache stampede | Simultaneous misses crush the DB. Fix with locks or jittered TTLs. |
| Consistent hashing | Adding a node remaps ~1/n keys, not all of them. |

---

*Previous: [Floor 2 — OOP and Design Patterns](../Floor2-OOP-SOLID/resource.md) · Next: [Floor 3.5 — Mobile System Design](../Floor3.5-Mobile-System-Design/resource.md) · [Index](../README.md)*
