# Async actions

In real-world applications, many actions involve asynchronous operations, such as fetching data from a server, performing calculations that take time, or interacting with external services. The Aurelia Store plugin seamlessly handles asynchronous actions by allowing actions to return promises that resolve with the new state.

### Returning Promises

An action can return a promise that, when resolved, provides the updated state. This allows you to perform asynchronous tasks within your actions and update the state once those tasks are complete.

### Example: Async Validation

Let's say you have an action that adds a new framework to the list, but you need to validate the framework name asynchronously before adding it. Here's how you might implement such an action:

```typescript
// actions.ts
import { State } from './state';

function validateFrameworkName(name: string): Promise<string> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (name.toLowerCase() === 'angularjs') {
        reject(new Error('Please use a more modern framework!'));
      } else {
        resolve(name);
      }
    }, 500); // Simulate a 500ms delay
  });
}

export async function addFramework(state: State, newFramework: string): Promise<State> {
  const validatedName = await validateFrameworkName(newFramework);
  const newState = { ...state }; // Create a shallow copy of the state
  newState.frameworks = [...newState.frameworks, validatedName]; // Update the frameworks array
  return newState;
}
```

**Explanation:**

1. **`validateFrameworkName(name: string): Promise<string>`:** This function simulates an asynchronous validation process. It returns a promise that resolves with the validated name if it's valid or rejects with an error if it's not.
2. **`async function addFramework(state: State, newFramework: string): Promise<State>`:**
   * The `async` keyword indicates that this function is asynchronous and will return a promise.
   * `await validateFrameworkName(newFramework)`: This line awaits the result of the `validateFrameworkName` promise. The execution of the `addFramework` action will pause until the promise resolves or rejects.
   * The rest of the action creates a new state object with the validated framework added to the list, similar to a synchronous action.
   * `return newState;`: The function returns the updated state. Because it's an `async` function, this is equivalent to returning `Promise.resolve(newState)`.

### Using `async/await`

The example above uses `async/await`, which is a more readable way of working with promises in JavaScript. It's highly recommended to use `async/await` whenever possible for better code clarity. However, you can also use traditional promise chaining with `.then()` if you prefer.
