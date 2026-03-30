# useForm Hook - Form State Management with React Hook Form

## 📌 Definition

**useForm** (from `react-hook-form`) is a hook that manages form state with minimal re-renders. It provides form registration, validation, error handling, and submission handling in a performant way without unnecessary re-renders.

---

## 🎯 Purpose & What It Does

- **Manages form state** - values, errors, touched fields
- **Minimal re-renders** - only re-renders affected fields
- **Built-in validation** - sync and async validation
- **Error handling** - field-level and form-level errors
- **Performance** - uncontrolled components by default

---

## 💡 Real-World Examples

### Example 1: Simple Login Form

```javascript
import { useForm, Controller } from "react-hook-form";
import { View, TextInput, TouchableOpacity, Text } from "react-native";

function LoginForm() {
  const {
    control,
    handleSubmit,
    formState: { errors },
  } = useForm({
    defaultValues: {
      email: "",
      password: "",
    },
  });

  const onSubmit = (data) => {
    console.log("Form data:", data);
    // Submit to API
  };

  return (
    <View>
      <Controller
        control={control}
        name="email"
        rules={{ required: "Email is required" }}
        render={({ field: { value, onChange } }) => (
          <TextInput
            value={value}
            onChangeText={onChange}
            placeholder="Email"
          />
        )}
      />
      {errors.email && <Text>{errors.email.message}</Text>}

      <Controller
        control={control}
        name="password"
        rules={{ required: "Password is required" }}
        render={({ field: { value, onChange } }) => (
          <TextInput
            value={value}
            onChangeText={onChange}
            placeholder="Password"
            secureTextEntry
          />
        )}
      />
      {errors.password && <Text>{errors.password.message}</Text>}

      <TouchableOpacity onPress={handleSubmit(onSubmit)}>
        <Text>Login</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 2: Form with Validation

```javascript
import { useForm, Controller } from "react-hook-form";
import { Yup } from "yup";

function SignupForm() {
  const validationSchema = Yup.object({
    name: Yup.string().required("Name required").min(2),
    email: Yup.string().required("Email required").email("Invalid email"),
    password: Yup.string().required("Password required").min(8),
    confirmPassword: Yup.string().oneOf(
      [Yup.ref("password")],
      "Passwords must match",
    ),
  });

  const {
    control,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: yupResolver(validationSchema),
  });

  return <View>{/* Form fields with validation */}</View>;
}
```

### Example 3: Dynamic Fields with useFieldArray

```javascript
import { useForm, Controller, useFieldArray } from "react-hook-form";
import {
  View,
  TouchableOpacity,
  Text,
  TextInput,
  FlatList,
} from "react-native";

function DynamicForm() {
  const { control, handleSubmit } = useForm({
    defaultValues: {
      contacts: [{ name: "", email: "" }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: "contacts",
  });

  return (
    <View>
      <FlatList
        data={fields}
        renderItem={({ item, index }) => (
          <View key={item.id}>
            <Controller
              control={control}
              name={`contacts.${index}.name`}
              render={({ field: { value, onChange } }) => (
                <TextInput
                  value={value}
                  onChangeText={onChange}
                  placeholder="Name"
                />
              )}
            />

            <TouchableOpacity onPress={() => remove(index)}>
              <Text>Remove</Text>
            </TouchableOpacity>
          </View>
        )}
        keyExtractor={(_, index) => index.toString()}
      />

      <TouchableOpacity onPress={() => append({ name: "", email: "" })}>
        <Text>Add Contact</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 4: Watch Field Values

```javascript
import { useForm, Controller, useWatch } from "react-hook-form";

function ConditionalForm() {
  const { control } = useForm({
    defaultValues: { type: "personal" },
  });

  const type = useWatch({ control, name: "type" });

  return (
    <View>
      <Controller
        control={control}
        name="type"
        render={({ field: { value, onChange } }) => (
          <Picker selectedValue={value} onValueChange={onChange}>
            <Picker.Item label="Personal" value="personal" />
            <Picker.Item label="Business" value="business" />
          </Picker>
        )}
      />

      {type === "business" && (
        <Controller
          control={control}
          name="company"
          render={({ field: { value, onChange } }) => (
            <TextInput
              value={value}
              onChangeText={onChange}
              placeholder="Company Name"
            />
          )}
        />
      )}
    </View>
  );
}
```

### Example 5: Async Validation

```javascript
import { useForm, Controller } from "react-hook-form";

function AsyncValidationForm() {
  const { control, handleSubmit } = useForm({
    defaultValues: { username: "" },
  });

  const checkUsernameAvailable = async (username) => {
    const response = await fetch(`/api/check-username/${username}`);
    const { available } = await response.json();
    if (!available) throw new Error("Username taken");
  };

  return (
    <View>
      <Controller
        control={control}
        name="username"
        rules={{
          validate: checkUsernameAvailable,
        }}
        render={({ field: { value, onChange } }) => (
          <TextInput
            value={value}
            onChangeText={onChange}
            placeholder="Username"
          />
        )}
      />
    </View>
  );
}
```

---

## ❓ Interview Q&A

**Q1: What's the advantage of useForm over useState?**

A: Minimal re-renders, built-in validation, less boilerplate code.

```javascript
// With useState - re-renders entire form
const [form, setForm] = useState({ name: "", email: "" });

// With useForm - only affected field re-renders
const { control } = useForm();
```

**Q2: What's the difference between register and Controller?**

A: register is for native inputs, Controller is for custom components.

```javascript
// Native input
<input {...register('name')} />

// Custom component
<Controller name="name" control={control} render={...} />
```

**Q3: How do you validate fields?**

A: Use rules or resolver with Zod/Yup.

```javascript
// Inline rules
<Controller
  rules={{
    required: "Required",
    minLength: { value: 5, message: "Min 5 chars" },
  }}
/>;

// Resolver schema
const { control } = useForm({
  resolver: yupResolver(schema),
});
```

**Q4: How do you get all form values?**

A: Use getValues() or watch().

```javascript
const { getValues, watch } = useForm();

// Get all values
const allValues = getValues();

// Watch specific field
const email = watch("email");
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Using Controlled Components

```javascript
// BAD - causes excessive re-renders
const [value, setValue] = useState('');
<Controller
  render={({ field: { value, onChange } }) => (
    <TextInput value={value} onChangeText={onChange} />
  )}
/>

// GOOD - let form handle state
<Controller
  render={({ field: { onChange } }) => (
    <TextInput onChangeText={onChange} />
  )}
/>
```

### ❌ Mistake 2: Not Validating on Submit

```javascript
// BAD - form submits invalid data
handleSubmit((data) => submitForm(data));

// GOOD - add validation rules
<Controller rules={{ required: true }} render={...} />
```

---

## 📝 Practice Exercise

**Task:** Create a profile form with:

1. Name, email, phone fields
2. Validation
3. Submit button
4. Show errors
5. Display submitted data

**Solution:**

```javascript
import { useForm, Controller } from "react-hook-form";
import {
  View,
  TextInput,
  TouchableOpacity,
  Text,
  Alert,
  ScrollView,
} from "react-native";
import { Yup } from "yup";
import { yupResolver } from "@hookform/resolvers/yup";

const schema = Yup.object({
  name: Yup.string().required("Name required").min(2),
  email: Yup.string().required("Email required").email(),
  phone: Yup.string().required("Phone required").min(10),
});

function ProfileForm() {
  const {
    control,
    handleSubmit,
    formState: { errors },
    reset,
  } = useForm({
    resolver: yupResolver(schema),
    defaultValues: { name: "", email: "", phone: "" },
  });

  const onSubmit = (data) => {
    Alert.alert("Success", JSON.stringify(data));
    reset();
  };

  return (
    <ScrollView style={{ padding: 20 }}>
      <Controller
        control={control}
        name="name"
        render={({ field: { value, onChange } }) => (
          <View>
            <TextInput
              value={value}
              onChangeText={onChange}
              placeholder="Name"
              style={{ borderWidth: 1, padding: 10, marginBottom: 5 }}
            />
            {errors.name && (
              <Text style={{ color: "red" }}>{errors.name.message}</Text>
            )}
          </View>
        )}
      />

      <Controller
        control={control}
        name="email"
        render={({ field: { value, onChange } }) => (
          <View>
            <TextInput
              value={value}
              onChangeText={onChange}
              placeholder="Email"
              style={{ borderWidth: 1, padding: 10, marginBottom: 5 }}
            />
            {errors.email && (
              <Text style={{ color: "red" }}>{errors.email.message}</Text>
            )}
          </View>
        )}
      />

      <Controller
        control={control}
        name="phone"
        render={({ field: { value, onChange } }) => (
          <View>
            <TextInput
              value={value}
              onChangeText={onChange}
              placeholder="Phone"
              style={{ borderWidth: 1, padding: 10, marginBottom: 5 }}
            />
            {errors.phone && (
              <Text style={{ color: "red" }}>{errors.phone.message}</Text>
            )}
          </View>
        )}
      />

      <TouchableOpacity
        onPress={handleSubmit(onSubmit)}
        style={{ backgroundColor: "blue", padding: 10 }}
      >
        <Text style={{ color: "white" }}>Submit</Text>
      </TouchableOpacity>
    </ScrollView>
  );
}

export default ProfileForm;
```

---

## 🎓 Key Takeaways

✅ **Minimal re-renders** - only affected fields update  
✅ **Built-in validation** - sync and async  
✅ **Less boilerplate** - simpler than useState  
✅ **Error handling** - automatic field errors  
✅ **Dynamic fields** - useFieldArray for lists

---

**Next:** Learn about useSharedValue and useFieldArray for advanced form handling.
