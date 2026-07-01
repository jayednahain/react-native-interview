# Floor 2 — OOP and Design Patterns

> **Prerequisites:** Floor 1 — Fundamentals
> **Language used:** TypeScript (standard in React Native projects). JavaScript equivalents shown where relevant.

---

## 2.1 The Four Pillars of OOP

### Abstraction

Hiding complexity behind a simple interface. The caller doesn't need to know *how* something works — only *what* it does.

```typescript
// You call this one line:
const user = await userRepository.getUser('user_123');

// You don't see any of this happening inside:
// → checks in-memory cache
// → if miss, checks AsyncStorage (local DB)
// → if miss, makes HTTP request to API
// → maps raw JSON to a typed User object
// → stores result in cache
// → returns the User

// All of that complexity is hidden. That is abstraction.
```

In React Native, you use abstraction every day:
- `camera.takePicture()` — hides thousands of lines of camera driver code
- `navigation.navigate('ProfileScreen')` — hides stack manipulation, transitions, gesture handling
- `useQuery('/api/users')` — hides caching, refetching, loading/error state management

```typescript
// Good abstraction — caller doesn't know or care where data comes from
class UserRepository {
  private cache = new Map<string, User>();

  async getUser(id: string): Promise<User> {
    if (this.cache.has(id)) {
      return this.cache.get(id)!;
    }

    const response = await fetch(`https://api.myapp.com/users/${id}`);
    const user: User = await response.json();

    this.cache.set(id, user);
    return user;
  }
}
```

### Encapsulation

Keeping internal state **private**, and only exposing what is necessary through a controlled interface.

```typescript
class AuthSession {
  // Private — nothing outside this class can touch these directly
  #accessToken: string = '';
  #refreshToken: string = '';
  #expiresAt: number = 0;

  // Public API — the only way to interact with session state
  get isValid(): boolean {
    return this.#accessToken !== '' && Date.now() < this.#expiresAt;
  }

  get needsRefresh(): boolean {
    // Token expires in less than 5 minutes
    return this.#expiresAt - Date.now() < 5 * 60 * 1000;
  }

  login(accessToken: string, refreshToken: string, expiresIn: number): void {
    this.#accessToken = accessToken;
    this.#refreshToken = refreshToken;
    this.#expiresAt = Date.now() + expiresIn * 1000;
  }

  logout(): void {
    this.#accessToken = '';
    this.#refreshToken = '';
    this.#expiresAt = 0;
  }

  getAuthHeader(): string {
    return `Bearer ${this.#accessToken}`;
  }
}

// Usage — caller can only do what we allow
const session = new AuthSession();
session.login('eyJhbGci...', 'refresh_token_here', 3600);
session.isValid;      // true
session.getAuthHeader(); // 'Bearer eyJhbGci...'
// session.#accessToken  // ❌ TypeScript error — private field
```

### Inheritance

A class inherits properties and methods from a parent class, and can extend or override behavior.

```typescript
// Base class — shared behavior for all screens
class BaseScreen {
  protected analytics: Analytics;

  constructor() {
    this.analytics = new Analytics();
  }

  protected trackScreenView(screenName: string): void {
    this.analytics.logEvent('screen_view', { screen: screenName });
  }

  protected handleError(error: Error): void {
    console.error(error);
    this.analytics.logError(error);
  }
}

// Subclass — inherits base behavior, adds its own
class ProfileScreen extends BaseScreen {
  private userId: string;

  constructor(userId: string) {
    super(); // Must call parent constructor
    this.userId = userId;
    this.trackScreenView('ProfileScreen'); // Inherited method
  }

  async loadProfile(): Promise<void> {
    try {
      const profile = await userRepository.getUser(this.userId);
      this.renderProfile(profile);
    } catch (error) {
      this.handleError(error as Error); // Inherited error handler
    }
  }

  private renderProfile(profile: User): void {
    // Profile-specific rendering logic
  }
}
```

> **Rule:** Only use inheritance for genuine "is-a" relationships. `ProfileScreen` IS-A `BaseScreen` ✅. Prefer **composition** (see 2.5) when the relationship is "has-a" or "uses-a".

### Polymorphism

The same interface, different behavior depending on which class implements it.

```typescript
// Define a contract (interface)
interface PaymentProvider {
  charge(amount: number, currency: string): Promise<PaymentResult>;
  refund(transactionId: string, amount: number): Promise<RefundResult>;
}

// Multiple implementations of the same contract
class StripeProvider implements PaymentProvider {
  async charge(amount: number, currency: string): Promise<PaymentResult> {
    // Stripe-specific API call
    return callStripeAPI('/charges', { amount, currency });
  }

  async refund(transactionId: string, amount: number): Promise<RefundResult> {
    return callStripeAPI(`/refunds`, { charge: transactionId, amount });
  }
}

class PayPalProvider implements PaymentProvider {
  async charge(amount: number, currency: string): Promise<PaymentResult> {
    // PayPal-specific API call — completely different implementation
    return callPayPalAPI('/v2/payments/captures', { amount, currency });
  }

  async refund(transactionId: string, amount: number): Promise<RefundResult> {
    return callPayPalAPI(`/v2/payments/captures/${transactionId}/refund`, { amount });
  }
}

// Your business logic doesn't care which provider is behind it
class CheckoutService {
  constructor(private provider: PaymentProvider) {}

  async processOrder(order: Order): Promise<void> {
    // Works with Stripe, PayPal, or any future provider
    const result = await this.provider.charge(order.total, order.currency);
    if (!result.success) throw new Error('Payment failed');
    await orderRepository.markAsPaid(order.id, result.transactionId);
  }
}

// Swap the provider without changing CheckoutService
const checkout = new CheckoutService(new StripeProvider());
// const checkout = new CheckoutService(new PayPalProvider()); // Just change this line
```

---

## 2.2 Classes, Abstract Classes, and Interfaces

TypeScript gives you all three. Understanding when to use each is important.

| Type | Can instantiate? | Can have implementation? | Use when |
|------|-----------------|--------------------------|----------|
| **Interface** | No | No (structural contract only) | Defining *what* something can do |
| **Abstract class** | No | Partial (some methods implemented) | Sharing base implementation + forcing overrides |
| **Concrete class** | Yes | Yes (fully implemented) | Ready-to-use objects |

```typescript
// INTERFACE — pure contract, no implementation
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

// ABSTRACT CLASS — partial implementation + enforced overrides
abstract class BaseApiClient {
  protected baseURL: string;
  protected headers: Record<string, string>;

  constructor(baseURL: string, token: string) {
    this.baseURL = baseURL;
    this.headers = {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    };
  }

  // Implemented — all subclasses inherit this
  protected async get<T>(path: string): Promise<T> {
    const response = await fetch(`${this.baseURL}${path}`, {
      headers: this.headers,
    });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }

  // Abstract — each subclass must implement its own
  abstract getResourceName(): string;
}

// CONCRETE CLASS — fully implemented
class UserApiClient extends BaseApiClient implements Repository<User> {
  getResourceName() { return 'users'; }

  async findById(id: string): Promise<User | null> {
    return this.get<User>(`/users/${id}`);
  }

  async findAll(): Promise<User[]> {
    return this.get<User[]>('/users');
  }

  async save(user: User): Promise<User> {
    const response = await fetch(`${this.baseURL}/users`, {
      method: 'POST',
      headers: this.headers,
      body: JSON.stringify(user),
    });
    return response.json();
  }

  async delete(id: string): Promise<void> {
    await fetch(`${this.baseURL}/users/${id}`, {
      method: 'DELETE',
      headers: this.headers,
    });
  }
}
```

---

## 2.3 SOLID Principles

Five rules for writing OOP code that stays maintainable as it grows.

### S — Single Responsibility Principle

A class should do **one thing** and have **one reason to change**.

```typescript
// ❌ WRONG — UserManager does too many unrelated things
class UserManager {
  async getUser(id: string) { /* fetch from API */ }
  async saveToDatabase(user: User) { /* DB logic */ }
  sendWelcomeEmail(user: User) { /* email logic */ }
  formatForDisplay(user: User) { /* UI logic */ }
  validatePassword(password: string) { /* validation logic */ }
}

// ✅ RIGHT — each class has exactly one job
class UserRepository {
  async findById(id: string): Promise<User> { /* fetch from API/DB */ }
  async save(user: User): Promise<User> { /* persist */ }
}

class UserEmailService {
  sendWelcomeEmail(user: User): void { /* send email */ }
  sendPasswordResetEmail(user: User, token: string): void { /* send email */ }
}

class UserValidator {
  isValidEmail(email: string): boolean { /* validation */ }
  isStrongPassword(password: string): boolean { /* validation */ }
}

class UserViewModel {
  formatName(user: User): string { return `${user.firstName} ${user.lastName}`; }
  getAvatarUrl(user: User): string { /* format URL */ }
}
```

In React Native: your components should render UI. Your hooks should manage state. Your services should handle data fetching. Don't mix these.

```typescript
// ❌ WRONG — component doing too much
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(data => {
        // Transforming data inside the component
        setUser({
          ...data,
          fullName: `${data.firstName} ${data.lastName}`,
          age: new Date().getFullYear() - new Date(data.birthYear).getFullYear(),
        });
      });
  }, [userId]);

  return <View>...</View>;
};

// ✅ RIGHT — responsibilities separated
const useUserProfile = (userId: string) => {
  const [user, setUser] = useState<UserProfile | null>(null);
  useEffect(() => {
    userService.getUserProfile(userId).then(setUser);
  }, [userId]);
  return user;
};

const UserProfile = ({ userId }) => {
  const user = useUserProfile(userId); // Hook owns data logic
  return <View>...</View>;            // Component owns only rendering
};
```

### O — Open/Closed Principle

A class should be **open for extension** but **closed for modification**. Add new behavior by adding new code — not by editing existing code.

```typescript
// ❌ WRONG — every new notification type requires editing this class
class NotificationService {
  send(type: string, message: string) {
    if (type === 'push') {
      sendPushNotification(message);
    } else if (type === 'email') {
      sendEmail(message);
    } else if (type === 'sms') {
      sendSMS(message);
    }
    // Adding 'in-app' requires editing here — risky
  }
}

// ✅ RIGHT — add a new notification type without touching existing code
interface NotificationChannel {
  send(message: string): Promise<void>;
}

class PushNotificationChannel implements NotificationChannel {
  async send(message: string): Promise<void> {
    await pushService.notify(message);
  }
}

class EmailNotificationChannel implements NotificationChannel {
  async send(message: string): Promise<void> {
    await emailService.send(message);
  }
}

// Adding InAppNotificationChannel — zero changes to existing code
class InAppNotificationChannel implements NotificationChannel {
  async send(message: string): Promise<void> {
    await inAppBanner.show(message);
  }
}

class NotificationService {
  constructor(private channels: NotificationChannel[]) {}

  async sendAll(message: string): Promise<void> {
    await Promise.all(this.channels.map(c => c.send(message)));
  }
}
```

### L — Liskov Substitution Principle

A subclass must be usable **wherever its parent is expected**, without breaking anything. The subclass must honor the parent's contract.

```typescript
// ❌ WRONG — Square breaks the Rectangle contract
class Rectangle {
  constructor(protected width: number, protected height: number) {}
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  area() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(w: number) {
    this.width = w;
    this.height = w; // Forces equal sides — BREAKS expected Rectangle behavior
  }
  setHeight(h: number) {
    this.width = h;
    this.height = h;
  }
}

// This function expects Rectangle behavior
function stretchWidth(rect: Rectangle) {
  rect.setWidth(10);
  rect.setHeight(5);
  console.log(rect.area()); // Expected: 50. With Square: 25. LSP violated!
}

// ✅ RIGHT — Use a common interface instead of forcing inheritance
interface Shape {
  area(): number;
  perimeter(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  area() { return this.width * this.height; }
  perimeter() { return 2 * (this.width + this.height); }
}

class Square implements Shape {
  constructor(private side: number) {}
  area() { return this.side * this.side; }
  perimeter() { return 4 * this.side; }
}
```

Practical React Native example: every custom navigator should behave like a standard navigator. A `ModalNavigator` that silently ignores `navigation.goBack()` violates LSP.

### I — Interface Segregation Principle

Don't force a class to implement methods it doesn't need. **Split large interfaces into smaller, focused ones**.

```typescript
// ❌ WRONG — one giant interface forces all implementors to handle everything
interface DataSource {
  fetchUsers(): Promise<User[]>;
  saveUser(user: User): Promise<void>;
  deleteUser(id: string): Promise<void>;
  fetchAnalytics(): Promise<Analytics[]>;
  generateReport(): Promise<Report>;
  sendEmail(to: string, body: string): Promise<void>;
}

// ✅ RIGHT — small, focused interfaces
interface UserReader {
  fetchUsers(): Promise<User[]>;
  fetchUserById(id: string): Promise<User | null>;
}

interface UserWriter {
  saveUser(user: User): Promise<void>;
  deleteUser(id: string): Promise<void>;
}

interface AnalyticsProvider {
  fetchAnalytics(): Promise<Analytics[]>;
  generateReport(): Promise<Report>;
}

// Read-only cache only needs to implement what it can do
class UserCache implements UserReader {
  fetchUsers(): Promise<User[]> { return this.cache.getAll(); }
  fetchUserById(id: string): Promise<User | null> { return this.cache.get(id); }
}

// Full repository implements both reading and writing
class UserApiRepository implements UserReader, UserWriter {
  fetchUsers(): Promise<User[]> { /* API call */ }
  fetchUserById(id: string): Promise<User | null> { /* API call */ }
  saveUser(user: User): Promise<void> { /* API call */ }
  deleteUser(id: string): Promise<void> { /* API call */ }
}
```

### D — Dependency Inversion Principle

High-level code should depend on **abstractions**, not concrete implementations.

```typescript
// ❌ WRONG — UserService is tightly coupled to a specific implementation
class UserService {
  private api = new AxiosApiClient();    // Can never swap this
  private storage = new AsyncStorage();  // Can never swap this
  private logger = new ConsoleLogger();  // Can never swap this

  async loadUser(id: string): Promise<User> {
    this.logger.log(`Loading user ${id}`);
    return this.api.get(`/users/${id}`);
  }
}

// ✅ RIGHT — UserService depends on interfaces it receives
interface ApiClient {
  get<T>(path: string): Promise<T>;
  post<T>(path: string, body: object): Promise<T>;
}

interface Storage {
  getItem(key: string): Promise<string | null>;
  setItem(key: string, value: string): Promise<void>;
}

interface Logger {
  log(message: string): void;
  error(message: string, error?: Error): void;
}

// UserService depends only on interfaces — never on concrete classes
class UserService {
  constructor(
    private api: ApiClient,
    private storage: Storage,
    private logger: Logger,
  ) {}

  async loadUser(id: string): Promise<User> {
    this.logger.log(`Loading user ${id}`);
    const cached = await this.storage.getItem(`user_${id}`);
    if (cached) return JSON.parse(cached);

    const user = await this.api.get<User>(`/users/${id}`);
    await this.storage.setItem(`user_${id}`, JSON.stringify(user));
    return user;
  }
}

// In production
const userService = new UserService(
  new AxiosApiClient(),
  new MMKVStorage(),
  new SentryLogger(),
);

// In tests — swap with mocks — no real network or storage
const userService = new UserService(
  new MockApiClient({ '/users/1': mockUser }),
  new InMemoryStorage(),
  new NullLogger(),
);
```

This is why dependency injection containers exist (React's Context API is a form of DI).

---

## 2.4 Common Design Patterns

Named, reusable solutions to problems that appear repeatedly.

### Repository Pattern

Abstracts data access. Callers don't know or care whether data came from the network, a local database, or a cache.

```typescript
interface UserRepository {
  getUser(id: string): Promise<User>;
  updateUser(id: string, data: Partial<User>): Promise<User>;
}

// Production implementation — checks cache, then network
class UserRepositoryImpl implements UserRepository {
  private cache = new Map<string, { user: User; timestamp: number }>();
  private CACHE_TTL = 5 * 60 * 1000; // 5 minutes

  async getUser(id: string): Promise<User> {
    const cached = this.cache.get(id);
    if (cached && Date.now() - cached.timestamp < this.CACHE_TTL) {
      return cached.user;
    }

    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    this.cache.set(id, { user, timestamp: Date.now() });
    return user;
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PATCH',
      body: JSON.stringify(data),
    });
    const user = await response.json();
    this.cache.set(id, { user, timestamp: Date.now() }); // Update cache
    return user;
  }
}

// Your components never need to change when you swap the implementation
const useUser = (id: string) => {
  const [user, setUser] = useState<User | null>(null);
  useEffect(() => {
    userRepository.getUser(id).then(setUser); // Works with any UserRepository
  }, [id]);
  return user;
};
```

### Observer Pattern

One thing changes → others are notified automatically. This is the foundation of React's state system, Redux, and EventEmitter.

```typescript
// Custom EventEmitter
type Listener<T> = (data: T) => void;

class EventEmitter<Events extends Record<string, unknown>> {
  private listeners = new Map<keyof Events, Set<Listener<unknown>>>();

  on<K extends keyof Events>(event: K, listener: Listener<Events[K]>): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(listener as Listener<unknown>);

    // Return unsubscribe function
    return () => this.listeners.get(event)?.delete(listener as Listener<unknown>);
  }

  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    this.listeners.get(event)?.forEach(listener => listener(data));
  }
}

// Usage — cart state management
type CartEvents = {
  itemAdded: { item: CartItem };
  itemRemoved: { itemId: string };
  cleared: void;
};

const cartEmitter = new EventEmitter<CartEvents>();

// Subscribe (in a component)
useEffect(() => {
  const unsubscribe = cartEmitter.on('itemAdded', ({ item }) => {
    showToast(`${item.name} added to cart`);
  });
  return unsubscribe; // Clean up on unmount
}, []);

// Emit from anywhere
cartEmitter.emit('itemAdded', { item: selectedProduct });
```

This pattern is how React Query's cache invalidation works — multiple hooks observing the same query key all re-render when the data changes.

### Factory Pattern

Creates objects without exposing creation logic to the caller.

```typescript
type NotificationType = 'push' | 'email' | 'sms' | 'in_app';

// Factory — knows how to create each type
class NotificationFactory {
  static create(type: NotificationType): NotificationChannel {
    switch (type) {
      case 'push':   return new PushNotificationChannel();
      case 'email':  return new EmailNotificationChannel();
      case 'sms':    return new SMSNotificationChannel();
      case 'in_app': return new InAppNotificationChannel();
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Caller doesn't need to know the constructor details
const channel = NotificationFactory.create('push');
await channel.send('Your order has shipped!');

// Real React Native example — factory for different API environments
class ApiClientFactory {
  static create(env: 'development' | 'staging' | 'production'): ApiClient {
    const baseURLs = {
      development: 'http://localhost:3000',
      staging: 'https://staging-api.myapp.com',
      production: 'https://api.myapp.com',
    };
    return new ApiClient(baseURLs[env]);
  }
}
```

### Singleton Pattern

Ensures only **one instance** of a class exists in the entire application.

```typescript
class AppDatabase {
  private static instance: AppDatabase | null = null;
  private db: SQLiteDatabase;

  // Private constructor — prevents direct instantiation
  private constructor() {
    this.db = SQLite.openDatabase('app.db');
  }

  // The only way to get the database instance
  static getInstance(): AppDatabase {
    if (!AppDatabase.instance) {
      AppDatabase.instance = new AppDatabase();
    }
    return AppDatabase.instance;
  }

  async query<T>(sql: string, params: unknown[] = []): Promise<T[]> {
    return this.db.executeSql(sql, params);
  }
}

// Always the same instance — no multiple connections opened
const db1 = AppDatabase.getInstance();
const db2 = AppDatabase.getInstance();
console.log(db1 === db2); // true

// Note: In React Native, module-level singletons are common
// The module system guarantees a module is only evaluated once
export const userRepository = new UserRepositoryImpl(); // Singleton by module
```

### Builder Pattern

Constructs complex objects **step by step**. Useful when an object has many optional fields.

```typescript
class HttpRequest {
  private method: string = 'GET';
  private url: string = '';
  private headers: Record<string, string> = {};
  private body?: string;
  private timeout: number = 30000;

  private constructor() {}

  static builder(): HttpRequest {
    return new HttpRequest();
  }

  withMethod(method: string): HttpRequest {
    this.method = method;
    return this; // Return this for chaining
  }

  withUrl(url: string): HttpRequest {
    this.url = url;
    return this;
  }

  withHeader(key: string, value: string): HttpRequest {
    this.headers[key] = value;
    return this;
  }

  withAuth(token: string): HttpRequest {
    this.headers['Authorization'] = `Bearer ${token}`;
    return this;
  }

  withBody(body: object): HttpRequest {
    this.body = JSON.stringify(body);
    this.headers['Content-Type'] = 'application/json';
    return this;
  }

  withTimeout(ms: number): HttpRequest {
    this.timeout = ms;
    return this;
  }

  async execute<T>(): Promise<T> {
    const response = await fetch(this.url, {
      method: this.method,
      headers: this.headers,
      body: this.body,
    });
    return response.json();
  }
}

// Clean, readable, step-by-step construction
const user = await HttpRequest
  .builder()
  .withMethod('POST')
  .withUrl('https://api.myapp.com/users')
  .withAuth(authToken)
  .withBody({ name: 'Alice', email: 'alice@example.com' })
  .withTimeout(10000)
  .execute<User>();
```

### Strategy Pattern

Swaps algorithms (strategies) at runtime without changing the calling code.

```typescript
// Each algorithm implements the same interface
interface SortStrategy<T> {
  sort(data: T[], compareFn: (a: T, b: T) => number): T[];
}

class QuickSort<T> implements SortStrategy<T> {
  sort(data: T[], compareFn: (a: T, b: T) => number): T[] {
    return [...data].sort(compareFn); // JS built-in uses quicksort
  }
}

class BubbleSort<T> implements SortStrategy<T> {
  sort(data: T[], compareFn: (a: T, b: T) => number): T[] {
    const arr = [...data];
    for (let i = 0; i < arr.length; i++) {
      for (let j = 0; j < arr.length - i - 1; j++) {
        if (compareFn(arr[j], arr[j + 1]) > 0) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    return arr;
  }
}

// Caller can swap the strategy at runtime
class FeedSorter {
  constructor(private strategy: SortStrategy<Post>) {}

  setStrategy(strategy: SortStrategy<Post>) {
    this.strategy = strategy;
  }

  sortPosts(posts: Post[]): Post[] {
    return this.strategy.sort(posts, (a, b) => b.likes - a.likes);
  }
}

// React Native feed that lets user toggle sort mode
const sorter = new FeedSorter(new QuickSort());
// User switches to "Trending" mode:
sorter.setStrategy(new TrendingScoreSort());
```

### Adapter Pattern

Makes **incompatible interfaces** work together. Wraps one interface so it looks like another.

```typescript
// Third-party payment library has a different interface than what your app expects
interface ThirdPartyPaypalSDK {
  makePayment(cents: number, currency: string): Promise<{ txId: string; status: string }>;
}

// Your app uses this interface everywhere
interface PaymentProvider {
  charge(amount: number, currency: string): Promise<PaymentResult>;
}

// Adapter — wraps the third-party SDK to match your interface
class PayPalAdapter implements PaymentProvider {
  constructor(private sdk: ThirdPartyPaypalSDK) {}

  async charge(amount: number, currency: string): Promise<PaymentResult> {
    // Translate: dollars → cents (your app uses dollars, SDK uses cents)
    const result = await this.sdk.makePayment(amount * 100, currency);

    // Translate: SDK response → your PaymentResult format
    return {
      success: result.status === 'COMPLETED',
      transactionId: result.txId,
    };
  }
}

// Now PayPal works anywhere a PaymentProvider is expected
const paypal = new PayPalAdapter(new ThirdPartyPaypalSDK());
const checkout = new CheckoutService(paypal);
```

---

## 2.5 Composition Over Inheritance

In JavaScript/TypeScript (and React), **composition** — assembling objects from smaller, focused pieces — is almost always preferred over deep inheritance chains.

React itself is built on this idea. You don't extend components; you compose them.

```typescript
// ❌ Deep inheritance chain — fragile, hard to follow
class Component extends BaseComponent
  extends WithAuth extends WithLogging
    extends WithAnalytics { }

// ✅ Composition with hooks — each concern is independent and reusable
const ProductScreen = () => {
  const { user, isAuthenticated } = useAuth();         // Auth concern
  const { track } = useAnalytics();                    // Analytics concern
  const { products, loading, error } = useProducts();  // Data concern
  const { addToCart } = useCart();                     // Cart concern

  if (!isAuthenticated) return <LoginPrompt />;
  if (loading) return <Skeleton />;
  if (error) return <ErrorView error={error} />;

  return <ProductList products={products} onAddToCart={addToCart} />;
};

// Each hook is independently testable and reusable
```

### Mixin Pattern (when you need multi-source behavior)

```typescript
// Mixins in TypeScript — compose behavior from multiple sources
type Constructor<T = {}> = new (...args: unknown[]) => T;

const Serializable = <TBase extends Constructor>(Base: TBase) =>
  class extends Base {
    toJSON(): string { return JSON.stringify(this); }
    static fromJSON<T>(json: string): T { return JSON.parse(json); }
  };

const Validatable = <TBase extends Constructor>(Base: TBase) =>
  class extends Base {
    validate(): boolean { return true; } // Override per class
    isValid(): boolean { return this.validate(); }
  };

class BaseModel {}

// Compose both behaviors onto a class
class UserModel extends Serializable(Validatable(BaseModel)) {
  constructor(public name: string, public email: string) {
    super();
  }

  validate(): boolean {
    return this.name.length > 0 && this.email.includes('@');
  }
}

const user = new UserModel('Alice', 'alice@example.com');
user.isValid();  // true
user.toJSON();   // '{"name":"Alice","email":"alice@example.com"}'
```

---

## Summary Table

| Concept | One-line summary |
|---------|----------------|
| Abstraction | Hide complexity. Expose only what callers need. |
| Encapsulation | Keep state private. Control access through methods. |
| Inheritance | `extends` — IS-A relationships only. Don't overuse. |
| Polymorphism | Same interface, different implementations. Swap at runtime. |
| Interface | Defines what a class can do. No implementation. |
| Abstract class | Partial implementation. Forces subclasses to fill in gaps. |
| SRP | One class, one job, one reason to change. |
| OCP | Extend by adding code, not by editing existing code. |
| LSP | Subclasses must honor the parent's contract. |
| ISP | Small, focused interfaces over one fat interface. |
| DIP | Depend on abstractions. Inject concrete implementations. |
| Repository | Abstracts data source. Callers don't know network/cache/DB. |
| Observer | Subscribe to changes. Foundation of React state system. |
| Factory | Centralized object creation. Hides constructor complexity. |
| Singleton | One instance. Use for shared resources (DB, config, logger). |
| Builder | Step-by-step construction of complex objects. |
| Strategy | Swap algorithms at runtime without changing calling code. |
| Adapter | Make an incompatible interface fit your expected interface. |
| Composition | Combine small, focused pieces instead of deep inheritance. |

---

*Next: Floor 3 — High-Level System Design*
