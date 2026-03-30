# useMemo Hook - Complete Guide

---

## 📌 Definition

**useMemo** is a React Hook that lets you cache (memoize) a computed value between re-renders. It only recalculates the value when its dependencies have changed.

---

## 🎯 Purpose & What It Does

### Purpose

- **Cache expensive computations** - Avoid recalculating on every render
- **Prevent unnecessary calculations** - Only recompute when dependencies change
- **Optimize performance** - Especially useful for heavy operations
- **Prevent child re-renders** - When memoized value is passed to memo'd components

### What It Actually Does

1. Takes a function and dependency array
2. Runs the function and caches the result
3. Returns cached result on subsequent renders
4. Only recomputes when dependencies change
5. Otherwise returns the same cached value

---

## 💡 Real-World Use Cases

1. **Expensive calculations** - Complex math, sorting, filtering
2. **Array/Object creation** - Creating new objects for props
3. **Filtering lists** - Filter and sort large datasets
4. **Derived state** - Calculate values from props/state
5. **Search/Find operations** - Complex search algorithms
6. **Transformations** - Format data for display

---

## 📝 Hook Examples

### Example 1: Basic Expensive Calculation

```javascript
import React, { useState, useMemo } from "react";
import { View, Text, Button, Slider } from "react-native";

function FactorialCalculator() {
  const [number, setNumber] = useState(5);
  const [count, setCount] = useState(0);

  // ❌ Without useMemo - recalculates every render
  // const factorial = calculateFactorial(number);

  // ✅ With useMemo - caches result
  const factorial = useMemo(() => {
    console.log("Calculating factorial...");
    let result = 1;
    for (let i = 2; i <= number; i++) {
      result *= i;
    }
    return result;
  }, [number]); // Only recalculate when number changes

  return (
    <View style={{ padding: 20 }}>
      <Text>Number: {number}</Text>
      <Text>Factorial: {factorial}</Text>

      <Text style={{ marginTop: 10 }}>Increment Count: {count}</Text>

      {/* Clicking this doesn't recalculate factorial */}
      <Button title="Increment Count" onPress={() => setCount(count + 1)} />
    </View>
  );
}

export default FactorialCalculator;
```

---

### Example 2: Filtering and Sorting Large List

```javascript
import React, { useState, useMemo } from "react";
import { View, Text, TextInput, FlatList, StyleSheet } from "react-native";

function UserList() {
  const [searchQuery, setSearchQuery] = useState("");
  const [sortBy, setSortBy] = useState("name"); // 'name' or 'email'
  const [renderCount, setRenderCount] = useState(0);

  // Large dataset
  const users = Array.from({ length: 1000 }, (_, i) => ({
    id: i + 1,
    name: `User ${i + 1}`,
    email: `user${i + 1}@example.com`,
    age: Math.floor(Math.random() * 50) + 20,
  }));

  // ✅ Memoize filtered and sorted results
  const filteredAndSortedUsers = useMemo(() => {
    console.log("Filtering and sorting...");

    let result = users.filter(
      (user) =>
        user.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
        user.email.toLowerCase().includes(searchQuery.toLowerCase()),
    );

    result.sort((a, b) => {
      if (sortBy === "name") {
        return a.name.localeCompare(b.name);
      } else if (sortBy === "email") {
        return a.email.localeCompare(b.email);
      }
      return 0;
    });

    return result;
  }, [searchQuery, sortBy, users]);

  return (
    <View style={styles.container}>
      <TextInput
        placeholder="Search users..."
        value={searchQuery}
        onChangeText={setSearchQuery}
        style={styles.input}
      />

      <Text>Sort By: {sortBy}</Text>
      <Button
        title={`Switch to ${sortBy === "name" ? "Email" : "Name"}`}
        onPress={() => setSortBy(sortBy === "name" ? "email" : "name")}
      />

      <Text>Results: {filteredAndSortedUsers.length}</Text>
      <Text>Render Count: {renderCount}</Text>

      {/* Only re-filters when searchQuery or sortBy changes */}
      <FlatList
        data={filteredAndSortedUsers}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.userItem}>
            <Text>{item.name}</Text>
            <Text style={styles.email}>{item.email}</Text>
          </View>
        )}
      />

      {/* Clicking this doesn't re-filter */}
      <Button
        title="Force Render"
        onPress={() => setRenderCount(renderCount + 1)}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 10 },
  input: { borderBottomWidth: 1, padding: 10, marginBottom: 10 },
  userItem: { padding: 10, borderBottomWidth: 1 },
  email: { fontSize: 12, color: "#666" },
});

export default UserList;
```

---

### Example 3: Computing Derived State

```javascript
import React, { useState, useMemo } from "react";
import { View, Text, Button, StyleSheet } from "react-native";

function ShoppingCart() {
  const [items, setItems] = useState([
    { id: 1, name: "Laptop", price: 1000, quantity: 1 },
    { id: 2, name: "Mouse", price: 25, quantity: 2 },
    { id: 3, name: "Keyboard", price: 75, quantity: 1 },
  ]);
  const [discountPercent, setDiscountPercent] = useState(0);

  // Compute cart summary with memoization
  const cartSummary = useMemo(() => {
    console.log("Computing cart summary...");

    const subtotal = items.reduce((sum, item) => {
      return sum + item.price * item.quantity;
    }, 0);

    const tax = subtotal * 0.1; // 10% tax
    const discountAmount = subtotal * (discountPercent / 100);
    const total = subtotal + tax - discountAmount;

    return {
      subtotal: subtotal.toFixed(2),
      tax: tax.toFixed(2),
      discount: discountAmount.toFixed(2),
      total: total.toFixed(2),
      itemCount: items.length,
    };
  }, [items, discountPercent]);

  return (
    <View style={styles.container}>
      <Text style={styles.header}>Shopping Cart</Text>

      {items.map((item) => (
        <View key={item.id} style={styles.item}>
          <Text>{item.name}</Text>
          <Text>
            ${item.price} x {item.quantity} = $
            {(item.price * item.quantity).toFixed(2)}
          </Text>
        </View>
      ))}

      <View style={styles.summary}>
        <Text>Subtotal: ${cartSummary.subtotal}</Text>
        <Text>Tax (10%): ${cartSummary.tax}</Text>
        <Text>
          Discount ({discountPercent}%): -${cartSummary.discount}
        </Text>
        <Text style={styles.total}>Total: ${cartSummary.total}</Text>
      </View>

      <Button
        title={`Apply 10% Discount`}
        onPress={() => setDiscountPercent(10)}
      />
      <Button title={`Clear Discount`} onPress={() => setDiscountPercent(0)} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  header: { fontSize: 20, fontWeight: "bold", marginBottom: 10 },
  item: { paddingVertical: 10, borderBottomWidth: 1 },
  summary: {
    marginTop: 20,
    padding: 10,
    backgroundColor: "#f5f5f5",
    borderRadius: 5,
  },
  total: { fontSize: 18, fontWeight: "bold", marginTop: 10 },
});

export default ShoppingCart;
```

---

### Example 4: Creating Objects for Props

```javascript
import React, { useState, useMemo, memo } from "react";
import { View, Text, Button } from "react-native";

// Memoized child that receives an object
const ThemeDisplay = memo(({ theme }) => {
  console.log("ThemeDisplay rendered");
  return (
    <View style={{ backgroundColor: theme.primary, padding: 20 }}>
      <Text style={{ color: theme.text }}>Primary Color: {theme.primary}</Text>
      <Text style={{ color: theme.text }}>Text Color: {theme.text}</Text>
    </View>
  );
});

function ThemeApp() {
  const [isDark, setIsDark] = useState(false);
  const [count, setCount] = useState(0);

  // ❌ Without useMemo - creates new object every render
  // const theme = {
  //   primary: isDark ? '#333' : '#fff',
  //   text: isDark ? '#fff' : '#000'
  // };

  // ✅ With useMemo - same object reference when isDark hasn't changed
  const theme = useMemo(
    () => ({
      primary: isDark ? "#333" : "#fff",
      text: isDark ? "#fff" : "#000",
    }),
    [isDark],
  );

  return (
    <View style={{ padding: 20 }}>
      <Text>Count: {count}</Text>
      <ThemeDisplay theme={theme} />

      {/* Clicking this doesn't recreate theme or re-render ThemeDisplay */}
      <Button title="Increment Count" onPress={() => setCount(count + 1)} />

      {/* Clicking this recreates theme and re-renders ThemeDisplay */}
      <Button title="Toggle Theme" onPress={() => setIsDark(!isDark)} />
    </View>
  );
}

export default ThemeApp;
```

---

## ❓ Interview Questions & Answers

### Q1: When should you use useMemo?

**Answer:**
Use useMemo when:

1. Computation is expensive (noticeable lag without it)
2. Value is passed to memoized child components
3. Value is a dependency in effects/callbacks

```javascript
// ✅ Good use - expensive calculation
const expensiveValue = useMemo(() => {
  return complexAlgorithm(data);
}, [data]);

// ✅ Good use - object for memoized child
const config = useMemo(() => ({ ...settings }), [settings]);

// ❌ Unnecessary - simple calculation
const doubled = useMemo(() => count * 2, [count]);
```

---

### Q2: What's the performance cost of useMemo?

**Answer:**
useMemo itself has overhead:

1. Comparing dependencies
2. Storing result in memory
3. Creating closure

Only use when the computation is more expensive than the useMemo overhead.

```javascript
// ❌ Not worth it - useMemo overhead > computation
const doubled = useMemo(() => count * 2, [count]);

// ✅ Worth it - expensive computation
const sorted = useMemo(() => {
  return items.sort((a, b) => {
    // Complex sorting logic
  });
}, [items]);
```

---

### Q3: Can useMemo be used instead of useCallback?

**Answer:**
Yes, technically:

```javascript
// Using useCallback
const memoizedCallback = useCallback(() => {
  return count + 1;
}, [count]);

// Equivalent with useMemo
const memoizedCallback = useMemo(() => {
  return () => count + 1;
}, [count]);
```

But useCallback is clearer when memoizing functions.

---

### Q4: What happens if you don't include a dependency?

**Answer:**
The value is memoized but never updates:

```javascript
// ❌ Wrong - value never updates
const result = useMemo(() => {
  return computeValue(data);
}, []); // Missing data dependency

// When data changes, result still has old value!

// ✅ Correct
const result = useMemo(() => {
  return computeValue(data);
}, [data]); // Include data
```

---

## 🔑 Key Takeaways

1. **useMemo caches computed values** - Prevents recalculation
2. **Check if it's necessary** - Only for expensive operations
3. **Benchmark first** - Measure performance before/after
4. **Include all dependencies** - Or value will be stale
5. **Not a silver bullet** - Over-using hurts performance

---

## 📚 Practice Exercise

Create a **Product Filter & Search** with:

1. Large product list (500+ items)
2. Search input (filters by name/description)
3. Price range slider (filters by price)
4. Category selector (filters by category)
5. Sort options (by name, price, rating)
6. Display filtered results count
7. Use useMemo to optimize filtering/sorting
8. Log when filtering happens

---

**Next:** Learn about [useRef Hook](./07-useRef.md)
