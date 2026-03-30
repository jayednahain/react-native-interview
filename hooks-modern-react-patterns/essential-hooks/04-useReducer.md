# useReducer Hook - Complete Guide

---

## 📌 Definition

**useReducer** is a React Hook for managing complex state logic. It lets you consolidate multiple pieces of state into a single state object and update them using actions, similar to Redux pattern. It's an alternative to useState for more complex scenarios.

---

## 🎯 Purpose & What It Does

### Purpose

- **Manage complex state** - Multiple related state variables
- **State logic in actions** - Centralize update logic
- **Easier testing** - Reducer logic is pure function
- **Handle related updates** - Update multiple state together
- **Prevent bugs** - Clearer state transitions

### What It Actually Does

1. Takes a reducer function and initial state
2. Returns current state and a dispatch function
3. dispatch function sends actions to the reducer
4. Reducer returns new state based on action
5. Component re-renders with new state

---

## 🔄 Class Component vs Hooks Comparison

### Class Component (Old Way)

```javascript
class Counter extends React.Component {
  constructor(props) {
    super(props);
    // Complex state with multiple properties
    this.state = {
      count: 0,
      step: 1,
      history: [0],
      maxCount: 0,
    };
  }

  // Multiple update methods
  increment = () => {
    this.setState((prev) => ({
      count: prev.count + prev.step,
      maxCount: Math.max(prev.maxCount, prev.count + prev.step),
      history: [...prev.history, prev.count + prev.step],
    }));
  };

  decrement = () => {
    this.setState((prev) => ({
      count: prev.count - prev.step,
      history: [...prev.history, prev.count - prev.step],
    }));
  };

  setStep = (step) => {
    this.setState({ step });
  };

  reset = () => {
    this.setState({
      count: 0,
      history: [0],
      maxCount: 0,
    });
  };

  render() {
    return <div>{this.state.count}</div>;
  }
}
```

### Hooks Version (Modern Way)

```javascript
// Reducer function (pure function)
function counterReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return {
        ...state,
        count: state.count + state.step,
        maxCount: Math.max(state.maxCount, state.count + state.step),
        history: [...state.history, state.count + state.step],
      };
    case "DECREMENT":
      return {
        ...state,
        count: state.count - state.step,
        history: [...state.history, state.count - state.step],
      };
    case "SET_STEP":
      return { ...state, step: action.payload };
    case "RESET":
      return {
        count: 0,
        step: 1,
        history: [0],
        maxCount: 0,
      };
    default:
      return state;
  }
}

function Counter() {
  const initialState = {
    count: 0,
    step: 1,
    history: [0],
    maxCount: 0,
  };

  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
      <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
    </div>
  );
}
```

### Why We Switched to Hooks

| Aspect            | Class Component         | useReducer                 |
| ----------------- | ----------------------- | -------------------------- |
| **Code org**      | Multiple methods        | Single reducer function    |
| **State updates** | Scattered logic         | Centralized in reducer     |
| **Testing**       | Need to test component  | Test pure reducer function |
| **Reusability**   | Logic tied to component | Reducer logic is reusable  |
| **Readability**   | Multiple update methods | Single dispatch pattern    |

---

## useState vs useReducer

### Use useState When

```javascript
// Simple, independent state
function Counter() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");
  // Each state is independent
}
```

### Use useReducer When

```javascript
// Complex, related state OR multiple updates
function Form() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  // State values are related
  // Multiple pieces update together
}
```

---

## 💡 Real-World Use Cases

1. **Todo App** - Add/remove/toggle todos
2. **Shopping Cart** - Add/remove/update quantities
3. **Form Handling** - Complex form with validation
4. **Authentication** - User login/logout/session
5. **Game State** - Player moves, score, lives
6. **Undo/Redo** - Track history of actions
7. **Notifications** - Add/remove toasts

---

## 📝 Hook Examples

### Example 1: Simple Counter with Actions

```javascript
import React, { useReducer } from "react";
import { View, Text, Button } from "react-native";

// Reducer function (pure function)
function counterReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    case "RESET":
      return 0;
    case "SET":
      return action.payload;
    default:
      return state;
  }
}

function Counter() {
  const [count, dispatch] = useReducer(counterReducer, 0);

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 24 }}>Count: {count}</Text>
      <Button title="+" onPress={() => dispatch({ type: "INCREMENT" })} />
      <Button title="-" onPress={() => dispatch({ type: "DECREMENT" })} />
      <Button title="Reset" onPress={() => dispatch({ type: "RESET" })} />
      <Button
        title="Set to 10"
        onPress={() => dispatch({ type: "SET", payload: 10 })}
      />
    </View>
  );
}

export default Counter;
```

---

### Example 2: Todo App with useReducer

```javascript
import React, { useReducer, useState } from "react";
import {
  View,
  Text,
  TextInput,
  Button,
  FlatList,
  StyleSheet,
} from "react-native";

// Reducer function
function todoReducer(state, action) {
  switch (action.type) {
    case "ADD_TODO":
      return [
        ...state,
        { id: Date.now(), text: action.payload, completed: false },
      ];

    case "REMOVE_TODO":
      return state.filter((todo) => todo.id !== action.payload);

    case "TOGGLE_TODO":
      return state.map((todo) =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo,
      );

    case "CLEAR_COMPLETED":
      return state.filter((todo) => !todo.completed);

    default:
      return state;
  }
}

function TodoApp() {
  const [todos, dispatch] = useReducer(todoReducer, []);
  const [input, setInput] = useState("");

  const handleAddTodo = () => {
    if (input.trim()) {
      dispatch({ type: "ADD_TODO", payload: input });
      setInput("");
    }
  };

  const completedCount = todos.filter((todo) => todo.completed).length;

  return (
    <View style={styles.container}>
      <Text style={styles.header}>My Todo List</Text>

      {/* Input */}
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Add a new todo..."
          value={input}
          onChangeText={setInput}
        />
        <Button title="Add" onPress={handleAddTodo} />
      </View>

      {/* Stats */}
      <Text style={styles.stats}>
        Total: {todos.length} | Completed: {completedCount}
      </Text>

      {/* Todo List */}
      <FlatList
        data={todos}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.todoItem}>
            <Text
              style={[styles.todoText, item.completed && styles.completedText]}
              onPress={() =>
                dispatch({ type: "TOGGLE_TODO", payload: item.id })
              }
            >
              {item.text}
            </Text>
            <Button
              title="Delete"
              onPress={() =>
                dispatch({ type: "REMOVE_TODO", payload: item.id })
              }
              color="red"
            />
          </View>
        )}
      />

      {/* Clear Completed */}
      {completedCount > 0 && (
        <Button
          title={`Clear ${completedCount} Completed`}
          onPress={() => dispatch({ type: "CLEAR_COMPLETED" })}
        />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  header: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  inputContainer: { flexDirection: "row", marginBottom: 20 },
  input: { flex: 1, borderBottomWidth: 1, paddingHorizontal: 10 },
  stats: { fontSize: 14, color: "#666", marginBottom: 10 },
  todoItem: {
    flexDirection: "row",
    justifyContent: "space-between",
    paddingVertical: 10,
    borderBottomWidth: 1,
  },
  todoText: { fontSize: 16, flex: 1 },
  completedText: { textDecorationLine: "line-through", color: "#999" },
});

export default TodoApp;
```

---

### Example 3: Complex Form with Validation

```javascript
import React, { useReducer } from "react";
import { View, Text, TextInput, Button, StyleSheet } from "react-native";

const initialState = {
  email: "",
  password: "",
  confirmPassword: "",
  errors: {},
  isSubmitting: false,
};

function formReducer(state, action) {
  switch (action.type) {
    case "FIELD_CHANGE":
      return {
        ...state,
        [action.field]: action.value,
        errors: { ...state.errors, [action.field]: "" }, // Clear error
      };

    case "SET_ERRORS":
      return { ...state, errors: action.payload };

    case "SUBMIT_START":
      return { ...state, isSubmitting: true };

    case "SUBMIT_END":
      return { ...state, isSubmitting: false };

    case "RESET":
      return initialState;

    default:
      return state;
  }
}

function SignupForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  const validate = () => {
    const errors = {};

    if (!state.email.includes("@")) {
      errors.email = "Invalid email";
    }

    if (state.password.length < 6) {
      errors.password = "Password must be 6+ characters";
    }

    if (state.password !== state.confirmPassword) {
      errors.confirmPassword = "Passwords do not match";
    }

    return errors;
  };

  const handleSubmit = async () => {
    const errors = validate();

    if (Object.keys(errors).length > 0) {
      dispatch({ type: "SET_ERRORS", payload: errors });
      return;
    }

    dispatch({ type: "SUBMIT_START" });

    try {
      // Simulating API call to https://xyzzy.com/api/signup
      const response = await fetch(
        "https://jsonplaceholder.typicode.com/users",
        {
          method: "POST",
          body: JSON.stringify({
            email: state.email,
            password: state.password,
          }),
        },
      );

      if (response.ok) {
        alert("Signup successful!");
        dispatch({ type: "RESET" });
      }
    } catch (error) {
      dispatch({
        type: "SET_ERRORS",
        payload: { form: "Signup failed. Please try again." },
      });
    } finally {
      dispatch({ type: "SUBMIT_END" });
    }
  };

  const handleChange = (field, value) => {
    dispatch({ type: "FIELD_CHANGE", field, value });
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sign Up</Text>

      {state.errors.form && (
        <Text style={styles.error}>{state.errors.form}</Text>
      )}

      <Text>Email</Text>
      <TextInput
        style={[styles.input, state.errors.email && styles.inputError]}
        placeholder="Enter email"
        value={state.email}
        onChangeText={(value) => handleChange("email", value)}
      />
      {state.errors.email && (
        <Text style={styles.error}>{state.errors.email}</Text>
      )}

      <Text style={{ marginTop: 10 }}>Password</Text>
      <TextInput
        style={[styles.input, state.errors.password && styles.inputError]}
        placeholder="Enter password"
        secureTextEntry
        value={state.password}
        onChangeText={(value) => handleChange("password", value)}
      />
      {state.errors.password && (
        <Text style={styles.error}>{state.errors.password}</Text>
      )}

      <Text style={{ marginTop: 10 }}>Confirm Password</Text>
      <TextInput
        style={[
          styles.input,
          state.errors.confirmPassword && styles.inputError,
        ]}
        placeholder="Confirm password"
        secureTextEntry
        value={state.confirmPassword}
        onChangeText={(value) => handleChange("confirmPassword", value)}
      />
      {state.errors.confirmPassword && (
        <Text style={styles.error}>{state.errors.confirmPassword}</Text>
      )}

      <Button
        title={state.isSubmitting ? "Signing up..." : "Sign Up"}
        onPress={handleSubmit}
        disabled={state.isSubmitting}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  input: { borderBottomWidth: 1, paddingVertical: 10, marginBottom: 5 },
  inputError: { borderBottomColor: "red" },
  error: { color: "red", marginBottom: 10, fontSize: 12 },
});

export default SignupForm;
```

---

## ❓ Interview Questions & Answers

### Q1: What's the difference between useState and useReducer?

**Answer:**

```javascript
// useState - for simple state
const [count, setCount] = useState(0);
setCount(count + 1); // Direct update

// useReducer - for complex state
const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: "INCREMENT" }); // Action-based update

// Choose useReducer when:
// - Multiple related state values
// - Complex update logic
// - Updating several pieces of state at once
// - Want to optimize performance (can pass dispatch to child components)
```

---

### Q2: What is a pure reducer function?

**Answer:**
A pure reducer function:

1. Takes current state and action as arguments
2. Returns new state
3. Has no side effects
4. Produces the same output for the same input
5. Never mutates the input state

```javascript
// ❌ Impure reducer (has side effects)
function impureReducer(state, action) {
  console.log("Reducer called"); // Side effect
  state.count++; // Mutation
  return state;
}

// ✅ Pure reducer
function pureReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 }; // New object
    default:
      return state;
  }
}
```

---

### Q3: How do you pass data to a reducer action?

**Answer:**
Use the `payload` property in the action object:

```javascript
function myReducer(state, action) {
  switch (action.type) {
    case "SET_NAME":
      return { ...state, name: action.payload };
    case "ADD_ITEM":
      return { ...state, items: [...state.items, action.payload] };
    default:
      return state;
  }
}

// Dispatching with payload
dispatch({ type: "SET_NAME", payload: "John" });
dispatch({ type: "ADD_ITEM", payload: { id: 1, text: "Todo" } });
```

---

### Q4: When should you use useReducer over useState?

**Answer:**
| Scenario | Use |
|----------|-----|
| Single piece of state | useState |
| Multiple related state | useReducer |
| Complex update logic | useReducer |
| Need to pass dispatch to children | useReducer |
| Optimizing performance | useReducer |
| Testing state logic | useReducer (pure function) |

---

### Q5: How do you initialize useReducer with computed values?

**Answer:**
Use lazy initialization:

```javascript
// ❌ Runs every render
const [state, dispatch] = useReducer(
  reducer,
  computeExpensiveInitialState(), // Runs every time!
);

// ✅ Runs once
function init(initialCount) {
  return { count: initialCount * 2 };
}

const [state, dispatch] = useReducer(
  reducer,
  initialCount,
  init, // Function that initializes state
);
```

---

## 🔑 Key Takeaways

1. **useReducer for complex state** - Multiple related values
2. **Pure reducer functions** - Easier to test and understand
3. **Action-based updates** - Clearer intent than direct setState
4. **Easier refactoring** - Logic concentrated in one place
5. **Can optimize child components** - Pass dispatch instead of multiple callbacks

---

## 🚨 Common Mistakes

```javascript
// ❌ Mutating state directly
function badReducer(state, action) {
  state.count++; // WRONG!
  return state;
}

// ✅ Create new state object
function goodReducer(state, action) {
  return { ...state, count: state.count + 1 };
}

// ❌ Forgetting to handle default case
function badReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    // Missing default!
  }
}

// ✅ Always include default case
function goodReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    default:
      return state;
  }
}
```

---

## 📚 Practice Exercise

Create a **Shopping Cart** with useReducer that:

1. Add items to cart
2. Remove items from cart
3. Update item quantity
4. Calculate total price
5. Apply discount codes (reducers action)
6. Clear entire cart
7. Show cart summary (count, total)

---

**Next:** Learn about [useCallback Hook](./05-useCallback.md)
