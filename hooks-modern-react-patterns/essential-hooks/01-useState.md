# useState Hook - Complete Guide

---

## 📌 Definition

**useState** is a React Hook that lets you add state to functional components. It allows you to create and update component-level data that, when changed, triggers a re-render of the component.

---

## 🎯 Purpose & What It Does

### Purpose

- **Add state to functional components** - Functional components were stateless before hooks; useState made them stateful
- **Trigger re-renders** - When state changes, the component re-renders with the new data
- **Track component data** - Any data that changes over time (user input, toggles, counters, etc.)

### What It Actually Does

1. Declares a state variable and a setter function
2. Returns an array with 2 elements: `[currentState, setterFunction]`
3. Calling the setter function updates the state AND triggers a re-render
4. On re-render, useState returns the updated state value

---

## 🔄 Class Component vs Hooks Comparison

### Class Component (Old Way)

```javascript
class Counter extends React.Component {
  constructor(props) {
    super(props);
    // State stored in this.state object
    this.state = {
      count: 0,
      name: "John",
    };
  }

  increment = () => {
    // Update state with this.setState()
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <p>Name: {this.state.name}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

### Why We Switched to Hooks

| Aspect              | Class Component                     | Hooks (useState)        |
| ------------------- | ----------------------------------- | ----------------------- |
| **Syntax**          | Complex constructor, this.state     | Simple, declarative     |
| **Multiple States** | All in one this.state object        | Separate useState calls |
| **Readability**     | Mixed logic spread across lifecycle | Grouped related logic   |
| **Reusability**     | Use HOC or render props             | Simple custom hooks     |
| **Bundle Size**     | Larger code footprint               | Smaller, more optimized |

---

## 💡 Real-World Use Cases

1. **Form Input Values** - Track input field states
2. **Toggle Visibility** - Show/hide modals, menus, dropdowns
3. **Counter Applications** - Like/dislike counters, item quantities
4. **Loading States** - Track data fetching status
5. **User Preferences** - Theme selection, language preference
6. **Shopping Cart** - Items count, total price

---

## 📝 Hook Examples

### Example 1: Basic Counter

```javascript
import React, { useState } from "react";
import { View, Text, Button, StyleSheet } from "react-native";

function Counter() {
  // Declare state variable 'count' with initial value 0
  // setState function to update it
  const [count, setCount] = useState(0);

  return (
    <View style={styles.container}>
      <Text style={styles.text}>Count: {count}</Text>
      <Button title="Increment" onPress={() => setCount(count + 1)} />
      <Button title="Decrement" onPress={() => setCount(count - 1)} />
      <Button title="Reset" onPress={() => setCount(0)} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
  text: { fontSize: 24, marginBottom: 20 },
});

export default Counter;
```

---

### Example 2: Form Input Tracking

```javascript
import React, { useState } from "react";
import { View, TextInput, Text, Button } from "react-native";

function UserForm() {
  // Multiple state variables for different form fields
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [isSubmitted, setIsSubmitted] = useState(false);

  const handleSubmit = () => {
    if (email && password) {
      console.log("Form submitted:", { email, password });
      setIsSubmitted(true);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        placeholder="Enter email"
        value={email}
        onChangeText={setEmail}
        style={{ borderBottomWidth: 1, marginBottom: 10, padding: 10 }}
      />
      <TextInput
        placeholder="Enter password"
        secureTextEntry
        value={password}
        onChangeText={setPassword}
        style={{ borderBottomWidth: 1, marginBottom: 20, padding: 10 }}
      />
      <Button title="Submit" onPress={handleSubmit} />
      {isSubmitted && (
        <Text style={{ color: "green", marginTop: 20 }}>Form submitted!</Text>
      )}
    </View>
  );
}

export default UserForm;
```

---

### Example 3: Toggle Visibility (Modal)

```javascript
import React, { useState } from "react";
import { View, Button, Modal, Text } from "react-native";

function ModalExample() {
  const [modalVisible, setModalVisible] = useState(false);

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Button title="Open Modal" onPress={() => setModalVisible(true)} />

      <Modal
        animationType="slide"
        transparent={true}
        visible={modalVisible}
        onRequestClose={() => setModalVisible(false)}
      >
        <View
          style={{ flex: 1, justifyContent: "center", alignItems: "center" }}
        >
          <Text style={{ fontSize: 20, marginBottom: 20 }}>Modal Content</Text>
          <Button title="Close Modal" onPress={() => setModalVisible(false)} />
        </View>
      </Modal>
    </View>
  );
}

export default ModalExample;
```

---

### Example 4: API Call with useState

```javascript
import React, { useState, useEffect } from "react";
import { View, Text, Button, ActivityIndicator } from "react-native";

function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // Dummy API call function
  const fetchUser = async () => {
    setLoading(true);
    setError(null);
    try {
      // Simulating API call to https://xyzzy.com/api/user/1
      const response = await fetch(
        "https://jsonplaceholder.typicode.com/users/1",
      );
      const data = await response.json();
      setUser(data);
    } catch (err) {
      setError("Failed to fetch user");
      console.error(err);
    } finally {
      setLoading(false);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <Button title="Load User" onPress={fetchUser} />

      {loading && <ActivityIndicator size="large" color="#0000ff" />}

      {error && <Text style={{ color: "red" }}>{error}</Text>}

      {user && (
        <View style={{ marginTop: 20 }}>
          <Text>Name: {user.name}</Text>
          <Text>Email: {user.email}</Text>
          <Text>Phone: {user.phone}</Text>
        </View>
      )}
    </View>
  );
}

export default UserProfile;
```

---

### Example 5: Shopping Cart Counter

```javascript
import React, { useState } from "react";
import { View, Text, Button, StyleSheet } from "react-native";

function ProductCard({ productName, price }) {
  const [quantity, setQuantity] = useState(0);

  const addToCart = () => setQuantity(quantity + 1);
  const removeFromCart = () => {
    if (quantity > 0) setQuantity(quantity - 1);
  };

  const totalPrice = (quantity * price).toFixed(2);

  return (
    <View style={styles.card}>
      <Text style={styles.name}>{productName}</Text>
      <Text style={styles.price}>${price}</Text>
      <Text style={styles.quantity}>Quantity: {quantity}</Text>
      <Text style={styles.total}>Total: ${totalPrice}</Text>

      <View style={styles.buttonContainer}>
        <Button title="+" onPress={addToCart} />
        <Button title="-" onPress={removeFromCart} color="red" />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  card: { padding: 15, borderWidth: 1, marginBottom: 10, borderRadius: 8 },
  name: { fontSize: 18, fontWeight: "bold" },
  price: { fontSize: 16, color: "green" },
  quantity: { fontSize: 14, marginTop: 10 },
  total: { fontSize: 16, fontWeight: "bold", marginTop: 5 },
  buttonContainer: {
    flexDirection: "row",
    justifyContent: "space-around",
    marginTop: 10,
  },
});

export default ProductCard;
```

---

## ❓ Interview Questions & Answers

### Q1: What is the difference between useState and this.state in class components?

**Answer:**

- `useState` is a hook for functional components, while `this.state` is for class components
- `useState` allows you to have multiple independent state variables
- In class components, all state is stored in one object (this.state)
- `useState` is more concise and easier to manage

**Example:**

```javascript
// Class: all states in one object
this.state = { count: 0, name: "John", isActive: true };
this.setState({ count: 1 });

// Hooks: separate state variables
const [count, setCount] = useState(0);
const [name, setName] = useState("John");
const [isActive, setIsActive] = useState(true);
setCount(1);
```

---

### Q2: What happens when you call the setState function?

**Answer:**
When you call a setState function (e.g., `setCount(5)`):

1. The state variable is updated with the new value
2. React marks the component as "dirty" (needs re-render)
3. React schedules a re-render
4. The component function runs again with the updated state value
5. The UI updates with the new values

---

### Q3: Can you use useState multiple times in a component?

**Answer:**
Yes! You can call useState as many times as you need:

```javascript
function ComplexForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [phone, setPhone] = useState("");
  const [isSubmitted, setIsSubmitted] = useState(false);
  // ... more useState calls
}
```

**Important:** The order of useState calls must be consistent across renders (no conditional hooks).

---

### Q4: What is the initial value in useState, and how do you set it?

**Answer:**
The initial value is the first argument to useState:

```javascript
const [count, setCount] = useState(0); // Initial value: 0
const [user, setUser] = useState(null); // Initial value: null
const [items, setItems] = useState([]); // Initial value: empty array
const [data, setData] = useState({ name: "", age: 0 }); // Initial value: object

// For complex initial values, use a function (lazy initialization)
const [largeData, setLargeData] = useState(() => {
  return expensiveComputation();
});
```

---

### Q5: How do you update state based on the previous state?

**Answer:**
Use a function in the setState call:

```javascript
// ❌ Wrong - might miss updates if called multiple times
setCount(count + 1);

// ✅ Correct - uses previous state
setCount((prevCount) => prevCount + 1);

// This is important in async scenarios
setCount((prevCount) => prevCount + 1);
setCount((prevCount) => prevCount + 1); // Both increments will apply
```

---

### Q6: What is the difference between state and props?

**Answer:**
| Aspect | State | Props |
|--------|-------|-------|
| **Ownership** | Owned by component | Passed from parent |
| **Mutable** | Can change with setState | Immutable (read-only) |
| **When to use** | Component-specific data | Pass data to children |
| **Example** | Form input value | Theme from parent |

---

## 🔑 Key Takeaways

1. **useState is for component-level state** - Use it to track data that changes
2. **Setter function triggers re-render** - Updating state causes the component to re-render
3. **Declarative and simple** - Much cleaner than class component state
4. **Multiple states** - Use separate useState calls for different data
5. **Use functional updates** - When new state depends on previous state

---

## 📚 Practice Exercise

Create a simple Todo App component using useState that:

1. Has an input field for adding new todos
2. Displays a list of todos
3. Can mark todos as complete/incomplete
4. Can delete todos
5. Shows count of completed vs total todos

**Hint:** You might need multiple useState calls for: todos list, current input value, and completion status.

---

**Next:** Learn about [useEffect Hook](./02-useEffect.md)
