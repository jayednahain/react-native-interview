# Floor 0 — The Design Process

> **Prerequisites:** None. Read this **first**, and re-read it after every other floor.
> **Why it exists:** Floors 1–5 teach you *components*. This floor teaches you the
> *procedure* for assembling them. Knowing what a load balancer is does not tell you
> when to reach for one — that's a different skill, and it's the one being tested.

---

## 0.1 What "system design" actually means

Here is the honest answer to the question you started with.

**System design is the skill of turning a vague requirement into a concrete technical plan, and being able to defend every decision in that plan.**

That's it. It is not a body of facts. It is a *procedure* applied to facts.

Two people can know identical facts and perform completely differently:

| | Person A | Person B |
|--|---------|---------|
| Asked "design a chat app" | Immediately draws Kafka, Redis, microservices | Asks "how many users? group chat or 1-1? does history need to sync across devices?" |
| Knows what Kafka is | Yes | Yes |
| Passes the interview | No | Yes |

Person A knows components. Person B has a process. **This floor is that process.**

### The one reframe that will save you

You keep asking "what type of system design should I learn?" — as if front-end, mobile, and backend system design were three different subjects.

They are the same subject applied to different constraints:

| | You are trading off... | Against... |
|--|----------------------|-----------|
| **Mobile** | Battery, data, storage, cold start, frame budget | Freshness, offline capability, feature richness |
| **Backend** | CPU, memory, database connections, cost | Latency, consistency, availability |
| **Infra** | Money, operational complexity | Reliability, scale, deploy speed |

Same verb — *trade off a scarce resource against a desirable property*. Different nouns. Once you see that, "which system design do I learn?" dissolves. You learn the verb, then you learn each floor's nouns.

And you already do this. When you chose `FlatList` over `ScrollView` for a 10,000-item list, you traded implementation complexity for memory and frame rate. That is a system design decision. Nobody told you it counted.

---

## 0.2 System design vs system architecture (the short version)

You asked about this specifically, so here it is up front, in the way that actually matters:

- **Architecture** is the *output* — the set of components and their relationships. A noun. It's what you'd draw on a whiteboard.
- **Design** is the *activity and the reasoning* — why those components, what happens when one dies, what you gave up to get there. A verb.

Two teams can arrive at the same architecture with completely different design quality. One picked microservices after measuring; one picked microservices because it's what they'd read about. Same diagram, different engineering.

**In an interview, the diagram gets you maybe 30% of the score. The reasoning is the rest.** This is why candidates who memorise architectures fail: they can reproduce the artifact but not the thinking that produced it.

---

## 0.3 The process — six steps, in order

Do not skip steps. Do not reorder them. Each one produces the input for the next.

```
1. REQUIREMENTS   →  What are we building, and what are we NOT building?
        ↓             (output: a scoped feature list + non-functional targets)
2. SCALE           →  How big is this? Rough numbers.
        ↓             (output: QPS, storage, bandwidth — order of magnitude)
3. API             →  What are the exact operations?
        ↓             (output: endpoint signatures)
4. DATA MODEL      →  What are we storing, and in what shape?
        ↓             (output: entities, relationships, DB choice)
5. HIGH-LEVEL      →  Draw the boxes and arrows.
        ↓             (output: the architecture diagram)
6. DEEP DIVE       →  Find the weakest point and fix it. Discuss trade-offs.
                      (output: bottleneck analysis + failure handling)
```

**Why this order?** You cannot choose a database (4) before you know the read/write ratio (2). You cannot know the read/write ratio before you know what the features are (1). Every step is genuinely blocked by the one before it. That's the "without step one I can't do step two" structure you asked for — and it's real, not arbitrary.

---

### Step 1 — Requirements (5 minutes, and never skip it)

**Never start drawing.** The most common failure in a system design interview is designing the wrong system, confidently and in detail.

Ask until you can write down three lists:

**Functional requirements — what it does:**
```
"Design Instagram" is not a task. This is:
  ✓ Users can post a photo with a caption
  ✓ Users can follow other users
  ✓ Users see a feed of posts from people they follow
  ✗ Stories, Reels, DMs, ads, search — OUT OF SCOPE (state this out loud)
```

Cutting scope explicitly is not dodging the question. It's the single clearest signal that you've done this before — because real projects are defined by what you refuse to build.

**Non-functional requirements — what qualities it must have:**
```
  - How many users? Daily active?
  - Read-heavy or write-heavy? (Feeds: ~100:1 reads. Analytics ingest: write-heavy.)
  - How fresh must data be? (Bank balance: instant. Follower count: 30s is fine.)
  - What's the availability target? 99.9% (8.8h down/year) or 99.99% (52min)?
  - Latency target? "Feed loads in under 200ms at p95."
  - Does it work offline? ← Ask this. You're a mobile dev. It's your edge.
```

**Constraints:**
```
  - Team size, deadline, existing stack, budget, compliance (GDPR, PCI, HIPAA)
```

> **Say this out loud in an interview:** *"Before I design anything, can I confirm the
> scope? I'm assuming X and Y are in, and Z is out. Is that right?"* You have just
> demonstrated more seniority than any diagram will.

---

### Step 2 — Estimate the scale (5 minutes)

You are not computing an exact answer. You are finding the **order of magnitude**, because that's what changes the design. The difference between 100 requests/sec and 100,000 requests/sec is a different architecture. The difference between 100 and 150 is nothing.

**The only formula you need:**

```
Daily Active Users × actions per user per day
─────────────────────────────────────────────  =  average requests per second
                  86,400
```

`86,400` is seconds in a day. Memorise it, or round to 100,000 and do it in your head.

**Worked example — a food delivery app:**

```
1,000,000 DAU
Each user opens the app ~5 times/day, ~20 API calls per session = 100 calls/day

  Average QPS = 1,000,000 × 100 / 86,400 ≈ 1,160 requests/sec

  Peak QPS ≈ 3× average ≈ 3,500 requests/sec
  (Peak matters more than average — you must survive dinner time, not the mean.)

Orders: 10% of DAU order once = 100,000 orders/day ≈ 1.2 writes/sec average
  → Reads outnumber writes ~1000:1. This single number tells you:
      - Read replicas, yes
      - Caching, absolutely
      - Sharding writes, not yet
```

**Storage:**
```
1 order ≈ 1 KB of structured data
100,000 orders/day × 1 KB × 365 = ~36 GB/year

That fits comfortably on one PostgreSQL instance. You do NOT need Cassandra.
Saying that out loud — "this doesn't need sharding" — scores points.
```

**Latency numbers worth knowing** (approximate, and that's fine — you need ratios, not precision):

| Operation | Time | In perspective |
|-----------|------|---------------|
| L1/memory reference | ~1 ns | — |
| Read 1 MB from RAM | ~10 µs | baseline |
| SSD random read | ~100 µs | 10× slower than RAM |
| Redis GET (same datacenter) | ~0.5 ms | |
| Database query (indexed, local) | ~1–10 ms | |
| Network round trip, same datacenter | ~0.5 ms | |
| Network round trip, cross-continent | ~100–150 ms | **physics — you cannot optimise this away** |
| Mobile 4G round trip | ~50–100 ms | |
| Mobile 3G round trip | ~200–500 ms | |
| One dropped frame at 60fps | 16.7 ms | your entire per-frame budget |

> **The mobile-specific insight:** at 100ms per round trip on cellular, **three
> sequential API calls cost you 300ms before any rendering happens**. This is why
> request waterfalls are the number one cause of a slow-feeling app, and why
> `Promise.all` and GraphQL/BFF endpoints exist. You know this from experience — now
> you have the numbers to justify it in a design discussion.

---

### Step 3 — Define the API (5 minutes)

This is where a mobile developer has an unfair advantage: **you have consumed hundreds of APIs and you know exactly which ones were painful.** Use that.

Write the operations before the architecture. It forces the data model to be concrete.

```
POST   /v1/orders
       Headers: Authorization: Bearer <jwt>, Idempotency-Key: <uuid>
       Body:    { restaurantId, items: [{ id, qty }], addressId }
       Returns: 201 { orderId, status, etaMinutes }

GET    /v1/orders?status=active&limit=20&cursor=<opaque>
       Returns: 200 { data: [...], nextCursor }

GET    /v1/orders/:id/tracking
       Returns: 200 { status, courierLocation, etaMinutes }
```

Three decisions to justify here, because they get asked every time:

1. **Cursor pagination, not offset.** `?page=2` on a live feed shows duplicates and skips items when new rows arrive between requests. Cursors are stable. Mobile feeds are always live. *(This is the kind of detail that reads as real experience.)*
2. **Idempotency key on writes** (Floor 3.11) — because phones retry on flaky networks and nobody should be charged twice.
3. **Polling vs push for tracking** — 5-second polling is simple and drains battery; WebSockets are efficient but need reconnection handling. Name the trade-off; either answer is acceptable if justified.

---

### Step 4 — Data model (5 minutes)

Entities, relationships, and *then* the database choice — in that order. Choosing the database first is choosing the answer before the question.

```
User    (id, name, phone, created_at)
Order   (id, user_id → User, restaurant_id → Restaurant, status, total, created_at)
OrderItem (id, order_id → Order, menu_item_id, qty, unit_price)
```

Then justify the storage:

| Data | Store | Why |
|------|-------|-----|
| Orders, users, payments | PostgreSQL | Relational, needs ACID transactions, money must not be approximate |
| Session / feed cache | Redis | Sub-millisecond reads, ephemeral by nature |
| Courier live location | Redis (or a geo/time-series store) | Written every few seconds, only the latest value matters, no durability needed |
| Order history archive | S3 / object storage | Cold, cheap, rarely read |
| Menu images | S3 + CDN | Static, large, must be geographically close to the user |

> **Index decisions come from Step 3.** You wrote `GET /orders?status=active` — so you
> need an index on `(user_id, status, created_at)`. Deriving indexes from your own API
> instead of sprinkling them randomly is exactly the "layer by layer" thinking you
> were asking for: each step's output is the next step's input.

---

### Step 5 — High-level design (10 minutes)

*Now* you draw. And because of steps 1–4, every box has a reason to exist.

```
[React Native App]
      │  HTTPS + JWT
      ▼
[CDN / Edge]────────────► static assets, menu images
      │
      ▼
[Load Balancer]
      │
      ▼
[API Gateway] ── auth, rate limiting, routing
      │
      ├──────────────┬──────────────────┐
      ▼              ▼                  ▼
[Order Service] [User Service]  [Tracking Service]
      │              │                  │
      ▼              ▼                  ▼
[PostgreSQL]    [PostgreSQL]        [Redis]
  primary +       replicas          (live locations)
  read replicas
      │
      ▼
[Message Queue] ──► [Notification Service] ──► push to device
```

**Narrate as you draw.** Silence is the enemy. *"I'm putting a cache here because step 2 showed a 1000:1 read-to-write ratio, so most of these reads never need to reach the database."* Every box, one sentence of justification.

**Start simple and add pressure.** A strong answer often begins: *"At 1,000 QPS I'd genuinely start with a modular monolith and one Postgres instance — it's operationally simpler and this scale doesn't require more. Let me show where it breaks first as we grow, and what I'd split out."* That is a senior answer. Reaching for microservices at 100 users is the classic junior tell.

---

### Step 6 — Deep dive and trade-offs (10 minutes)

The interviewer will pick a component and push. Or better — **you** pick the weak point first, which shows you can evaluate your own work.

Have an answer ready for each of these:

**"What breaks first as this grows?"**
> "The orders table. At 100k orders/day it's fine for years, but the feed query joining orders to restaurants to reviews will slow before storage does. I'd add a covering index first, then a read replica, and only shard if we hit ~50k writes/sec — which this workload won't."

**"What happens when component X dies?"**
Go through each box and answer honestly:
```
Cache dies      → traffic falls through to the DB. Is the DB sized to survive
                  that spike? If not, that's a cascading failure — I need a
                  circuit breaker and possibly stale-while-revalidate.
DB primary dies → failover to a replica. Some seconds of write downtime.
                  Do we lose in-flight writes? Depends on replication mode.
Queue backs up  → notifications are delayed but orders still succeed.
                  Correct degradation — the core path is unaffected.
Phone is offline→ queue the write locally, retry with backoff + idempotency key.
```

**"What did you trade away?"**
Every choice costs something. Naming the cost yourself is the strongest move available to you:
```
Cache            → gained speed, lost freshness (stale reads possible)
Read replicas    → gained read scale, lost read-your-own-writes consistency
Microservices    → gained team autonomy, lost easy transactions + simple debugging
Async queue      → gained responsiveness, lost immediate confirmation
JWT              → gained statelessness, lost instant revocation
Denormalisation  → gained read speed, lost write simplicity + integrity guarantees
```

> **There is no perfect design.** An interviewer asking "why did you do that?" is
> usually not saying you're wrong — they're checking whether you know the cost. "I
> chose X, which costs me Y, and I accept that because Z" is the complete answer
> shape. Memorise the shape, not the examples.

---

## 0.4 Time budget for a 45-minute interview

```
 0–5   min  Requirements and scope       ← never skip, never rush
 5–10  min  Scale estimation
10–15  min  API design
15–20  min  Data model
20–30  min  High-level architecture
30–40  min  Deep dive on 1–2 components
40–45  min  Failure modes, trade-offs, "what I'd do next"
```

If you're 20 minutes in and still on requirements, you've over-indexed. If you started drawing at minute 2, you've under-indexed. Watch the clock out loud: *"I want to leave time for the data model, so let me move on."*

---

## 0.5 The seven mistakes that sink candidates

1. **Designing before asking.** The most common and the most fatal.
2. **Jumping to microservices/Kafka/Kubernetes** for a problem that doesn't have them. Complexity you can't justify reads as inexperience, not expertise.
3. **Silence while thinking.** They cannot grade what they can't hear. Narrate.
4. **Naming technologies instead of properties.** "I'd use Redis" is weak. "I need sub-millisecond reads on ephemeral data, so an in-memory store — Redis" is strong. The property is the reasoning; the product is a detail.
5. **No numbers.** Without Step 2, every later decision is unjustifiable.
6. **Ignoring failure.** A design that only works when everything works isn't a design.
7. **Defending instead of listening.** When an interviewer pushes back, they're usually handing you a hint. Engage with it. Changing your mind on good evidence is a strength, not a retreat.

---

## 0.6 Practice — do these in order

Give yourself 45 minutes each, on paper, following the six steps. Do them **before** you read a solution anywhere.

| # | Problem | What it teaches |
|---|---------|----------------|
| 1 | A URL shortener | The whole process at minimum complexity. Hashing, read-heavy caching. |
| 2 | An offline-capable note-taking app | Sync, conflicts, local-first. Your home turf — start here if #1 feels dry. |
| 3 | A news feed (following model) | Fan-out on read vs write. The classic. |
| 4 | 1-to-1 chat with delivery receipts | WebSockets, ordering, presence, offline queueing. |
| 5 | Live delivery tracking | Geo data, high-frequency writes, battery constraints. |
| 6 | Push notification system | Queues, fan-out, retries, third-party (APNs/FCM) failure handling. |

After each one, check yourself against this list — it's the actual grading rubric:

- [ ] Did I scope the problem before designing?
- [ ] Did I produce at least one number, and did it change a decision?
- [ ] Can I justify every box in my diagram in one sentence?
- [ ] Did I say what happens when each component fails?
- [ ] Did I name what each choice cost me?
- [ ] Did I start simple and scale under pressure, rather than starting complex?

Six checkmarks is a pass at most companies, regardless of whether your architecture matches the "official" answer. There isn't one.

---

## Summary

| Step | Question it answers | Output |
|------|--------------------|--------|
| 1. Requirements | What are we building — and not building? | Scoped feature list + targets |
| 2. Scale | How big? | QPS, storage, read/write ratio |
| 3. API | What are the exact operations? | Endpoint signatures |
| 4. Data model | What do we store, where? | Entities + DB choices |
| 5. High-level | How do the pieces fit? | The diagram |
| 6. Deep dive | Where does it break, and what did it cost? | Bottlenecks + trade-offs |

**The one sentence to keep:** system design is not knowing components, it's knowing *which* components, *why*, and *what you gave up* — and the six steps are how you get there reliably instead of by luck.

---

*Next: [Floor 1 — Fundamentals](../Floor1-Fundamentals/resource.md)*
