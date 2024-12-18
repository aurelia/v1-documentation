# Navigating through history

The Aurelia Store plugin provides a built-in action called `jump` that allows you to navigate through the state history. This is the foundation for implementing undo/redo functionality or time-traveling debugging.

```typescript
// app.ts
import { inject } from 'aurelia-framework';
import { Store, jump, StateHistory } from 'aurelia-store';
import { State } from './state';

@inject(Store)
export class App {
  private state: StateHistory<State>;

  constructor(private store: Store<StateHistory<State>>) {
      this.store.state.subscribe(
        (state: StateHistory<State>) => (this.state = state)
      );
  }

  undo() {
    this.store.dispatch(jump, -1); // Go back one step in history
  }

  redo() {
    this.store.dispatch(jump, 1); // Go forward one step in history
  }
}
```

**Explanation:**

* **`this.store.dispatch(jump, -1);`:** Dispatches the `jump` action with a negative number to move backward in history. `-1` moves to the previous state, `-2` moves two steps back, and so on.
* **`this.store.dispatch(jump, 1);`:** Dispatches the `jump` action with a positive number to move forward in history. `1` moves to the next state (if available), `2` moves two steps forward, and so on.

**Overflow Handling:**

The `jump` action automatically handles overflows. If you try to go beyond the beginning or end of the history, it will simply return the current `StateHistory` object without changing the `present` state.

### Limiting the Number of History Items

Storing the entire history of state changes can consume a significant amount of memory, especially in long-running applications. To prevent this, you can limit the number of history items that are stored in the `past` and `future` arrays.

You can configure the history limit during plugin registration:

```typescript
// main.ts
aurelia.use.plugin('aurelia-store', {
  initialState,
  history: {
    undoable: true,
    limit: 10, // Limit history to 10 items
  },
});
```

**Explanation:**

* **`limit: 10`:** This option sets the maximum number of items that will be stored in the `past` and `future` arrays.

**Overflow Behavior:**

When the history limit is reached:

* **Adding new states to `past`:** The oldest state in the `past` array will be removed to make room for the new state.
* **Adding new states to `future`:** The most recently undone state in the `future` array will be removed.

This ensures that the history never exceeds the specified limit.
