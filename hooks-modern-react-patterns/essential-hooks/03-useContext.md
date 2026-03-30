# useContext Hook - Complete Guide

---

## 📌 Definition

**useContext** is a React Hook that lets you subscribe to context without introducing nesting. Context provides a way to pass data through the component tree without having to pass props down manually at every level (avoiding "prop drilling").

---

## 🎯 Purpose & What It Does

### Purpose

- **Avoid prop drilling** - Pass data deep into component tree without intermediate props
- **Access context values** - Consume context in functional components easily
- **Share global data** - Share user info, theme, language, authentication state, etc.
- **Reduce boilerplate** - Simpler than passing props through many components

### What It Actually Does

1. Takes a context object (created with React.createContext)
2. Returns the current context value
3. Re-renders when context value changes
4. Must be used inside a provider that supplies the context value

---

## 🔄 Class Component vs Hooks Comparison

### Class Component (Old Way)

```javascript
import React, { createContext } from "react";

const ThemeContext = createContext("light");

// Using context in class component
class Button extends React.Component {
  static contextType = ThemeContext; // Can only use ONE context

  render() {
    const theme = this.context; // Access context via this.context
    return (
      <button style={{ background: theme === "dark" ? "#333" : "#fff" }}>
        Click me
      </button>
    );
  }
}

// For multiple contexts, need Consumer component (verbose)
class Card extends React.Component {
  render() {
    return (
      <ThemeContext.Consumer>
        {(theme) => (
          <LanguageContext.Consumer>
            {(language) => (
              <div style={{ background: theme === "dark" ? "#333" : "#fff" }}>
                {language === "en" ? "Hello" : "Hola"}
              </div>
            )}
          </LanguageContext.Consumer>
        )}
      </ThemeContext.Consumer>
    );
  }
}
```

### Hooks Version (Modern Way)

```javascript
import React, { createContext, useContext } from "react";

const ThemeContext = createContext("light");
const LanguageContext = createContext("en");

// Simple and clean - can use multiple contexts easily
function Button() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme === "dark" ? "#333" : "#fff" }}>
      Click me
    </button>
  );
}

// Easy to use multiple contexts
function Card() {
  const theme = useContext(ThemeContext);
  const language = useContext(LanguageContext);
  return (
    <div style={{ background: theme === "dark" ? "#333" : "#fff" }}>
      {language === "en" ? "Hello" : "Hola"}
    </div>
  );
}
```

### Why We Switched to Hooks

| Aspect                | Class Component                              | useContext Hook           |
| --------------------- | -------------------------------------------- | ------------------------- |
| **Syntax**            | static contextType or Consumer               | Simple useContext call    |
| **Multiple contexts** | Nested Consumer components (pyramid of doom) | Multiple useContext calls |
| **Readability**       | Verbose Consumer syntax                      | Clean and declarative     |
| **Refactoring**       | Hard to add/remove contexts                  | Easy to add/remove        |

---

## 🔄 How Context Works

```javascript
// Step 1: Create context
const MyContext = React.createContext(defaultValue);

// Step 2: Create provider component
function MyProvider({ children }) {
  const [value, setValue] = useState(defaultValue);
  return (
    <MyContext.Provider value={{ value, setValue }}>
      {children}
    </MyContext.Provider>
  );
}

// Step 3: Use context in components
function MyComponent() {
  const { value, setValue } = useContext(MyContext);
  return <div>{value}</div>;
}

// Step 4: Wrap app with provider
<MyProvider>
  <MyComponent />
</MyProvider>;
```

---

## 💡 Real-World Use Cases

1. **Theme (Dark/Light Mode)** - Share theme preference across app
2. **Authentication** - Share user info after login
3. **Language/i18n** - Share selected language globally
4. **User Preferences** - Font size, notifications, etc.
5. **API Client** - Share configured API client
6. **Modal/Dialog** - Share modal state and functions
7. **Notifications** - Share toast/notification system

---

## 📝 Hook Examples

### Example 1: Simple Theme Context

```javascript
import React, { createContext, useState, useContext } from "react";
import { View, Text, Button, StyleSheet } from "react-native";

// Step 1: Create context
const ThemeContext = createContext();

// Step 2: Create custom hook to use context easily
function useTheme() {
  return useContext(ThemeContext);
}

// Step 3: Create provider component
function ThemeProvider({ children }) {
  const [isDark, setIsDark] = useState(false);

  const toggleTheme = () => setIsDark(!isDark);

  const theme = {
    isDark,
    colors: {
      background: isDark ? "#333" : "#fff",
      text: isDark ? "#fff" : "#000",
      primary: "#007AFF",
    },
    toggleTheme,
  };

  return (
    <ThemeContext.Provider value={theme}>{children}</ThemeContext.Provider>
  );
}

// Step 4: Use context in components
function ThemedButton() {
  const { colors, toggleTheme, isDark } = useTheme();

  return (
    <Button
      title={`Current Theme: ${isDark ? "Dark" : "Light"}`}
      onPress={toggleTheme}
      color={colors.primary}
    />
  );
}

function ThemedCard() {
  const { colors } = useTheme();

  return (
    <View style={[styles.card, { backgroundColor: colors.background }]}>
      <Text style={[styles.text, { color: colors.text }]}>
        This card adapts to theme
      </Text>
    </View>
  );
}

// Step 5: Wrap app with provider
function App() {
  return (
    <ThemeProvider>
      <View style={styles.container}>
        <ThemedButton />
        <ThemedCard />
      </View>
    </ThemeProvider>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
  card: { padding: 20, borderRadius: 10 },
  text: { fontSize: 16 },
});

export default App;
```

---

### Example 2: Authentication Context

```javascript
import React, { createContext, useState, useContext } from "react";
import { View, Text, Button, TextInput } from "react-native";

const AuthContext = createContext();

function useAuth() {
  return useContext(AuthContext);
}

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const login = async (email, password) => {
    setLoading(true);
    setError(null);
    try {
      // Simulating: POST https://xyzzy.com/api/login
      const response = await fetch(
        "https://jsonplaceholder.typicode.com/users/1",
      );
      const userData = await response.json();

      // In real app, verify email/password against backend
      setUser({
        id: userData.id,
        email: email,
        name: userData.name,
      });
    } catch (err) {
      setError("Login failed");
    } finally {
      setLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
  };

  const value = { user, loading, error, login, logout, isLoggedIn: !!user };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// Use in components
function LoginScreen() {
  const { login, loading, error } = useAuth();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleLogin = () => {
    login(email, password);
  };

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        style={{ borderBottomWidth: 1, marginBottom: 10, padding: 10 }}
      />
      <TextInput
        placeholder="Password"
        secureTextEntry
        value={password}
        onChangeText={setPassword}
        style={{ borderBottomWidth: 1, marginBottom: 20, padding: 10 }}
      />
      <Button
        title={loading ? "Logging in..." : "Login"}
        onPress={handleLogin}
        disabled={loading}
      />
      {error && <Text style={{ color: "red", marginTop: 10 }}>{error}</Text>}
    </View>
  );
}

function ProfileScreen() {
  const { user, logout } = useAuth();

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 20, marginBottom: 10 }}>Profile</Text>
      <Text>Name: {user?.name}</Text>
      <Text>Email: {user?.email}</Text>
      <Button title="Logout" onPress={logout} color="red" />
    </View>
  );
}

// App with auth provider
function App() {
  const { isLoggedIn } = useAuth();

  return (
    <View style={{ flex: 1 }}>
      {isLoggedIn ? <ProfileScreen /> : <LoginScreen />}
    </View>
  );
}

export default function AppWrapper() {
  return (
    <AuthProvider>
      <App />
    </AuthProvider>
  );
}
```

---

### Example 3: Multiple Contexts (Theme + Language)

```javascript
import React, { createContext, useState, useContext } from "react";
import { View, Text, Button } from "react-native";

// Create multiple contexts
const ThemeContext = createContext();
const LanguageContext = createContext();

// Custom hooks for easy access
function useTheme() {
  return useContext(ThemeContext);
}

function useLanguage() {
  return useContext(LanguageContext);
}

// Combined provider component
function AppProvider({ children }) {
  const [isDark, setIsDark] = useState(false);
  const [language, setLanguage] = useState("en");

  const themeValue = {
    isDark,
    toggleTheme: () => setIsDark(!isDark),
  };

  const languageValue = {
    language,
    setLanguage,
    t: (key) => {
      const translations = {
        en: { greeting: "Hello", goodbye: "Goodbye" },
        es: { greeting: "Hola", goodbye: "Adiós" },
      };
      return translations[language][key];
    },
  };

  return (
    <ThemeContext.Provider value={themeValue}>
      <LanguageContext.Provider value={languageValue}>
        {children}
      </LanguageContext.Provider>
    </ThemeContext.Provider>
  );
}

// Use multiple contexts in a component
function GreetingCard() {
  const { isDark } = useTheme();
  const { t, setLanguage, language } = useLanguage();

  return (
    <View
      style={{
        padding: 20,
        backgroundColor: isDark ? "#333" : "#fff",
      }}
    >
      <Text style={{ color: isDark ? "#fff" : "#000" }}>{t("greeting")}</Text>
      <Button
        title={language === "en" ? "Switch to Spanish" : "Switch to English"}
        onPress={() => setLanguage(language === "en" ? "es" : "en")}
      />
    </View>
  );
}

export default function App() {
  return (
    <AppProvider>
      <GreetingCard />
    </AppProvider>
  );
}
```

---

### Example 4: Notification/Toast Context

```javascript
import React, { createContext, useState, useCallback, useContext } from "react";
import { View, Text, Button } from "react-native";

const NotificationContext = createContext();

function useNotification() {
  return useContext(NotificationContext);
}

function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);

  const showNotification = useCallback((message, type = "info") => {
    const id = Date.now();
    setNotifications((prev) => [...prev, { id, message, type }]);

    // Auto remove after 3 seconds
    setTimeout(() => {
      setNotifications((prev) => prev.filter((n) => n.id !== id));
    }, 3000);
  }, []);

  const removeNotification = useCallback((id) => {
    setNotifications((prev) => prev.filter((n) => n.id !== id));
  }, []);

  return (
    <NotificationContext.Provider
      value={{ showNotification, removeNotification }}
    >
      {children}
      <View style={{ position: "absolute", top: 20, right: 20, width: 300 }}>
        {notifications.map((notif) => (
          <View
            key={notif.id}
            style={{
              backgroundColor: notif.type === "error" ? "red" : "green",
              padding: 10,
              marginBottom: 10,
              borderRadius: 5,
            }}
          >
            <Text style={{ color: "white" }}>{notif.message}</Text>
          </View>
        ))}
      </View>
    </NotificationContext.Provider>
  );
}

// Use in any component
function UserActions() {
  const { showNotification } = useNotification();

  const handleSave = async () => {
    try {
      // Simulating API call to https://xyzzy.com/api/save
      const response = await fetch(
        "https://jsonplaceholder.typicode.com/posts",
        {
          method: "POST",
          body: JSON.stringify({ title: "New Post" }),
        },
      );
      if (response.ok) {
        showNotification("Changes saved successfully!", "success");
      }
    } catch (error) {
      showNotification("Failed to save", "error");
    }
  };

  const handleDelete = () => {
    showNotification("Item deleted", "info");
  };

  return (
    <View style={{ padding: 20 }}>
      <Button title="Save" onPress={handleSave} />
      <Button title="Delete" onPress={handleDelete} color="red" />
    </View>
  );
}

export default function App() {
  return (
    <NotificationProvider>
      <UserActions />
    </NotificationProvider>
  );
}
```

---

## ❓ Interview Questions & Answers

### Q1: What is prop drilling, and how does useContext solve it?

**Answer:**
Prop drilling is passing props through many intermediate components that don't use them:

```javascript
// Without useContext - Prop Drilling Problem
function App() {
  const [theme, setTheme] = useState("light");
  // theme passed through multiple components
  return <Layout theme={theme} />;
}

function Layout({ theme }) {
  return <Header theme={theme} />;
}

function Header({ theme }) {
  return <Button theme={theme} />;
}

function Button({ theme }) {
  return <button style={{ background: theme === "dark" ? "#333" : "#fff" }} />;
}

// With useContext - Clean Solution
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeContext.Provider value={theme}>
      <Layout />
    </ThemeContext.Provider>
  );
}

function Layout() {
  return <Header />; // No theme prop needed
}

function Header() {
  return <Button />; // No theme prop needed
}

function Button() {
  const theme = useContext(ThemeContext); // Get directly from context
  return <button style={{ background: theme === "dark" ? "#333" : "#fff" }} />;
}
```

---

### Q2: What's the difference between createContext default value and Provider value?

**Answer:**

```javascript
// Default value - used if no Provider is wrapping the component
const MyContext = createContext("defaultValue");

function ComponentWithoutProvider() {
  // This will use 'defaultValue' if component is NOT wrapped in provider
  const value = useContext(MyContext); // Returns 'defaultValue'
}

function ComponentWithProvider() {
  return (
    <MyContext.Provider value="providerValue">
      <InnerComponent />
    </MyContext.Provider>
  );
}

function InnerComponent() {
  // This will use 'providerValue' from the Provider
  const value = useContext(MyContext); // Returns 'providerValue'
}
```

**Best Practice:** Always provide a Provider, use default value only for fallback/testing.

---

### Q3: How do you prevent unnecessary re-renders with context?

**Answer:**
Split context into separate pieces or use useMemo to prevent re-renders:

```javascript
// ❌ Problem: Every context change re-renders all consumers
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");

  const value = { user, setUser, theme, setTheme };
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// ✅ Solution 1: Split contexts
const UserContext = createContext();
const ThemeContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        {children}
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// ✅ Solution 2: Memoize context value
function AppProvider({ children }) {
  const [user, setUser] = useState(null);

  const value = useMemo(() => ({ user, setUser }), [user]);

  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}
```

---

### Q4: When should you use context vs other state management solutions?

**Answer:**
| Use Case | Solution |
|----------|----------|
| **Simple global UI state** (theme, language) | Context API |
| **Global auth state** | Context API |
| **Complex state logic** | useReducer + Context |
| **Heavy server state with caching** | TanStack Query |
| **Large app with many features** | Zustand or Redux Toolkit |
| **Real-time data sync** | Zustand + WebSocket |

---

### Q5: Can you use useContext inside useEffect?

**Answer:**
**Yes**, you can use useContext inside useEffect, but be careful with dependencies:

```javascript
function MyComponent() {
  const { userId } = useContext(UserContext);

  useEffect(() => {
    const fetchData = async () => {
      const response = await fetch(`/api/users/${userId}`);
      // Handle response
    };
    fetchData();
  }, [userId]); // Include context value in dependency array
}
```

---

### Q6: What happens if you don't wrap a component with a Provider?

**Answer:**
If a component uses useContext but no Provider is wrapping it:

```javascript
const MyContext = createContext('defaultValue');

function MyComponent() {
  const value = useContext(MyContext); // Will be 'defaultValue'
}

// Without provider
<MyComponent /> // value = 'defaultValue' (if provided)

// With provider
<MyContext.Provider value="custom">
  <MyComponent /> // value = 'custom'
</MyContext.Provider>
```

---

## 🔑 Key Takeaways

1. **Context solves prop drilling** - Pass data deep without intermediate props
2. **Always create custom hooks** - Makes consumption easier (useTheme vs useContext)
3. **Provider wraps components** - Only consumers inside provider get the value
4. **Split complex contexts** - Keep related data together, separate concerns
5. **Memoize context value** - Prevent unnecessary re-renders
6. **Not a replacement for state management** - For simple global state only

---

## 🚨 Common Mistakes

```javascript
// ❌ Forgetting to provide default value
const MyContext = createContext(); // Risky

// ✅ Always provide default value
const MyContext = createContext(null);

// ❌ Creating context inside component (recreated every render)
function App() {
  const MyContext = createContext(); // New context each render!
}

// ✅ Create context outside component
const MyContext = createContext();

function App() {
  // ...
}

// ❌ Not memoizing context value (causes re-renders)
function Provider({ children }) {
  const value = { theme: isDark, toggleTheme }; // New object every render
  return <Context.Provider value={value}>{children}</Context.Provider>;
}

// ✅ Memoize the value
function Provider({ children }) {
  const value = useMemo(() => ({ theme: isDark, toggleTheme }), [isDark]);
  return <Context.Provider value={value}>{children}</Context.Provider>;
}
```

---

## 📚 Practice Exercise

Create a "Settings Provider" that manages:

1. **Theme** (light/dark)
2. **Language** (en/es/fr)
3. **Font Size** (small/medium/large)
4. **Notifications** (on/off)

Requirements:

- Create separate contexts for each setting (to prevent unnecessary re-renders)
- Create a custom hook to combine all settings
- Persist settings to localStorage
- Restore settings on app startup
- Create a Settings screen to change all preferences

---

**Next:** Learn about [useReducer Hook](./04-useReducer.md)
