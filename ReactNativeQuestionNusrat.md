1. **What is the role of Metro Bundler in React Native?**  
Metro Bundler is like a packer. It takes all your JavaScript files, images, and code, then makes one bundle so the app can run. It also reloads fast when you change code.  
**Simple example:** When you save a file, Metro quickly updates the app screen.

2. **How do you implement platform-specific style for Android and iOS?**  
We can check which phone system is running and give different styles. React Native has `Platform` for this.  
**Simple example:** On iOS use more top padding, on Android use less.

```js
import { Platform } from 'react-native';

const styles = {
	paddingTop: Platform.OS === 'ios' ? 20 : 10,
};
git ad```

3. **How do you manage global themes in React Native?**  
Global theme means colors, fonts, and dark mode used in the whole app. Usually we keep theme data in `Context` or a state library.  
**Simple example:** One theme object has `primaryColor: 'blue'` and every screen uses it.

4. **How do you improve performance in React Native?**  
We keep the app light and fast. Do not render extra things, use `FlatList` for big lists, and avoid doing heavy work many times.  
**Simple example:** Instead of showing 1000 items at once, `FlatList` only shows what is on screen.

5. **How do you handle API pagination in a React Native app?**  
Pagination means loading data little by little. First load page 1, then load page 2 when the user scrolls more.  
**Simple example:** A product list loads 10 items first, then 10 more when reaching the bottom.

6. **How do you handle errors in React Native?**  
We use `try/catch` for API calls and show a friendly message to the user. We can also log the error for debugging.  
**Simple example:** If internet is off, show: `Something went wrong. Please try again.`

```js
try {
	const data = await getUsers();
} catch (error) {
	setError('Please try again');
}
```

7. **What is the difference between Promise and Async/Await?**  
Both are used for async work. Promise uses `.then()` and `.catch()`. `async/await` is just a cleaner, easier way to write the same thing.  
**Simple example:** `await fetchData()` looks simpler than `fetchData().then(...)`.

8. **How do you build a React Native app for iOS?**  
Usually we install pods, open Xcode, choose device or simulator, then build the app. For release, we archive it in Xcode.  
**Simple example:** Run `npx pod-install`, open the `ios` project in Xcode, then press build.

9. **Which workspace is usually opened for iOS work: general workspace or Expo workspace?**  
If it is a normal React Native app, we usually open the Xcode workspace from the `ios` folder, often `YourApp.xcworkspace`. If it is Expo managed, most work is done with Expo tools unless native iOS code is needed.  
**Simple example:** Normal RN app -> open Xcode workspace. Expo app -> mostly use Expo CLI.

10. **Have you implemented analytics in React Native or Firebase Console?**  
Yes, analytics is used to track user actions like button press, screen open, or purchase. Firebase Analytics is a common tool for this.  
**Simple example:** Track when a user opens the Home screen.

```js
analytics().logScreenView({ screen_name: 'Home' });
```

11. **Have you used any state management library?**  
Yes. Common ones are Context API, Redux Toolkit, Zustand, and React Query. We choose based on app size and need.  
**Simple example:** Redux Toolkit can store login user data for the whole app.

12. **What is the difference between global and local state management in React Native?**  
Local state is used only inside one component. Global state is shared across many screens or components.  
**Simple example:** Input text in one form is local state. Logged-in user info is global state.

13. **How do you avoid unnecessary re-render in React Native?**  
Use `React.memo`, keep state small, and avoid creating new functions or objects again and again if not needed.  
**Simple example:** A list item should not re-render when another item changes.

14. **How do you manage a large form with many fields in React Native? Give steps too.**  
For big forms, use one form library like `react-hook-form` or manage fields in one state object. Validation should also be clear.  
**Steps:**  
1. Create form state.  
2. Add fields like name, email, password.  
3. Validate each field.  
4. Show error message under wrong field.  
5. Submit data only when form is valid.  
**Simple example:** Signup form with name, email, and password.

15. **How do you implement code splitting in React Native?**  
Code splitting means loading some code only when needed, so the app feels lighter. In React Native, this is often done with lazy loading screens or features.  
**Simple example:** Load the Profile screen only when the user opens it.

16. **What are some best practices for writing maintainable React Native code?**  
Keep files clean, use small reusable components, choose good names, and separate UI from business logic. Also keep one coding style in the project.  
**Simple example:** Put API code in a service file instead of writing it on every screen.

17. **How can you implement localization in a React Native app for multiple languages?**  
Localization means showing the app in different languages. We keep text in language files like English and Bangla, then show the right one based on user choice or device language.  
**Simple example:** `hello` becomes `Hello` in English and `হ্যালো` in Bangla.