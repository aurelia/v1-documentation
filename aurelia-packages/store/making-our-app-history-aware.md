# Making our app history-aware

To properly update the state history when dispatching actions, you need to make your actions "history-aware." This means that instead of returning a new `State` object, your actions should return a new `StateHistory<State>` object.

The Aurelia Store plugin provides a helper function called `nextStateHistory` to simplify this process:

```typescript
// actions.ts
import { nextStateHistory, StateHistory } from 'aurelia-store';
import { State } from './state';

export const addFramework = (
  currentState: StateHistory<State>,
  frameworkName: string
): StateHistory<State> => {
  const newPresentState: State = {
    ...currentState.present,
    frameworks: [...currentState.present.frameworks, frameworkName],
  };

  return nextStateHistory(currentState, newPresentState);
};
```

**Explanation:**

1. **`currentState: StateHistory<State>`:** The action now receives the entire `StateHistory` object as its first argument.
2. **`newPresentState: State`:** We create a new `State` object representing the new present state, as we would normally do in a non-history-aware action.
3. **`return nextStateHistory(currentState, newPresentState);`:** We use the `nextStateHistory` helper function to create a new `StateHistory` object. This function does the following:
   * Moves the current `present` state to the `past` array.
   * Sets the `newPresentState` as the new `present` state.
   * Clears the `future` array.

**Manual Approach (not recommended):**

You could manually create the `StateHistory` object, but it's more verbose and error-prone:

```typescript
// actions.ts (manual approach - not recommended)
import { StateHistory } from 'aurelia-store';
import { State } from './state';

export const addFramework = (
  currentState: StateHistory<State>,
  frameworkName: string
): StateHistory<State> => {
  return {
    past: [...currentState.past, currentState.present],
    present: {
      ...currentState.present,
      frameworks: [...currentState.present.frameworks, frameworkName],
    },
    future: [],
  };
};
```

It's highly recommended to use the `nextStateHistory` helper function to ensure that the history is updated correctly.
