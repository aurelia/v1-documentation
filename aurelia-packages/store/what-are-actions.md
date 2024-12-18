# What are actions?

Actions are the only way to modify the application's state in the Aurelia Store plugin. They are essentially functions that take the current state and optionally some arguments, and return a new state.

### Immutability

It is crucial that actions **do not mutate** the existing state object. Instead, they should create a new state object with the desired changes. This ensures that the state remains predictable and that we can easily track changes over time.

### Example Action

Let's create an action that adds a new framework to the `frameworks` array in our state:

```typescript
// actions.ts
import { State } from './state';

export function addFramework(currentState: State, newFramework: string): State {
  return {
    ...currentState, // Copy the existing state
    frameworks: [...currentState.frameworks, newFramework] // Create a new array with the added framework
  };
}
```

**Explanation:**

1. **`export function addFramework(...)`**: Defines a function called `addFramework` that takes two arguments:
   * `currentState: State`: The current state object.
   * `newFramework: string`: The name of the new framework to add.
2. **`return { ...currentState, ... };`**: This uses the object spread syntax to create a new object that is a copy of the `currentState`.
3. **`frameworks: [...currentState.frameworks, newFramework]`**: This creates a new array that contains all the elements of the existing `frameworks` array, plus the `newFramework` appended to the end. This ensures that we don't modify the original `frameworks` array.

### Registering Actions

Before you can dispatch an action, you need to register it with the store using the `store.registerAction` method. This is typically done in the constructor of your component or service where you want to use the action:

```typescript
// app.ts
import { inject } from 'aurelia-dependency-injection';
import { Store } from 'aurelia-store';
import { State } from './state';
import { addFramework } from './actions';

@inject(Store)
export class App {
  constructor(private store: Store<State>) {
    this.store.registerAction('AddFramework', addFramework);
  }

  // ...
}
```

**Explanation:**

1. **`import { addFramework } from './actions';`**: Imports the `addFramework` action.
2. **`this.store.registerAction('AddFramework', addFramework);`**:
   * `'AddFramework'` is a string identifier for the action. This is useful for debugging and logging.
   * `addFramework` is the actual action function.

### Unregistering Actions

You can unregister an action using the `store.unregisterAction` method:

```typescript
this.store.unregisterAction(addFramework);
```

This removes the action from the store's registry.
