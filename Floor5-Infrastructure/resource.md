# Floor 5 — Infrastructure and DevOps

> **Prerequisites:** Floors 1–4
> **Focus:** How code gets from a developer's machine to production, and how it keeps running reliably at scale.

---

## 5.1 Containers — Docker

A **container** packages your application and all its dependencies into a single, portable unit that runs the same everywhere: your laptop, staging, production.

### The Problem Docker Solves

```
Developer: "Works on my machine!"
QA:        "Broken on the test server."
DevOps:    "Works in staging, fails in production."

Root cause: different Node.js versions, missing env variables, different OS libraries.

Docker solution: "Works in my container" = works everywhere the container runs.
```

### Dockerfile — Building an Image

```dockerfile
# Start from an official Node.js base image
FROM node:20-alpine

# Set working directory inside the container
WORKDIR /app

# Copy dependency files first (for layer caching)
COPY package.json package-lock.json ./

# Install dependencies
RUN npm ci --only=production

# Copy the rest of the source code
COPY . .

# Build the TypeScript project
RUN npm run build

# Tell Docker which port the app uses
EXPOSE 3000

# Command to run when the container starts
CMD ["node", "dist/server.js"]
```

### Common Docker Commands

```bash
# Build an image from the Dockerfile in the current directory
docker build -t myapp:1.0.0 .

# Run a container from the image
docker run -p 3000:3000 --name myapp-container myapp:1.0.0

# Run with environment variables
docker run \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://... \
  -e JWT_SECRET=super_secret \
  myapp:1.0.0

# View running containers
docker ps

# View logs
docker logs myapp-container

# Stop a container
docker stop myapp-container

# Remove a container
docker rm myapp-container
```

### Docker Compose — Running Multiple Services Locally

When your backend needs a database and a Redis cache, Docker Compose orchestrates all of them together.

```yaml
# docker-compose.yml
version: '3.9'

services:
  api:
    build: .
    ports:
      - '3000:3000'
    environment:
      DATABASE_URL: postgres://user:password@postgres:5432/myapp
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev_secret_only
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./src:/app/src  # Hot reload during development

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U user']
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'

volumes:
  postgres_data:
```

```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose up --build
```

This is the local backend setup for a React Native developer. When you `npx react-native run-android` alongside `docker-compose up`, your app has a full backend stack running locally.

### Image Layers and Caching

Docker builds images in layers. Each instruction in the Dockerfile is a layer. Layers are cached — if nothing changed, Docker reuses the cached layer.

```dockerfile
# ✅ GOOD — dependency layer cached separately from source code
COPY package.json package-lock.json ./  # Layer 1 — only re-runs if deps change
RUN npm ci                               # Layer 2 — only re-runs if deps change
COPY . .                                 # Layer 3 — re-runs when source changes
RUN npm run build                        # Layer 4 — re-runs when source changes

# ❌ BAD — COPY . . first invalidates all subsequent layers on every change
COPY . .
RUN npm ci      # Re-installs ALL packages on every source change
RUN npm run build
```

---

## 5.2 Orchestration — Kubernetes

When you have many containers to manage across many machines, Kubernetes (K8s) automates everything.

### Core Concepts

**Pod** — the smallest unit in Kubernetes. Usually wraps one container.

**Deployment** — declares how many pods to run and how to update them.

**Service** — a stable network endpoint that routes traffic to healthy pods. Pods can be replaced; the Service IP stays constant.

**Ingress** — routes external HTTP traffic to the correct Service.

**ConfigMap / Secret** — store configuration and secrets separately from the container image.

**Namespace** — logical grouping. You can have `development`, `staging`, `production` namespaces in the same cluster.

### A Kubernetes Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  namespace: production
spec:
  replicas: 3            # Run 3 copies of the pod
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: myapp/api:1.2.0
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: api-secrets        # From a Kubernetes Secret
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                configMapKeyRef:
                  name: api-config         # From a ConfigMap
                  key: redis-url
          resources:
            requests:
              memory: '256Mi'
              cpu: '250m'
            limits:
              memory: '512Mi'
              cpu: '500m'
          readinessProbe:                  # Only send traffic when ready
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:                   # Restart if unhealthy
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: production
spec:
  selector:
    app: api             # Routes to pods with label app=api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - api.myapp.com
      secretName: api-tls-secret
  rules:
    - host: api.myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
```

### What Kubernetes Gives You

| Feature | What happens |
|---------|-------------|
| **Self-healing** | Pod crashes → Kubernetes automatically restarts it |
| **Scaling** | `kubectl scale deployment api --replicas=10` → 10 instances immediately |
| **Rolling updates** | Deploy new version → pods replaced one at a time → zero downtime |
| **Rollback** | `kubectl rollout undo deployment/api` → instantly revert to previous version |
| **Service discovery** | Containers find each other by name: `http://api-service` |
| **Load balancing** | Service automatically balances across all healthy pods |
| **Auto-scaling** | HPA watches CPU/memory → adds/removes pods automatically |

### Common kubectl Commands

```bash
# Get status of pods
kubectl get pods -n production

# Get detailed info about a pod
kubectl describe pod api-deployment-7d89c7b5-xkl2p -n production

# Stream pod logs
kubectl logs -f api-deployment-7d89c7b5-xkl2p -n production

# Deploy new image version
kubectl set image deployment/api api=myapp/api:1.3.0 -n production

# Watch rollout progress
kubectl rollout status deployment/api -n production

# Roll back
kubectl rollout undo deployment/api -n production

# Scale manually
kubectl scale deployment api --replicas=5 -n production

# Open a shell inside a running pod (debugging)
kubectl exec -it api-deployment-7d89c7b5-xkl2p -n production -- /bin/sh
```

---

## 5.3 CI/CD — Continuous Integration and Continuous Delivery

Automating the path from code commit to production deployment.

### Continuous Integration (CI)

Every code push triggers an automated pipeline:
1. Run linter (ESLint, TypeScript check)
2. Run unit tests
3. Run integration tests
4. Build the Docker image
5. If all pass → merge is safe

### Continuous Delivery (CD)

After CI passes:
1. Push Docker image to registry (Docker Hub, ECR, GCR)
2. Deploy to staging automatically
3. Run end-to-end tests on staging
4. Manual approval gate (optional)
5. Deploy to production

### GitHub Actions — CI/CD for Your API

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  ECR_REPOSITORY: myapp-api
  EKS_CLUSTER_NAME: myapp-production

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: TypeScript check
        run: npx tsc --noEmit

      - name: Lint
        run: npm run lint

      - name: Unit tests
        run: npm run test:unit

      - name: Integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/testdb

  build-and-deploy:
    name: Build and Deploy
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Only deploy on main branch

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
                     $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

      - name: Deploy to EKS
        run: |
          aws eks update-kubeconfig --name $EKS_CLUSTER_NAME
          kubectl set image deployment/api \
            api=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
            -n production
          kubectl rollout status deployment/api -n production
```

### GitHub Actions — CI/CD for React Native

```yaml
# .github/workflows/rn-ci.yml
name: React Native CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx tsc --noEmit
      - run: npm run lint
      - run: npm run test -- --coverage --watchAll=false

  android:
    name: Android Build
    runs-on: ubuntu-latest
    needs: quality
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - run: npm ci
      - name: Build Android APK
        run: |
          cd android
          ./gradlew assembleRelease \
            -PMYAPP_RELEASE_KEY_ALIAS=${{ secrets.ANDROID_KEY_ALIAS }} \
            -PMYAPP_RELEASE_KEY_PASSWORD=${{ secrets.ANDROID_KEY_PASSWORD }} \
            -PMYAPP_RELEASE_STORE_PASSWORD=${{ secrets.ANDROID_STORE_PASSWORD }}
      - name: Upload APK artifact
        uses: actions/upload-artifact@v4
        with:
          name: android-release
          path: android/app/build/outputs/apk/release/app-release.apk

  ios:
    name: iOS Build
    runs-on: macos-latest
    needs: quality
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: cd ios && pod install
      - name: Build iOS
        run: |
          xcodebuild -workspace ios/MyApp.xcworkspace \
            -scheme MyApp \
            -configuration Release \
            -archivePath ios/MyApp.xcarchive \
            archive
```

### Fastlane — Mobile-Specific Automation

Fastlane automates mobile-specific tasks that GitHub Actions doesn't know about.

```ruby
# fastlane/Fastfile

platform :ios do
  desc "Run tests"
  lane :test do
    run_tests(scheme: "MyApp")
  end

  desc "Build and upload to TestFlight"
  lane :beta do
    increment_build_number                  # Auto-increment build number
    build_app(scheme: "MyApp")
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
    slack(message: "iOS beta uploaded to TestFlight ✅")
  end

  desc "Release to App Store"
  lane :release do
    ensure_git_status_clean                  # No uncommitted changes
    increment_version_number(bump_type: "minor")
    build_app(scheme: "MyApp")
    upload_to_app_store(
      force: true,
      submit_for_review: true,
      automatic_release: false             # Manual review approval
    )
  end
end

platform :android do
  desc "Build and upload to Play Console internal track"
  lane :beta do
    gradle(
      task: 'bundle',
      build_type: 'Release',
    )
    upload_to_play_store(
      track: 'internal',
      aab: 'android/app/build/outputs/bundle/release/app-release.aab'
    )
  end
end
```

---

## 5.4 OTA Updates — CodePush

Over-the-air (OTA) updates let you push JavaScript bundle changes to users **without going through the App Store or Play Store**. Only JavaScript and assets can be updated this way — native code always requires a store update.

```javascript
// App.tsx — integrate CodePush
import CodePush from 'react-native-code-push';

const codePushOptions = {
  checkFrequency: CodePush.CheckFrequency.ON_APP_RESUME,
  updateDialog: {
    appendReleaseDescription: true,
    title: 'Update available',
  },
  installMode: CodePush.InstallMode.IMMEDIATE,
};

const App = () => {
  return (
    <NavigationContainer>
      <AppNavigator />
    </NavigationContainer>
  );
};

export default CodePush(codePushOptions)(App);
```

```bash
# Deploy a JS bundle update via CodePush
appcenter codepush release-react \
  -a myorg/MyApp-iOS \
  -d Production \
  --target-binary-version "~1.2.0" \
  --description "Fix crash on profile screen"
```

**When to use OTA:** Bug fixes, UI tweaks, copy changes, A/B tests.
**When you MUST use a store update:** New native modules, permissions, binary dependencies.

---

## 5.5 Horizontal vs Vertical Scaling — In Depth

### Vertical Scaling (Scale Up)

```
Before: 1 × t3.medium  (2 vCPU, 4GB RAM)  — $0.041/hour
After:  1 × c5.2xlarge (8 vCPU, 16GB RAM) — $0.34/hour
```

**Pros:** Zero code changes, no distribution complexity.
**Cons:**
- Hard ceiling — there's a biggest machine
- Single point of failure — if it goes down, everything stops
- Expensive at the high end

### Horizontal Scaling (Scale Out)

```
Before: 1 × t3.medium
After:  10 × t3.medium  — distribute load, survive individual failures
```

**Requirements for horizontal scaling:**

1. **Stateless services** — no session state in server memory. Sessions in Redis.

```javascript
// ❌ Stateful — only works on one server
const sessions = new Map(); // In-memory — server 2 doesn't know about this

app.post('/login', (req, res) => {
  const sessionId = uuid();
  sessions.set(sessionId, { userId: user.id }); // Stored in memory
  res.cookie('sessionId', sessionId);
});

// ✅ Stateless — any server can handle any request
import redis from './redis';

app.post('/login', async (req, res) => {
  const sessionId = uuid();
  await redis.setEx(`session:${sessionId}`, 86400, JSON.stringify({ userId: user.id }));
  res.cookie('sessionId', sessionId);
});
```

2. **Shared file storage** — use S3/cloud storage, not local disk. A file uploaded to server 1 isn't visible to server 2.

3. **Health checks** — the load balancer needs to know which servers are alive.

```javascript
// Health check endpoint — load balancer probes this every few seconds
app.get('/health', async (req, res) => {
  try {
    await db.query('SELECT 1'); // Check DB connectivity
    await redis.ping();          // Check cache connectivity
    res.json({ status: 'healthy', uptime: process.uptime() });
  } catch (error) {
    res.status(503).json({ status: 'unhealthy', error: error.message });
  }
});
```

---

## 5.6 Cloud Providers — AWS, GCP, Azure

The major cloud providers offer the same core building blocks, with different names.

| Service Category | AWS | GCP | Azure |
|-----------------|-----|-----|-------|
| Virtual Machines | EC2 | Compute Engine | Virtual Machines |
| Container Service | ECS / EKS | GKE | AKS |
| Serverless Functions | Lambda | Cloud Functions | Azure Functions |
| Object Storage | S3 | Cloud Storage | Blob Storage |
| Managed SQL DB | RDS | Cloud SQL | Azure SQL |
| Managed NoSQL | DynamoDB | Firestore | Cosmos DB |
| In-Memory Cache | ElastiCache (Redis) | Memorystore | Azure Cache for Redis |
| Message Queue | SQS | Pub/Sub | Service Bus |
| CDN | CloudFront | Cloud CDN | Azure CDN |
| DNS | Route 53 | Cloud DNS | Azure DNS |
| Monitoring | CloudWatch | Cloud Monitoring | Azure Monitor |

### AWS for React Native Developers

The most common AWS services you'll encounter:

**S3 (Simple Storage Service)** — Store user-uploaded images, app bundles, static assets.

```javascript
// Uploading a user's profile photo from React Native to S3
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

// Server generates a pre-signed URL (short-lived, direct upload permission)
const generateUploadUrl = async (userId, fileType) => {
  const key = `avatars/${userId}/${Date.now()}.jpg`;
  const command = new PutObjectCommand({
    Bucket: 'myapp-uploads',
    Key: key,
    ContentType: fileType,
  });
  const uploadUrl = await getSignedUrl(s3Client, command, { expiresIn: 300 });
  return { uploadUrl, key };
};

// React Native client uploads directly to S3 (bypasses your server)
const uploadAvatar = async (imageUri) => {
  // 1. Get pre-signed URL from your server
  const { uploadUrl, key } = await apiClient.post('/upload-url', {
    fileType: 'image/jpeg',
  });

  // 2. Upload directly to S3 — no traffic through your server
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': 'image/jpeg' },
    body: await fetch(imageUri).then(r => r.blob()),
  });

  // 3. Tell your server the upload is done
  await apiClient.patch('/users/me', { avatarKey: key });
};
```

**Lambda** — Run code in response to events, without managing servers.

```javascript
// Lambda function — runs only when called, scales to zero
export const handler = async (event) => {
  // Triggered by: API Gateway, S3 events, SQS messages, scheduled cron, etc.

  if (event.source === 'aws.events') {
    // Scheduled cron job — runs daily to send digests
    await sendDailyDigests();
    return;
  }

  // API Gateway proxy event
  const { httpMethod, path, body } = event;
  // handle HTTP request...
};
```

---

## 5.7 Serverless Architecture

Run business logic **without managing servers**. You deploy functions; the cloud handles capacity, scaling, and availability.

```
Traditional server:
  - Always running — you pay even when idle
  - You manage OS updates, security patches
  - You manage scaling

Serverless (Lambda/Cloud Functions):
  - Runs only when triggered — pay per invocation
  - Cloud manages everything
  - Scales to zero (no traffic = no cost) and to thousands (spike traffic)
```

```javascript
// Serverless function — Express-like but no always-on server
// Deployed to Vercel, AWS Lambda, Cloudflare Workers

// Vercel serverless function (api/users/[id].js)
export default async function handler(req, res) {
  const { id } = req.query;

  if (req.method === 'GET') {
    const user = await db.query('SELECT * FROM users WHERE id = $1', [id]);
    if (!user.rows[0]) return res.status(404).json({ error: 'Not found' });
    return res.json(user.rows[0]);
  }

  if (req.method === 'PATCH') {
    const updated = await db.query(
      'UPDATE users SET name = $1 WHERE id = $2 RETURNING *',
      [req.body.name, id]
    );
    return res.json(updated.rows[0]);
  }

  res.setHeader('Allow', ['GET', 'PATCH']);
  res.status(405).end();
}
```

**Serverless is great for:** Event-driven processing, sporadic workloads, simple APIs, scheduled tasks.
**Serverless is not great for:** Long-running processes (max ~15 min), persistent connections (WebSockets), consistent low latency (cold starts).

---

## 5.8 Monitoring and Observability

You can't fix what you can't see. Observability has three pillars: **logs, metrics, traces**.

### Logs — What Happened

```javascript
// Structured logging — machine-readable JSON logs
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
});

app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    logger.info({
      requestId: req.headers['x-request-id'],
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration: Date.now() - start,
      userId: req.user?.id,
    });
  });
  next();
});

// Never log sensitive data
logger.info({ userId: user.id, action: 'login' }); // ✅
logger.info({ user }); // ❌ May log password, token, credit card
```

### Metrics — How the System Is Performing

Key metrics to track:

| Metric | What it measures | Alert threshold |
|--------|-----------------|-----------------|
| Request rate | Requests per second | Unusual spike or drop |
| Error rate | % of requests returning 5xx | > 1% |
| Latency (p95, p99) | 95th/99th percentile response time | p95 > 500ms |
| CPU usage | Server load | > 80% sustained |
| Memory usage | RAM consumption | > 85% |
| DB connection pool | Connections used/available | > 90% |
| Cache hit rate | % served from cache | < 80% |

### Distributed Tracing — Following a Request Across Services

When a request fails in microservices, tracing shows exactly where it broke.

```javascript
// Add trace ID to all requests — passes through every service
import { NodeSDK } from '@opentelemetry/sdk-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: 'https://api.honeycomb.io/v1/traces',
  }),
});
sdk.start();

// All downstream requests automatically carry the trace
// Honeycomb/Datadog/Jaeger shows the full request journey:
// API Gateway → User Service → Database → Cache → Response
```

### React Native Crash Reporting

```javascript
// Sentry — captures JavaScript and native crashes with full stack traces
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: 'https://your-dsn@o123456.ingest.sentry.io/123456',
  environment: __DEV__ ? 'development' : 'production',
  tracesSampleRate: 0.2, // Sample 20% of transactions for performance
});

// Enrich error reports with user context
Sentry.setUser({ id: user.id, email: user.email });

// Manual error capture with context
try {
  await processPayment(order);
} catch (error) {
  Sentry.captureException(error, {
    extra: { orderId: order.id, amount: order.total },
    tags: { feature: 'checkout' },
  });
  showErrorToast('Payment failed. Please try again.');
}
```

---

## 5.9 Environment Management

Never hardcode configuration. Separate config from code.

```javascript
// .env — local development (NEVER commit to git)
DATABASE_URL=postgres://user:password@localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379
JWT_SECRET=dev_only_secret_not_for_production
API_BASE_URL=http://localhost:3000

// .env.staging
DATABASE_URL=postgres://...
API_BASE_URL=https://staging-api.myapp.com

// .env.production
DATABASE_URL=postgres://...
API_BASE_URL=https://api.myapp.com
```

```javascript
// Validate env vars at startup — fail fast if something is missing
import { z } from 'zod';

const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(['development', 'staging', 'production']).default('development'),
});

const env = EnvSchema.parse(process.env);
// If any required var is missing → throws at startup, not 30 minutes into production

export default env;
```

```javascript
// React Native — environment config per build variant
// react-native-config library reads from .env files

import Config from 'react-native-config';

const apiClient = new ApiClient(Config.API_BASE_URL);

// android/app/build.gradle — use different .env per build type
android {
  buildTypes {
    debug {
      resValue "string", "build_config_package", "com.myapp"
    }
    staging {
      // Loads .env.staging
    }
    release {
      // Loads .env.production
    }
  }
}
```

---

## 5.10 CDN — Content Delivery Network

A CDN caches your static content (images, fonts, JS bundles) on servers geographically distributed around the world.

```
Without CDN:
  User in Tokyo → requests image → server in US-East → 200ms round trip

With CDN:
  User in Tokyo → requests image → CDN node in Tokyo → 5ms round trip
  (image was cached at the Tokyo CDN node from the first request)
```

**What to serve from CDN:**
- Images uploaded by users (via S3 + CloudFront)
- React Native JS bundles for OTA (via CodePush CDN)
- App icons, splash screens, static assets
- Video content

```javascript
// React Native — serve images from CDN, not directly from S3
const getAvatarUrl = (key) => `https://cdn.myapp.com/${key}`;

// With image transformations (Cloudinary or imgix CDN)
// Resize + compress on the fly
const getOptimizedAvatar = (key, width) =>
  `https://res.cloudinary.com/myapp/image/upload/w_${width},f_auto,q_auto/${key}`;

// Load the right size for the screen density
const avatarUrl = getOptimizedAvatar(user.avatarKey, 100 * PixelRatio.get());
```

---

## 5.11 The Full Picture — End to End

When a user taps "Place Order" in your React Native app:

```
[React Native App]
  ↓  HTTPS POST /v1/orders  (JWT Bearer token)
  
[CDN / Edge (CloudFront)]
  ↓  Forwards to origin (non-cached POST)
  
[Load Balancer (AWS ALB)]
  ↓  Round-robin to healthy API pod
  
[API Gateway / Ingress (Kubernetes)]
  ↓  TLS termination, rate limit check
  
[Order Service (Pod in Kubernetes Deployment)]
  ↓  JWT validated by auth middleware
  ↓  Request validated (Zod schema)
  ↓  Business logic: check inventory, calculate total
  
[PostgreSQL (RDS Primary)]
  ↓  BEGIN TRANSACTION
  ↓  INSERT INTO orders ...
  ↓  UPDATE inventory SET stock = stock - 1 ...
  ↓  COMMIT
  
[Redis (ElastiCache)]
  ↓  Invalidate user's order cache
  
[SQS Queue]
  ↓  Publish "order.confirmed" message

[React Native App]
  ←  201 Created { orderId: "ord_xyz" }   ← Response returns here

[Notification Service (consuming SQS)] — runs asynchronously
  ↓  Push notification: "Your order has been confirmed!"

[React Native App]
  ←  Push notification received
```

Every floor played a role. That is system design in production.

---

## Summary

| Concept | Key Takeaway |
|---------|-------------|
| Docker | Package app + deps. Runs the same everywhere. |
| Dockerfile | Recipe to build a Docker image. Layer order matters for cache. |
| Docker Compose | Run multi-container stacks locally (API + DB + Redis). |
| Kubernetes | Orchestrates containers. Self-healing, scaling, rolling deploys. |
| Pod | One container (usually). Smallest K8s unit. |
| Deployment | Declares how many pods and how to update them. |
| Service | Stable network endpoint routing to healthy pods. |
| Ingress | Routes external HTTP traffic to services. TLS termination. |
| CI | Automated test + build on every commit. |
| CD | Automated deploy to staging/production after CI passes. |
| GitHub Actions | YAML-based CI/CD pipelines. Free for public repos. |
| Fastlane | Mobile-specific automation: TestFlight, Play Store uploads. |
| CodePush | OTA updates for RN JS bundle. No store update needed. |
| Vertical scaling | Bigger machine. Simple, hard ceiling. |
| Horizontal scaling | More machines. Stateless required. Unlimited ceiling. |
| Health check | `/health` endpoint. Load balancer routes only to healthy instances. |
| S3 | Object storage. Images, files, bundles. Pre-signed URLs for direct upload. |
| Lambda | Serverless. Pay per invocation. No always-on server. |
| CloudFront | CDN. Cache static assets at edge. Millisecond latency globally. |
| Logs | Structured JSON. What happened. Never log secrets. |
| Metrics | Rate, latency, error rate, CPU, memory. Alert on anomalies. |
| Distributed tracing | Follow a request across services. Find exactly where it broke. |
| Sentry | React Native crash reporting. Stack traces with user context. |
| Env vars | Separate config from code. Validate at startup. Never commit secrets. |

---

*You have now covered all five floors. Review them in order. The end-to-end diagram above is the architecture you understand.*
