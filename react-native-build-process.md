# React Native Metro Bundler - Complete Guide

## Table of Contents
1. [What is Metro Bundler?](#what-is-metro-bundler)
2. [How Metro Works](#how-metro-works)
3. [The Bundle Communication](#the-bundle-communication)
4. [Development vs Production](#development-vs-production)
5. [Common Questions](#common-questions)

---

## What is Metro Bundler?

**Metro** is the **JavaScript bundler** for React Native. It takes all your JavaScript code (and dependencies) and packages them so your app can run on iOS/Android devices.

### Why Do We Need a Bundler?

When you write React Native code, you have:
- Multiple `.js` files
- `import` and `require()` statements
- JSX syntax (e.g., `<View>`, `<Text>`)
- Modern JavaScript features (ES6+, async/await, etc.)
- npm packages from `node_modules`

**iOS and Android don't understand all of that directly.**

### What Metro Does

1. **Transforms** your code (JSX → plain JavaScript using Babel)
2. **Bundles** all files into a single (or a few) JavaScript file(s)
3. **Resolves** all `import`/`require` statements
4. **Optimizes** for development or production
5. **Serves** the bundle to your app over HTTP (in dev mode)

### Metro vs Other Bundlers

| Bundler | Used for |
|---------|----------|
| **Metro** | React Native apps |
| **Webpack** | Web apps (React, Vue, etc.) |
| **Vite** | Modern web apps (faster dev server) |
| **Rollup** | Libraries and packages |
| **esbuild** | Fast bundling (newer tool) |

---

## How Metro Works

### The React Native App Has TWO Parts

#### 1. Native Part (Java/Kotlin for Android, Swift/Objective-C for iOS)
- This is the **container/shell**
- Handles UI rendering, device APIs, sensors, camera, etc.
- Written in native Android/iOS code
- Compiled into the `.apk` (Android) or `.ipa` (iOS)

#### 2. JavaScript Part (Your React Native Code)
- This is **your app logic** (`MobileLogin.js`, `App.js`, etc.)
- Written in JavaScript/JSX
- This is what **Metro bundles**

### How They Work Together
