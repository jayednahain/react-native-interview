# System Design — Floor by Floor

> A complete learning guide for app developers. Start at Floor 1. Do not skip floors.

---

## How to use this guide

Think of software engineering knowledge as a building. Every floor depends on the one below it. You cannot understand Floor 3 without Floor 2. You cannot skip to the roof and expect it to stand.

As a mobile app developer, you already live on **Floor 1** with real production experience. That is a massive advantage over someone learning system design from scratch. This guide shows you what you already know, names it properly, and shows you exactly what to learn next.

---

## Floor 1 — Fundamentals (Start here)

### What this floor covers

This is the foundation everything else is built on. If this floor is shaky, every floor above it will crack.

### 1.1 How data moves — the request/response cycle

Every time your app calls an API, this is what happens:

1. Your app builds an HTTP request (method, URL, headers, body)
2. That request travels over TCP/IP to a server
3. The server processes it and sends back an HTTP response (status code, headers, body)
4. Your app reads the response and updates the UI

As a mobile developer, you already do this every day. The vocabulary to know:

- **HTTP methods** — GET (read), POST (create), PUT/PATCH (update), DELETE (remove)
- **Status codes** — 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Server Error
- **Headers** — metadata on every request/response (Content-Type, Authorization, Cache-Control)
- **JSON** — the data format most APIs use for the body

### 1.2 Networking basics

- **TCP/IP** — the protocol that makes data reliably travel across the internet, in order, without loss
- **DNS** — translates `api.myapp.com` into an IP address (`192.168.1.1`)
- **TLS/HTTPS** — encrypts the connection so nobody can read your data in transit
- **Latency vs bandwidth** — latency is how long one request takes (milliseconds); bandwidth is how much data flows per second

### 1.3 Data structures you must know

These appear everywhere in system design:

| Structure | What it is | Where you use it |
|-----------|-----------|-----------------|
| Array / List | Ordered collection | Lists in your UI, API response arrays |
| Hash map / Dictionary | Key → value lookup | Caching, indexing, grouping data |
| Queue | First in, first out | Task queues, notifications |
| Stack | Last in, first out | Navigation history, undo |
| Tree | Hierarchical nodes | File systems, UI view hierarchies |
| Graph | Nodes connected by edges | Social networks, routing |

### 1.4 What you already know as a mobile developer

You handle things backend developers rarely think about:

- **Local storage** — SQLite, SharedPreferences, UserDefaults, Room, CoreData
- **Network connectivity** — detecting offline state, queuing requests, retry logic
- **SDK and NDK** — calling platform APIs, writing native code via JNI or similar
- **Animation frame rate** — keeping UI at 60fps or 120fps, avoiding jank on the main thread
- **Device diversity** — different screen sizes, OS versions, hardware capabilities
- **App lifecycle** — foreground, background, killed, and how state survives each

This is genuine system design. You design for constraints that backend developers outsource to infrastructure.

---

## Floor 2 — OOP and Design Patterns

### What this floor covers

Object-Oriented Programming (OOP) is the way most production code is structured. Design patterns are named, reusable solutions to problems that appear again and again in OOP code.

### 2.1 The four pillars of OOP

#### Abstraction

Hiding complexity behind a simple interface. The caller doesn't need to know how it works — only what it does.

```kotlin
// You call this:
val user = userRepository.getUser(id)

// You don't see this:
// → opens DB connection
// → runs SQL query
// → maps result to User object
// → closes connection
// All hidden. That's abstraction.
```

In mobile development, you use abstraction every time you call a platform API. `camera.takePhoto()` hides thousands of lines of camera driver code.

#### Encapsulation

Keeping the internal state of an object private, and only exposing what is needed.

```kotlin
class UserSession {
    private var token: String = ""   // private — nobody else can touch this directly
    private var expiresAt: Long = 0

    fun isValid(): Boolean {         // public — the only way to check
        return token.isNotEmpty() && System.currentTimeMillis() < expiresAt
    }

    fun refresh(newToken: String, expiresAt: Long) {
        this.token = newToken
        this.expiresAt = expiresAt
    }
}
```

#### Inheritance

A class inherits the properties and methods of another class, and can add or override behavior.

```kotlin
open class BaseApiClient {
    fun get(url: String): Response { /* ... */ }
    fun post(url: String, body: Any): Response { /* ... */ }
}

class UserApiClient : BaseApiClient() {
    fun getUser(id: String) = get("/users/$id")     // inherited method, new purpose
    fun createUser(data: UserData) = post("/users", data)
}
```

Inheritance is powerful but easy to overuse. Prefer it only when there is a genuine "is-a" relationship.

#### Polymorphism

The same interface, different behavior depending on which class is behind it.

```kotlin
interface PaymentProvider {
    fun charge(amount: Double): Result
}

class StripeProvider : PaymentProvider {
    override fun charge(amount: Double) = callStripeAPI(amount)
}

class PayPalProvider : PaymentProvider {
    override fun charge(amount: Double) = callPayPalAPI(amount)
}

// Your code doesn't care which provider — it just calls charge()
fun processOrder(provider: PaymentProvider, total: Double) {
    provider.charge(total)
}
```

### 2.2 Abstract classes vs interfaces vs concrete classes

| Type | What it is | Can be instantiated? |
|------|-----------|---------------------|
| **Concrete class** | A fully implemented class | Yes — `val obj = MyClass()` |
| **Abstract class** | A partial class with some unimplemented methods | No — must be subclassed |
| **Interface** | A contract — only method signatures, no implementation | No — must be implemented |

Use an interface when you want to define *what* something can do without caring *how*.
Use an abstract class when you want to share some implementation but force subclasses to fill in the rest.
Use a concrete class when the implementation is complete and ready to use.

### 2.3 SOLID principles

SOLID is five rules for writing OOP code that stays maintainable as it grows. Each letter is one rule.

#### S — Single Responsibility Principle

A class should do one thing, and have one reason to change.

```kotlin
// WRONG — this class does too many things
class UserManager {
    fun getUser(id: String) { }
    fun saveUserToDatabase(user: User) { }
    fun sendWelcomeEmail(user: User) { }
    fun formatUserForDisplay(user: User) { }
}

// RIGHT — split into focused classes
class UserRepository { fun getUser(id: String) { } }
class UserEmailService { fun sendWelcomeEmail(user: User) { } }
class UserViewModel { fun formatForDisplay(user: User) { } }
```

#### O — Open/Closed Principle

A class should be open for extension but closed for modification. Add new behavior by adding new code, not by editing existing code.

```kotlin
// WRONG — every new payment type requires editing this class
class PaymentProcessor {
    fun process(type: String, amount: Double) {
        if (type == "stripe") { /* stripe logic */ }
        else if (type == "paypal") { /* paypal logic */ }
        // Adding Apple Pay = editing this class = risk of breaking existing logic
    }
}

// RIGHT — add a new provider without touching existing code
interface PaymentProvider { fun charge(amount: Double) }
class StripeProvider : PaymentProvider { override fun charge(amount: Double) { } }
class ApplePayProvider : PaymentProvider { override fun charge(amount: Double) { } }
```

#### L — Liskov Substitution Principle

A subclass should be usable wherever its parent class is expected, without breaking anything.

If you have a function that accepts `Animal`, and you pass it a `Dog` (which extends `Animal`), nothing should break. The subclass must honor the contract of the parent.

#### I — Interface Segregation Principle

Don't force a class to implement methods it doesn't need. Split large interfaces into smaller, focused ones.

```kotlin
// WRONG — forces all implementors to handle everything
interface Worker {
    fun work()
    fun eat()
    fun sleep()
}

// RIGHT — separate concerns
interface Workable { fun work() }
interface Eatable  { fun eat() }
// A Robot can implement Workable without being forced to implement eat()
```

#### D — Dependency Inversion Principle

Depend on abstractions, not concrete implementations. High-level code should not depend on low-level code directly.

```kotlin
// WRONG — UserService is tightly coupled to MySQLDatabase
class UserService {
    private val db = MySQLDatabase()  // can never swap this out
}

// RIGHT — UserService depends on an interface
class UserService(private val db: Database) {  // inject any Database implementation
    fun getUser(id: String) = db.find(id)
}
```

This is why dependency injection (Hilt, Dagger, Koin on Android) exists — it's SOLID's D in practice.

### 2.4 Common design patterns

These are named recipes for common OOP problems.

| Pattern | Category | What it solves |
|---------|----------|---------------|
| **Repository** | Structural | Abstracts data access — your ViewModel doesn't know if data came from network or cache |
| **Observer / LiveData** | Behavioral | One thing changes, others are notified automatically |
| **Factory** | Creational | Creates objects without exposing creation logic |
| **Singleton** | Creational | Ensures only one instance of a class exists (e.g. a database connection) |
| **Builder** | Creational | Constructs complex objects step by step |
| **Adapter** | Structural | Makes incompatible interfaces work together |
| **Strategy** | Behavioral | Swaps algorithms at runtime without changing the calling code |

---

## Floor 3 — High-Level System Design

### What this floor covers

This is where interviews live. You are no longer thinking about classes and methods — you are thinking about entire systems: how services talk to each other, where data lives, what happens when things fail, and how the system handles millions of users.

### 3.1 System design vs system architecture

**System architecture** is the big-picture plan — the blueprint. It answers: what are the major pieces, and how do they relate?

**System design** is the detailed decision-making — it answers: how exactly does each piece work, what protocols does it use, what happens when it fails, how does it scale?

Architecture is the noun. System design is the verb.

### 3.2 Monolith vs microservices

**Monolith** — the entire application is one deployable unit.

- Simple to develop and deploy at first
- One codebase, one process, one database
- Becomes hard to scale and maintain as it grows
- A bug in one part can crash the whole system

**Microservices** — the application is split into many small, independent services.

- Each service does one thing (User Service, Order Service, Payment Service)
- Each service has its own database
- Services communicate over HTTP (REST) or message queues
- Can scale each service independently
- More complex to operate — you need orchestration, service discovery, monitoring

Most companies start with a monolith and break it into microservices as they grow.

### 3.3 How services communicate

**Synchronous (direct)** — one service calls another and waits for the response.
- REST over HTTP — most common, simple, stateless
- gRPC — faster, binary protocol, used for internal service-to-service calls

**Asynchronous (via queue)** — one service puts a message on a queue, another service picks it up later.
- Message queue (RabbitMQ, AWS SQS) — guarantees delivery, decouples services
- Event streaming (Kafka) — services publish events, others subscribe and react

### 3.4 Scalability patterns

**Vertical scaling** — make the server bigger (more CPU, more RAM). Simple but has a hard ceiling.

**Horizontal scaling** — add more servers and distribute traffic between them. Requires a load balancer.

**Load balancer** — sits in front of your servers and distributes incoming requests. If one server goes down, the others keep serving traffic.

**Caching** — store the result of expensive operations so you don't repeat them.
- In-memory cache (Redis, Memcached) — microsecond reads, lives on a dedicated server
- CDN — caches static files (images, JS, CSS) geographically close to users
- Application cache — store results in your app's memory for the duration of a request

**Database sharding** — split a large database across multiple machines by a key (e.g. user ID). Each shard handles a subset of the data.

**Read replicas** — one primary database handles writes; multiple replica databases handle reads. Most apps read far more than they write.

### 3.5 The CAP theorem

In a distributed system, you can only guarantee two of these three things at once:

- **Consistency** — every read gets the most recent write
- **Availability** — every request gets a response (even if it's not the latest data)
- **Partition tolerance** — the system keeps working even if network connections between nodes fail

Network partitions always happen in real distributed systems. So the real choice is: when a partition occurs, do you prioritize consistency or availability?

---

## Floor 4 — Backend Architecture

### What this floor covers

How the server side of your app is actually built. This is the code that runs when your mobile app hits an API endpoint.

### 4.1 REST API design

A well-designed REST API follows predictable conventions:

```
GET    /users          → list all users
GET    /users/{id}     → get one user
POST   /users          → create a user
PUT    /users/{id}     → replace a user
PATCH  /users/{id}     → partially update a user
DELETE /users/{id}     → delete a user
```

Nested resources:
```
GET  /users/{id}/orders         → all orders for a user
GET  /users/{id}/orders/{oid}   → one specific order
```

### 4.2 Authentication and authorization

**Authentication** — who are you? (identity)
**Authorization** — what are you allowed to do? (permissions)

Common authentication patterns:

- **API keys** — a long secret string sent in a header. Simple, used for server-to-server calls.
- **JWT (JSON Web Token)** — a signed token that contains claims (user ID, roles, expiry). Your mobile app stores this after login and sends it on every request.
- **OAuth 2.0** — the protocol behind "Sign in with Google/Apple". Delegates authentication to a trusted provider.
- **Session cookies** — the server stores a session; the client stores a session ID cookie. More common in web apps than mobile.

The typical mobile login flow:
1. User enters credentials → app sends POST `/auth/login`
2. Server validates and returns a JWT access token + refresh token
3. App stores tokens securely (Keychain on iOS, EncryptedSharedPreferences on Android)
4. Every API call includes `Authorization: Bearer <access_token>`
5. When the access token expires, app uses the refresh token to get a new one silently

### 4.3 Databases

**Relational databases (SQL)** — data stored in tables with rows and columns. Relationships between tables via foreign keys.

- PostgreSQL, MySQL, SQLite
- Strong consistency, ACID transactions
- Best for structured data with clear relationships
- Scales vertically well; horizontal scaling is complex

**Non-relational databases (NoSQL)** — many types, each optimized for a different access pattern:

| Type | Examples | Best for |
|------|---------|----------|
| Document | MongoDB, Firestore | JSON-like records, flexible schema |
| Key-value | Redis, DynamoDB | Fast lookups by a single key |
| Column-family | Cassandra | Write-heavy time-series data |
| Graph | Neo4j | Highly connected data (social graphs) |

### 4.4 Transactions and the deadlock problem

A **transaction** groups multiple database operations into one atomic unit — either all succeed or all fail. This prevents your data from ending up in a half-updated state.

```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- If anything fails between BEGIN and COMMIT, both updates are rolled back
```

A **deadlock** happens when two transactions are each waiting for the other to release a lock, so neither can proceed.

Example:
- Transaction A locks User 1, then tries to lock User 2
- Transaction B locks User 2, then tries to lock User 1
- Both wait forever — deadlock

Solutions: consistent lock ordering, timeouts, retry logic, or optimistic locking.

### 4.5 Caching strategies

**Cache-aside (lazy loading)** — app checks cache first; if miss, loads from DB and populates cache.

**Write-through** — every write goes to cache and DB simultaneously. Cache is always fresh.

**Write-behind** — write to cache immediately, sync to DB asynchronously. Fast but risks data loss on crash.

**Cache invalidation** — deciding when to evict stale data. This is famously one of the hardest problems in computer science.

### 4.6 Message queues

When Service A needs Service B to do something, but A shouldn't wait for B to finish:

1. A puts a message on a queue ("send welcome email to user 123")
2. A continues doing other work immediately
3. B picks up the message when it's ready and processes it
4. If B fails, the message stays on the queue and can be retried

This decouples services, improves resilience, and smooths out traffic spikes.

---

## Floor 5 — Infrastructure and DevOps

### What this floor covers

How code gets from a developer's machine to production, and how it keeps running reliably at scale.

### 5.1 Containers — Docker

A container packages your application and all its dependencies into a single, portable unit that runs the same everywhere.

```dockerfile
FROM node:20
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

`docker build` creates an image. `docker run` starts a container from that image. The container runs in isolation — its own filesystem, network, and process space.

Why this matters for mobile developers: the API you call is likely running in a Docker container. When you test against a local backend, you are often running Docker.

### 5.2 Orchestration — Kubernetes

When you have many containers to manage across many machines, Kubernetes (K8s) automates:

- **Scheduling** — decides which machine to run each container on
- **Scaling** — automatically adds or removes containers based on traffic
- **Self-healing** — restarts crashed containers automatically
- **Rolling updates** — deploys new versions without downtime
- **Service discovery** — containers find each other by name, not IP address

Key concepts:

- **Pod** — the smallest unit in Kubernetes; usually one container
- **Deployment** — declares how many replicas of a pod to run
- **Service** — a stable network endpoint that routes to healthy pods
- **Ingress** — routes external HTTP traffic to the right service

### 5.3 CI/CD — Continuous Integration and Continuous Delivery

**Continuous Integration (CI)** — every code change triggers an automatic build and test run. Catches bugs before they reach production.

**Continuous Delivery (CD)** — after CI passes, the code is automatically deployed to staging (and optionally production).

A typical pipeline:
```
Developer pushes code
  → CI runs tests
  → CI builds Docker image
  → Image is pushed to registry
  → CD deploys image to staging
  → Manual approval (or automatic) → CD deploys to production
```

Common CI/CD tools: GitHub Actions, GitLab CI, CircleCI, Jenkins, Fastlane (for mobile).

### 5.4 Horizontal vs vertical scaling — in depth

**Vertical scaling (scale up):**
- Replace a 4-core server with a 32-core server
- Simple — no code changes needed
- Hard ceiling — there is a biggest machine, and it's expensive
- Single point of failure — if that machine goes down, everything goes down

**Horizontal scaling (scale out):**
- Add more servers behind a load balancer
- Theoretically unlimited — keep adding machines
- Requires stateless services — each request can go to any server
- More complex to operate

For a service to scale horizontally, it must be **stateless** — it cannot store session data in memory. Sessions must live in a shared store (Redis, database) that all instances can access.

### 5.5 Cloud providers — AWS, GCP, Azure

Cloud providers offer:
- **Compute** — virtual machines (EC2), containers (ECS, EKS), serverless (Lambda)
- **Storage** — object storage (S3), block storage (EBS), file storage (EFS)
- **Database** — managed RDS, DynamoDB, Firestore, BigQuery
- **Networking** — VPC, CDN (CloudFront), DNS (Route 53), load balancers
- **Monitoring** — CloudWatch, Stackdriver, logs, alerts, dashboards

The key insight: you are renting infrastructure instead of buying hardware. You pay for what you use, and the provider handles physical machine maintenance.

---

## How everything connects — end to end

When a user taps "Place Order" in your mobile app, here is what happens across all five floors:

```
[Your mobile app — Floor 1]
  ↓ HTTP POST /orders (JWT in header)
[Load balancer — Floor 5]
  ↓ routes to an available server
[API gateway / auth middleware — Floor 4]
  ↓ validates JWT, identifies user
[Order Service — Floor 3/4]
  ↓ validates request, creates order
[Database transaction — Floor 4]
  ↓ writes order to PostgreSQL
[Message queue — Floor 4]
  ↓ publishes "order created" event
[Notification Service — Floor 3/4]
  ↓ consumes event, sends push notification
[Your mobile app — Floor 1]
  ← receives push notification
```

Every floor played a role. That is system design.

---

## Your learning path — step by step

| Step | Topic | Why first |
|------|-------|-----------|
| 1 | HTTP, TCP/IP, status codes | You already do this — just learn the vocabulary |
| 2 | OOP basics — classes, objects, interfaces | Foundation for everything above |
| 3 | SOLID principles | Rules that make OOP code maintainable |
| 4 | Design patterns — Repository, Observer, Factory | Named recipes you are already using |
| 5 | REST API design | Bridges Floor 1 and Floor 4 |
| 6 | Databases — SQL vs NoSQL | How data is stored and why it matters |
| 7 | Authentication — JWT, OAuth | How identity works end to end |
| 8 | Scalability — caching, load balancing, queues | How systems survive growth |
| 9 | Microservices — when and why | Floor 3 system thinking |
| 10 | Docker and Kubernetes basics | Floor 5 — how code ships |

---

*Generated for a mobile app developer learning system design layer by layer.*
