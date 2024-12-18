# Recording a navigable history of the stream of states

The Aurelia Store plugin provides a powerful feature for recording and navigating through the history of state changes in your application. This can be incredibly useful for debugging, implementing undo/redo functionality, and gaining insights into how your application's state evolves over time.

To enable history tracking, you need to configure it during the plugin registration in your `main.ts` file:

```typescript
// main.ts
import { Aurelia } from 'aurelia-framework';
import { initialState } from './state';

export function configure(aurelia: Aurelia) {
  aurelia.use
    .standardConfiguration()
    .feature('resources');

  // ... other configuration ...

  aurelia.use.plugin('aurelia-store', {
    initialState,
    history: {
      undoable: true, // Enable history tracking
    },
  });

  aurelia.start().then(() => aurelia.setRoot());
}
```

**Explanation:**

* **`history: { undoable: true }`:** This option tells the Aurelia Store plugin to start recording the history of state changes.

### The `StateHistory<T>` Object

When history is enabled, the `state` property of the store (which you subscribe to) will no longer emit simple `State` objects. Instead, it will emit `StateHistory<State>` objects. The `StateHistory` interface looks like this:

```typescript
// aurelia-store -> history.ts
export interface StateHistory<T> {
  past: T[];
  present: T;
  future: T[];
}
```

**Explanation:**

* **`past: T[]`:** An array of previous states, representing the history of state changes.
* **`present: T`:** The current state of the application.
* **`future: T[]`:** An array of states that have been undone (using the `jump` action, which we'll discuss later). These states can be "redone" by moving them back to the `present`.

**Example:**

```typescript
// app.ts
import { inject } from 'aurelia-framework';
import { Store, StateHistory } from 'aurelia-store';
import { State } from './state'; // Import your State interface

@inject(Store)
export class App {
  private state: StateHistory<State>;

  constructor(private store: Store<StateHistory<State>>) {
      this.store.state.subscribe(
        (state: StateHistory<State>) => (this.state = state)
      );
  }
}
```

In this example, the `state` property of the `App` component will now hold a `StateHistory<State>` object, giving you access to the `past`, `present`, and `future` states.
