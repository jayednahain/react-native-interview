# Metro Bundle Process

## Table of Contents
1. [What is a Bundler?](#what-is-a-bundler)
2. [Metro's Core Responsibilities](#metros-core-responsibilities)
3. [When Does Metro Actually Bundle?](#when-does-metro-actually-bundle)
4. [Fast Refresh (Incremental Bundling)](#fast-refresh-incremental-bundling)
5. [Key Takeaways](#key-takeaways)

---

## What is a Bundler?

When you write a React Native app, your code is **spread across many files**:

- Multiple `.js` files
- `import` / `require()` statements linking them together
- JSX syntax (`<View>`, `<Text>`)
- Modern JS features (ES6+, `async/await`)
- Packages from `node_modules`

**The problem?** iOS and Android devices **don't understand all of that directly.**

### A Bundler's Job is to:

| Step | What it does |
|------|--------------|
| **Transform** | Converts JSX → plain JavaScript (using Babel) |
| **Bundle** | Combines all your files into one (or a few) JS file(s) |
| **Resolve** | Follows all `import`/`require` statements and includes them |
| **Optimize** | Minifies for production, or keeps readable for development |
| **Serve** | Delivers the bundle to your app (over HTTP in dev mode) |

### Simple Analogy 🍳

Think of a bundler like a **restaurant kitchen**:

- Your raw source files = **raw ingredients**
- The bundler = **kitchen that cooks and plates everything**
- The final bundle = **the meal served to your app**

The device only ever sees the final "meal" (the bundle) — it never sees your raw ingredients (source files).

---

## Metro's Core Responsibilities

Metro does **more than just bundle** — it has **3 core responsibilities**:

### 1. 🔄 Transform
Metro **converts your modern code** into something the JS engine can run:
- JSX (`<View />`) → plain JavaScript (`React.createElement(View, ...)`)
- ES6+ syntax → compatible JavaScript (via **Babel**)

### 2. 📦 Bundle (Resolve + Pack)
Metro **follows every `import` / `require`** in your code and:
- Pulls in all your files (`App.js`, `MobileLogin.js`, etc.)
- Pulls in all packages from `node_modules`
- **Packs them all into one single `.js` file** → this is the **bundle**

### 3. 🚀 Serve
Metro **sends the bundle to your app** over HTTP:
```
http://localhost:8081/index.bundle?platform=android
```

---

## When Does Metro Actually Bundle?

This is a key point — **Metro does NOT bundle immediately on `npm start`.**

```
npm start          → Metro starts, just waits 👀
run-android        → App launches, requests the bundle
                   → Metro NOW transforms + bundles 🔥
                   → Sends bundle to app ✅
```

> Metro only bundles **on-demand** — when the app requests it!

### Visual Timeline

```
npm start
  ↓
Metro dev server starts (waiting...)
  ↓
npx react-native run-android
  ↓
Android builds & installs
  ↓
App launches and requests: http://localhost:8081/index.bundle?platform=android
  ↓
🔥 Metro NOW transforms all your .js files (JSX → JS using Babel)
  ↓
Metro sends bundle to app
  ↓
App displays your UI on Android device
```

---

## Fast Refresh (Incremental Bundling)

When you **edit a file**, Metro doesn't re-bundle everything:

```
You save MobileLogin.js
      ↓
Metro detects the change
      ↓
Re-transforms ONLY that file (fast!)
      ↓
Sends a small "delta bundle" to the app
      ↓
UI updates without full reload ✅
```

This is called **incremental bundling** — only changed files are re-transformed, making development fast.

---

## Key Takeaways

1. **A bundler** takes your many source files → transforms them → packs them into one optimized file the device can run.

2. **Metro is React Native's bundler** — it transforms, bundles, and serves your JavaScript code.

3. **Metro starts with `npm start`** but does NOT bundle until the app requests it.

4. **The bundle is sent over HTTP** (`localhost:8081`) from Metro to your app during development.

5. **Fast Refresh** sends only the changed modules (delta bundle), so you don't wait for a full re-bundle on every save.

6. **Other bundlers exist** too:

| Bundler | Used for |
|---------|----------|
| **Metro** | React Native apps |
| **Webpack** | Web apps (React, Vue, etc.) |
| **Vite** | Modern web apps (faster dev server) |
| **Rollup** | Libraries and packages |
| **esbuild** | Fast bundling (newer tool) |

---

**Created from discussion on Metro Bundler concepts and process.**
