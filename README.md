# System Design, Floor by Floor

A system design curriculum written for **React Native / mobile developers** — people who already build real systems for a living and want the vocabulary, the missing layers, and the interview process to go with it.

**Start at [Floor 0](./Floor0-The-Design-Process/resource.md).** It's the shortest floor and the only one that teaches a *procedure* rather than facts. Everything else is easier afterwards.

---

## The building

| Floor | Topic | Read it for |
|-------|-------|-------------|
| **0** | [The Design Process](./Floor0-The-Design-Process/resource.md) | The six-step method for actually *doing* a design. What system design means, vs architecture. Scale math. Practice problems. |
| **1** | [Fundamentals](./Floor1-Fundamentals/resource.md) | HTTP, TCP/IP, DNS, TLS, data structures, the event loop, RN threading. The floor you already live on. |
| **2** | [OOP, SOLID and Patterns](./Floor2-OOP-SOLID/resource.md) | Abstraction, inheritance, interfaces vs abstract vs concrete classes, SOLID, design patterns, MVVM and Clean Architecture. |
| **3** | [High-Level System Design](./Floor3-HighLevel-SystemDesign/resource.md) | Monolith vs microservices, sync vs async, scaling, caching, CAP, circuit breakers, idempotency, reliability patterns. |
| **3.5** | [Mobile System Design](./Floor3.5-Mobile-System-Design/resource.md) | **The mobile-specific interview.** Offline-first sync, the outbox pattern, conflict resolution, push architecture, release engineering, client security. |
| **4** | [Backend Architecture](./Floor4-Backend-Architecture/resource.md) | REST design, auth (JWT/OAuth/RBAC), SQL vs NoSQL, indexes, transactions, deadlocks, N+1 queries, caching, queues, GraphQL. |
| **5** | [Infrastructure and DevOps](./Floor5-Infrastructure/resource.md) | Docker, Kubernetes, CI/CD, Fastlane, OTA updates, scaling, AWS, serverless, observability. |
| — | [Condensed overview](./SystemDesignResource.md) | All five original floors summarised on one page. Good for revision, not for learning. |

---

## Two ways through it

**If you're preparing for interviews (fastest useful path):**
```
Floor 0  →  Floor 3  →  Floor 3.5  →  Floor 4  →  skim Floor 5
```
Floors 0, 3 and 3.5 are where mobile system design interviews actually happen.

**If you're filling in foundations (thorough path):**
```
Floor 0 → 1 → 2 → 3 → 3.5 → 4 → 5, then Floor 0's practice problems
```

Floor 3.5 sits deliberately between 3 and 4: you need Floor 3's vocabulary to describe client problems properly, but you shouldn't wait until you've learned Kubernetes to study the thing you do every day.

---

## Am I ready to move up?

Don't move to the next floor because you finished reading. Move when you can answer these **out loud, without notes**. If you can't, the gap is specific and you know exactly what to re-read.

**Leaving Floor 0**
- [ ] I can name the six steps in order and say why each one blocks the next
- [ ] I can estimate QPS from daily active users in my head
- [ ] I can state the difference between system design and system architecture in one sentence

**Leaving Floor 1**
- [ ] I can trace what happens between `fetch()` and a rendered list, naming DNS, TCP, TLS, HTTP
- [ ] I can explain why a `for` loop over 100,000 items drops frames
- [ ] I can say when to use a Map vs an object vs a Set, and why

**Leaving Floor 2**
- [ ] I can explain abstraction, encapsulation, inheritance and polymorphism with examples from my own code
- [ ] I can name all five SOLID principles and show a violation of each
- [ ] I can explain when an interface beats an abstract class
- [ ] I can say what a Clean Architecture domain layer is for — and when it isn't worth it

**Leaving Floor 3**
- [ ] I can argue *both sides* of monolith vs microservices
- [ ] I can explain CAP without saying "pick two"
- [ ] I can explain why a POST needs an idempotency key and where the key comes from
- [ ] I can describe what happens to the database when the cache dies

**Leaving Floor 3.5**
- [ ] I can design offline mode for a note-taking app, including the write queue
- [ ] I can explain why last-write-wins is dangerous, and what I'd do instead
- [ ] I can explain why a forced-upgrade gate must ship in version 1.0
- [ ] I can explain why push notifications must not carry the only copy of the data

**Leaving Floor 4**
- [ ] I can design a REST API for a feature, with pagination and error shapes
- [ ] I can explain the access/refresh token split and what JWTs cost you
- [ ] I can spot an N+1 query and give two fixes
- [ ] I can explain a deadlock and three ways to prevent it

**Leaving Floor 5**
- [ ] I can explain what Docker actually solves, and why layer order matters
- [ ] I can explain what Kubernetes does that Docker alone doesn't
- [ ] I can explain why horizontal scaling requires stateless services
- [ ] I can name the three pillars of observability and what each answers

---

## How to use this if you're short on time

Reading all seven floors is a few weeks of evenings. If that's not available:

1. Read **Floor 0** end to end. Non-negotiable — it's the load-bearing one.
2. Do practice problem #2 (offline note-taking app) on paper. Badly. That's fine.
3. Read **Floor 3.5**, then redo the same problem. The difference between your two attempts *is* the learning, and it will be obvious.
4. Repeat with problems 3–6, reading whichever floor each one exposes as your weak spot.

Passive reading builds recognition. Attempting a design first, then reading, builds recall — and recall is what an interview tests.
