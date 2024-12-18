# Accessing the original (unmodified) state in a middleware

You might need to access the original, unmodified state within a middleware function in certain scenarios. This is the state before any middleware in the chain has had a chance to modify it. The Aurelia Store plugin provides this capability through the second argument of the middleware function.

**Signature:**

```typescript
(currentState: T, originalState?: T, settings?: any, action?: { name: string; params: any[] }) =>
  | Promise<T | undefined | void>
  | T
  | undefined
  | void;
```

**`originalState: T`:** This argument represents the state before any middleware modifications.

**Use Cases:**

* **Determining State Diff:** You can compare `currentState` with `originalState` to see what changes have been made by previous middleware functions in the chain.
* **Resetting to Original State:** In certain conditions, you might want to discard the changes made by previous middleware and revert to the original state.
* **Conditional Logic:** You can use the `originalState` to make decisions about how the middleware should behave based on the initial state of the application.

**Example: "Blacklister" Middleware**

Let's say you want to implement a middleware that prevents actions from modifying the state if it contains certain blacklisted words.

```typescript
// app.ts
import { inject, PLATFORM } from 'aurelia-framework';
import { Store, MiddlewarePlacement, StateHistory } from 'aurelia-store';
import { State } from './state'; // Import your State interface

const blacklisterMiddleware = (
  currentState: StateHistory<State>,
  originalState: StateHistory<State>
) => {
  const isBlacklisted = (text: string) => {
    const blacklist = ['f**k', 's**t', 'etc']; // Your blacklist
    return blacklist.some((word) => text.includes(word));
  };

  if (isBlacklisted(currentState.present.someTextProperty)) {
    console.warn('Blacklisted word detected! Reverting to original state.');
    return originalState; // Revert to the original state
  }

  return currentState; // Otherwise, pass the current state along
};

@inject(Store)
export class App {
  constructor(private store: Store<StateHistory<State>>) {
    this.store.registerMiddleware(blacklisterMiddleware, PLATFORM.moduleName('before'));
  }
}
```

**Explanation:**

1. **`isBlacklisted` Function:** A helper function that checks if a given text contains any blacklisted words.
2. **`if (isBlacklisted(currentState.present.someTextProperty))`:** We check if the relevant property in the `currentState` contains any blacklisted words.
3. **`return originalState;`:** If a blacklisted word is found, we return the `originalState`, effectively discarding any changes made by previous middleware or the action itself.
4. **`return currentState;`:** If no blacklisted words are found, we return the `currentState`, allowing the middleware chain to continue.
