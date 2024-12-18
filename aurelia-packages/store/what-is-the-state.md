# What is the State?

The state is a central concept in the Aurelia Store plugin. It represents the entire state of your application at a specific point in time. Think of it as a large JavaScript object that contains all the data and status information that defines your application's current condition.

### Components of the State

A typical application state consists of:

* **Data:** This includes the data that is displayed in your application's UI, such as lists of products, user information, or any other data that your application works with.
* **Status:** This includes various status flags and indicators that reflect the current state of the application, such as whether a modal is open, the currently selected theme, the active page, or the loading state of a component.

### Serializable Properties

It is crucial that your state object only contains **serializable** properties. This means that the properties should be of types that can be easily converted to and from a JSON string, such as:

* Numbers
* Strings
* Booleans
* Arrays
* Objects (with serializable properties)

### **Why is serializability important?**

* **Snapshots:** It allows you to easily create snapshots of your application's state, which can be useful for debugging, undo/redo functionality, and other advanced features.
* **Time-Traveling:** It enables time-traveling debugging, where you can step through different states of your application to see how it evolved over time.
* **Persistence:** It allows you to easily persist the state to storage, such as local storage, and rehydrate it later.

### What to Store in the State

The general rule of thumb is to store data in the state if:

* **Multiple components need access to the same data:** Instead of passing data around between components or using complex service interactions, you can store the data in the state, and all components can access it from there.
* **The data affects the state of multiple components:** If a piece of data or a status flag affects the behavior or appearance of multiple components, it should be stored in the state.

### Example: `initialState`

Here's an example of how you might define an initial state for your application:

```typescript
// state.ts
export interface State {
  frameworks: string[];
  user: {
    firstName: string;
    lastName: string;
  } | null;
  isLoading: boolean;
}

export const initialState: State = {
  frameworks: ['Aurelia', 'React', 'Vue'],
  user: null,
  isLoading: false,
};
```

In this example, the state includes:

* `frameworks`: An array of strings representing a list of frameworks.
* `user`: An object representing the currently logged-in user, or `null` if no user is logged in.
* `isLoading`: A boolean flag indicating whether the application is currently loading data.
