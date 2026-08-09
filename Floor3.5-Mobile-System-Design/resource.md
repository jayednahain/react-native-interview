# Floor 3.5 — Mobile System Design

> **Prerequisites:** Floor 0 (the process) + Floor 3 (distributed systems vocabulary)
> **Why this floor exists:** "Mobile system design" is a real, separately-interviewed
> discipline — and it is the one where you are already an expert without knowing it.
> Everything here is a problem you have solved on the job. This floor gives it names.

---

## 3.5.1 Why the client is a distributed system

You wrote in your original question that app development is harder than web front-end because you manage devices, connectivity, storage, and performance. That instinct is right, and here is the precise reason:

**Your app is not a client. It is a replica of your backend's data, running on hardware you don't control, on a network that fails constantly, that must stay useful while disconnected.**

That is the definition of a node in a distributed system. Every hard problem from Floor 3 shows up on the device:

| Floor 3 concept | Where it lives in your app |
|-----------------|---------------------------|
| Eventual consistency | Local cache vs server truth |
| Partition tolerance | Offline mode — this is a permanent partition, by design |
| Conflict resolution | Two devices editing the same record |
| Idempotency | Retry after a request times out on a dying connection |
| Replication lag | User posts, then pulls-to-refresh and doesn't see it |
| Backpressure | 200 queued offline writes reconnecting at once |
| Circuit breaker | Stop hammering an API that's clearly down |

A backend engineer gets to assume reliable power, reliable network, and a machine they can restart. You get none of those. **You have the harder consistency problem** — you just have fewer users per node.

**The constraints that are uniquely yours:**

```
Battery      — a wakeup every 30s is a 1-star review about battery drain
Data plan    — some of your users pay per megabyte
Storage      — you can be evicted from disk by the OS at any time
Frame budget — 16.7ms at 60fps. Blow it and users feel it, even if it "works"
Cold start   — every millisecond before first paint is measured
Upgrades     — you cannot force them. Old versions live for YEARS
Store review — you cannot hotfix native code in under 24 hours
OS lifecycle — you get killed in the background without warning
```

That last group is the real difference from web. **A web front-end deploys instantly and everyone is on the newest version within a day. Your v2.1 users will still be hitting your API in eighteen months.** Every mobile API design decision flows from that fact.

---

## 3.5.2 Client state architecture — the split that fixes most apps

The most common architectural mistake in React Native is treating all state as one thing and putting it all in Redux. There are **four** kinds, and they want different tools.

| Kind | Example | Lives in | Lifetime |
|------|---------|----------|----------|
| **Server cache** | The user's orders, the feed | React Query / SWR / RTK Query | Until invalidated |
| **Global client state** | Theme, auth status, feature flags | Zustand / Context / Redux | App session |
| **Local UI state** | Is this modal open, form input | `useState` | Component |
| **Persisted state** | Draft messages, offline queue, tokens | MMKV / SQLite / SecureStore | Across launches |

```typescript
// ❌ Common mistake — server data in a global store, hand-managed
const useStore = create((set) => ({
  orders: [],
  loading: false,
  error: null,
  fetchOrders: async () => {
    set({ loading: true });
    try {
      const orders = await api.getOrders();
      set({ orders, loading: false });
    } catch (e) { set({ error: e, loading: false }); }
  },
}));
// You now hand-write: caching, deduplication, refetch-on-focus, retry,
// stale detection, pagination, and invalidation. For every entity. Forever.

// ✅ Server data is a CACHE of someone else's state — use a cache library
const useOrders = () => useQuery({
  queryKey: ['orders'],
  queryFn: api.getOrders,
  staleTime: 30_000,       // Trust it for 30s without refetching
  gcTime: 10 * 60_000,     // Keep it in memory 10min after last use
});

// ✅ Global CLIENT state is genuinely yours — a small store is right
const useAppStore = create((set) => ({
  theme: 'system',
  setTheme: (theme) => set({ theme }),
}));
```

> **The principle worth stating in an interview:** *"Server state isn't state, it's a
> cache — and caching is a solved problem I shouldn't re-solve per feature."* This is
> Floor 3's caching section applied on the client. Same concept, different floor.

---

## 3.5.3 Offline-first and sync — the flagship mobile design problem

If you get asked one mobile system design question, it will be this one, in some disguise. "Design offline mode for a notes app / chat / delivery tracker" is all the same problem.

### The three levels of offline

```
Level 1 — Read-only offline
  Cache the last fetched data, show it when disconnected, banner says "offline".
  Difficulty: low. Covers ~70% of apps.

Level 2 — Queued writes
  Accept user actions offline, queue them, replay when reconnected.
  Difficulty: medium. Needs idempotency + ordering. Covers most of the rest.

Level 3 — Full bidirectional sync
  Multiple devices edit the same data offline; merge on reconnect.
  Difficulty: high. This is where CRDTs and conflict resolution live.
```

**Say which level you're building and why.** Building Level 3 when the product needs Level 1 is the mobile equivalent of premature microservices.

### The Outbox pattern (Level 2)

The standard answer. Writes go to a local durable queue first, and the UI reads from local state immediately.

```typescript
// The user's action is recorded LOCALLY and durably, before any network attempt.
type OutboxEntry = {
  id: string;              // Also the idempotency key — generated ONCE, here
  operation: 'CREATE_NOTE' | 'UPDATE_NOTE' | 'DELETE_NOTE';
  payload: unknown;
  createdAt: number;
  attempts: number;
  status: 'pending' | 'syncing' | 'failed';
};

const enqueue = async (operation: OutboxEntry['operation'], payload: unknown) => {
  const entry: OutboxEntry = {
    id: uuidv4(),
    operation,
    payload,
    createdAt: Date.now(),
    attempts: 0,
    status: 'pending',
  };

  await db.outbox.insert(entry);   // Durable: survives an app kill
  await applyLocally(entry);       // Optimistic: UI updates NOW
  syncEngine.wake();               // Try to flush, but don't block the user
  return entry.id;
};

// The flush loop — runs on reconnect, on app foreground, and after each success
const flush = async () => {
  if (!(await NetInfo.fetch()).isConnected) return;

  // Order matters: a DELETE must not overtake the CREATE of the same note
  const pending = await db.outbox.where({ status: 'pending' }).orderBy('createdAt');

  for (const entry of pending) {
    try {
      await db.outbox.update(entry.id, { status: 'syncing' });

      await api.send(entry.operation, entry.payload, {
        headers: { 'Idempotency-Key': entry.id },  // Safe to retry — Floor 3.11
      });

      await db.outbox.delete(entry.id);
    } catch (error) {
      const attempts = entry.attempts + 1;

      if (isPermanentFailure(error)) {
        // 400/403/422 will never succeed. Retrying forever blocks the queue
        // and burns battery. Surface it to the user instead.
        await db.outbox.update(entry.id, { status: 'failed' });
        await revertLocally(entry);
        notifyUser(entry);
        continue;
      }

      await db.outbox.update(entry.id, { status: 'pending', attempts });
      if (attempts > 5) scheduleBackgroundRetry(entry);
      break;  // Stop on transient failure — preserve ordering, don't hammer
    }
  }
};
```

Four details that separate a real answer from a textbook one, and all four come from having been burned:

1. **The idempotency key is the outbox row's ID.** Generated once when the user acts, reused on every retry forever. This is the whole reason double-orders don't happen.
2. **Permanent vs transient failures are handled differently.** A 422 retried 500 times blocks every write behind it and drains the battery. Failed-permanently must be a visible state.
3. **Order is preserved by stopping on failure**, not by skipping ahead. Skipping produces "delete before create" bugs that are miserable to debug.
4. **The queue is durable, not in-memory.** The OS kills backgrounded apps without warning. An in-memory queue silently loses the user's work — which they will notice and you won't.

### Conflict resolution (Level 3)

Two devices edited the same note offline. Both reconnect. Now what?

| Strategy | How it works | Cost |
|----------|-------------|------|
| **Last-write-wins (LWW)** | Highest timestamp wins | Trivial to build. **Silently destroys data.** Fine for settings, wrong for documents. |
| **Server-wins** | Client discards its version | Simple, predictable, loses offline work |
| **Client-wins** | Client overwrites the server | Simple, dangerous with multiple devices |
| **Version vectors** | Each record carries a version; mismatch = conflict | Detects conflicts reliably, but someone must resolve them |
| **Prompt the user** | Show both, let them choose | Honest, and often correct. Users hate silent data loss more than a dialog. |
| **CRDTs** | Data types that merge deterministically (Yjs, Automerge) | Genuinely automatic merges — real complexity and payload size cost |

```typescript
// Version-vector conflict detection — the practical middle ground
const pushUpdate = async (note: Note) => {
  const response = await api.patch(`/notes/${note.id}`, {
    ...note.changes,
    baseVersion: note.serverVersion,   // "I edited version 7"
  });

  if (response.status === 409) {
    // Server is on version 9 — someone else changed it while we were offline
    const serverNote = response.data.current;
    return resolveConflict(note, serverNote);   // Merge, or ask the user
  }

  return response.data;
};
```

> **The strongest thing you can say here:** *"I'd pick last-write-wins only for
> data where losing an edit is acceptable — like a theme preference. For anything the
> user typed, silent LWW is data loss, and users will not forgive it. I'd detect the
> conflict with a version and surface it."* That single distinction is a senior-level
> answer, and most candidates never make it.

---

## 3.5.4 Designing APIs *for* mobile clients

Backend engineers design APIs for the database. You know what actually hurts on a phone.

### The round-trip tax

```
Screen needs: user profile + recent orders + loyalty points

❌ Three sequential calls on 4G:  100ms + 100ms + 100ms = 300ms of dead time
⚠️  Three parallel calls:          ~120ms, but 3× connection + auth overhead,
                                   3× the chance one fails
✅ One aggregated endpoint:        ~110ms, one failure mode, one loading state
```

The **BFF (Backend For Frontend)** pattern exists for this: a thin service that composes internal service calls into exactly the payload one screen needs. GraphQL is one implementation; a plain `GET /v1/screens/home` endpoint is another, and is often the better trade.

> Asking *"can we add an endpoint shaped like the screen, so we're not making three
> round trips on cellular?"* is a mobile-specific insight backend-only candidates
> rarely raise. It's a differentiator — use it.

### Payload size actually matters

```javascript
// ❌ 400KB of JSON — the phone must download it, parse it (blocking the JS
//    thread), and hold it in memory
GET /products   → 500 products, every field, full-resolution image URLs

// ✅ Page it, trim it, and let the CDN size the images
GET /products?limit=20&cursor=...&fields=id,name,price,thumbnailUrl
```

JSON parsing is not free — it happens on the JS thread and a large payload will drop frames. This is a system design decision with a directly visible UX consequence, which is not something backend engineers usually have to reason about.

### Versioning and the forced upgrade

Because old app versions live for years, you need both of these from day one:

```javascript
// Every response carries a minimum-supported-version signal
{
  "data": { },
  "meta": {
    "minSupportedVersion": "2.1.0",   // Below this: hard block, force update
    "latestVersion":       "3.4.0"    // Below this: soft nudge, dismissible
  }
}
```

```typescript
// A kill switch you can flip from the server, without a store release
const useVersionGate = () => {
  const { data } = useQuery({ queryKey: ['config'], queryFn: api.getConfig });

  if (!data) return 'ok';
  if (semverLt(DeviceInfo.getVersion(), data.minSupportedVersion)) return 'blocked';
  if (semverLt(DeviceInfo.getVersion(), data.latestVersion))       return 'nudge';
  return 'ok';
};
```

Ship this in version 1.0. If you add it in 2.0, it does nothing for the 1.x users who are exactly the ones you'll eventually need to block.

---

## 3.5.5 Push notifications — the architecture

Push is a system design question because delivery is **not guaranteed** and the moving parts are all outside your control.

```
[Your Backend]
     │  1. Event: "order confirmed"
     ▼
[Notification Service]
     │  2. Look up the user's device tokens (a user has MANY devices)
     │  3. Check preferences, quiet hours, dedupe
     ▼
[APNs (iOS)]  /  [FCM (Android)]
     │  4. Third party. Best-effort. Can drop, delay, or reject.
     ▼
[Device]
     │  5. May be off, offline, in Low Power Mode, or have notifications denied
     ▼
[Your App]
```

**Token lifecycle — where the bugs are:**

```typescript
// Tokens rotate: reinstall, restore-from-backup, OS update, ~monthly refresh.
// A stale token silently delivers nothing. Register on EVERY launch, not just
// the first — this is the single most common push bug in production.
useEffect(() => {
  const register = async () => {
    const authStatus = await messaging().requestPermission();
    if (!authStatus) return;

    const token = await messaging().getToken();
    await api.post('/devices', { token, platform: Platform.OS });
  };

  register();

  // And handle rotation while running
  const unsub = messaging().onTokenRefresh(token =>
    api.post('/devices', { token, platform: Platform.OS })
  );
  return unsub;
}, []);
```

**The rules that matter in design discussions:**

1. **Push is a hint, not a transport.** Never send data that exists *only* in the notification. The payload may never arrive. Send "something changed, come fetch it" and let the app pull the truth.
2. **Delete tokens on logout**, or the next user of that device gets someone else's notifications — a genuine privacy incident.
3. **One user has many devices.** Notifications fan out; read-state should sync so dismissing on the phone clears the tablet.
4. **Deduplicate server-side.** Retried queue messages (Floor 4.6) will otherwise buzz the user three times for one order.

---

## 3.5.6 Startup, performance, and the frame budget

### Cold start

```
Process spawn → native init → JS bundle load → JS execution
              → first render → data fetch → content visible
                                            ↑ this is what the user measures
```

Design decisions that move this number:

- **Don't block first paint on the network.** Render cached content immediately, refresh underneath. A skeleton is better than a spinner; real stale data is better than either.
- **Lazy-load routes.** Not every screen's code needs to parse at launch.
- **Defer non-critical init** — analytics, crash reporting config, remote-config fetch — until after first interactive frame, via `InteractionManager` (Floor 1).
- **Hermes with precompiled bytecode** removes JS parse time at startup.

### The 16.7ms budget

At 60fps you have 16.7ms per frame for *everything*. This gives you a concrete framework for performance work rather than guesswork:

| Symptom | Usual cause | Fix |
|---------|------------|-----|
| Jank while scrolling | Re-rendering whole list on every item change | `memo`, stable keys, `getItemLayout`, FlashList |
| Freeze on screen open | Synchronous heavy work on the JS thread | Defer with `InteractionManager`, chunk the work |
| Animation stutters | Animation driven from the JS thread | `useNativeDriver: true`, or Reanimated worklets (UI thread) |
| Slow images | Full-resolution downloads | CDN-resized variants, `expo-image`/FastImage caching |
| Memory growth | Unbounded caches, retained listeners | TTL + size caps, clean up subscriptions |

> Frame budget is genuinely the same discipline as backend latency budgets from
> Floor 0 — a fixed amount of time, divided among competing consumers, where
> exceeding it degrades the product. You've been doing capacity planning at 16.7ms
> resolution this whole time.

---

## 3.5.7 Release engineering — the part web developers never learn

You cannot roll back an app the way you roll back a server. This changes everything about how you ship.

```
Backend rollback:  kubectl rollout undo    →  fixed in 30 seconds
Mobile rollback:   submit a new build      →  review queue, then users must
                                              choose to update. Days. Some never do.
```

**Therefore, mobile release safety is built in advance, not applied after:**

| Mechanism | What it buys you |
|-----------|-----------------|
| **Staged rollout** (1% → 10% → 50% → 100%) | Catch a crash spike while it affects 1% of users |
| **Feature flags** | Ship code dark; enable server-side; disable instantly without a release |
| **Kill switch** | Turn off a broken feature for everyone in seconds |
| **Forced upgrade gate** | Block versions with a critical bug (see 3.5.4) |
| **OTA JS updates** | Fix JS-only bugs in hours instead of days (Floor 5.4) |
| **Crash-rate monitoring** | Halt the rollout automatically on regression |

```typescript
// Feature flags decouple "deployed" from "released" — the core idea
const NewCheckout = () => {
  const enabled = useFeatureFlag('new_checkout_flow');
  return enabled ? <CheckoutV2 /> : <CheckoutV1 />;
};
// The code ships to 100% of users. The FEATURE turns on for 5% of them.
// If the conversion rate drops, you flip it off — no store review involved.
```

> **The interview line:** *"Because I can't roll back a mobile release, every risky
> change ships behind a flag and goes out in a staged rollout. The deploy and the
> release are separate events."* That reframing is the whole discipline in one
> sentence, and it's rare enough to be memorable.

---

## 3.5.8 Client-side security

```typescript
// The foundational rule: THE CLIENT IS HOSTILE TERRITORY.
// Anyone can extract your APK, read the bundle, and inspect your traffic.
```

| Concern | The answer |
|---------|-----------|
| Tokens at rest | Keychain / Keystore (SecureStore) — never AsyncStorage or MMKV |
| API keys in the app | There is no safe way. Proxy through your own server. |
| Traffic inspection | TLS always; certificate pinning for high-value apps |
| Tampered/rooted devices | Play Integrity / App Attest — a signal, never a guarantee |
| Reverse engineering | Obfuscation raises cost, it does not prevent |
| Sensitive screens | Block screenshots, blur in the app switcher |
| Client-side validation | UX only. **Every rule must be re-enforced on the server.** |

> **The mental model:** every client-side control is a speed bump for an attacker and
> a convenience for an honest user. The security boundary is your server, always.
> Candidates who say "we validate on the client so the server doesn't have to" fail
> the security question outright.

---

## 3.5.9 A worked example — "Design the chat feature in our app"

Run Floor 0's six steps, with mobile constraints in the foreground.

**1. Requirements.** 1-to-1 chat, text only. Must work offline (compose + queue). Delivery/read receipts. Push when backgrounded. History across devices. *Out of scope:* groups, media, calls, E2E encryption.

**2. Scale.** 500k DAU, ~40 messages/user/day = 20M messages/day ≈ 230 writes/sec average, ~700 peak. Reads ~5×. 20M × 1KB = 20GB/day. Chat is **write-heavy** — unusual, and it changes the storage choice.

**3. API.**
```
WS   /v1/chat              → connect, auth by token; server pushes new messages
POST /v1/conversations/:id/messages   (Idempotency-Key: <clientMessageId>)
GET  /v1/conversations/:id/messages?before=<cursor>&limit=50
POST /v1/conversations/:id/read       { upToMessageId }
```

**4. Data model.** `Conversation(id, participantIds, lastMessageAt)`, `Message(id, conversationId, senderId, body, createdAt, serverSeq)`. Write-heavy + time-ordered + rarely updated → Cassandra or DynamoDB partitioned by `conversationId`, sorted by `serverSeq`. On-device: SQLite mirror plus the outbox.

**5. High-level.**
```
[RN App] ←─WebSocket─→ [Chat Gateway] ─→ [Message Service] ─→ [Cassandra]
   │                        │                    │
[SQLite +                [Redis:               [Queue] ─→ [Push Service] → APNs/FCM
 outbox]                 presence,                          (only if socket is dead)
                         socket routing]
```

**6. Deep dive — the mobile-specific problems:**

- **Message ordering.** Client timestamps are unreliable (clock skew, offline composition). The **server assigns a monotonic `serverSeq` per conversation** and that's the sort key. The client shows optimistic messages at the end until the server sequence arrives, then reconciles.
- **Optimistic send.** Message appears instantly with a `clientMessageId`, marked "sending". The server echoes that ID back so the client can match the confirmed message to the optimistic one instead of showing a duplicate.
- **Offline compose.** Straight into the outbox (3.5.3). The `clientMessageId` is the idempotency key, so a retry after a timeout can't double-send.
- **Sync on reconnect.** Don't refetch everything. Send the last known `serverSeq` and receive only the delta. This is the difference between a 50KB reconnect and a 20MB one on someone's data plan.
- **Push vs socket.** Socket alive → deliver over the socket. Socket dead (backgrounded/killed) → the queue triggers a push carrying only "new message in conversation X", and the app fetches on open. Never put the message body solely in the push payload.
- **Failure modes.** Socket drops → exponential backoff with jitter, fall back to polling. Push service down → messages still arrive when the app opens; degraded, not broken. Device offline for a week → the delta sync is bounded by a page limit, and the client pages through.

That answer is *mostly mobile concerns*, and it is a strong answer. The backend in it is deliberately unremarkable, because for this feature the interesting engineering genuinely lives on the client.

---

## Summary

| Concept | Key takeaway |
|---------|-------------|
| The client is a replica | Offline is a permanent network partition. CAP applies on the device. |
| Four kinds of state | Server cache ≠ global state ≠ UI state ≠ persisted state. Different tools. |
| Outbox pattern | Durable local queue + idempotency key. The answer to "design offline mode". |
| Conflict resolution | LWW silently loses data. Detect with versions; ask the user when it matters. |
| BFF / aggregation | Round trips cost ~100ms each on cellular. Shape endpoints like screens. |
| Forced upgrade gate | Ship it in v1.0 or it can't help you later. Old versions live for years. |
| Push is a hint | Delivery is never guaranteed. Send "come fetch", not the payload. |
| Token registration | Register on every launch and on refresh, not just first run. |
| 16.7ms budget | The client's equivalent of a latency budget. |
| Deploy ≠ release | Flags + staged rollout, because there is no mobile rollback. |
| The client is hostile | No secrets on device. Every client-side check is re-checked server-side. |

---

*Next: [Floor 4 — Backend Architecture](../Floor4-Backend-Architecture/resource.md)*
