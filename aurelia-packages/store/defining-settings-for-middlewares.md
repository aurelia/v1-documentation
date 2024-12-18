# Defining settings for middlewares

Some middleware functions might require configuration options to customize their behavior. The Aurelia Store plugin allows you to pass settings to middleware through the third argument of the middleware function and when registering the middleware using `registerMiddleware`.

**Signature:**

```typescript
(currentState: T, originalState?: T, settings?: any, action?: { name: string; params: any[] }) =>
  | Promise<T | undefined | void>
  | T
  | undefined
  | void;
```

**`settings?: any`:** An optional object that can contain any configuration data you want to pass to the middleware.

**Example: `customLogMiddleware` with Settings**

Let's extend the `customLogMiddleware` from previous examples to accept a `logType` setting that determines which `console` method to use (e.g., `log`, `debug`, `warn`, `error`).

```typescript
// app.ts
import { inject, PLAT কোন্‌ } from 'aurelia-framework';
import { Store, MiddlewarePlacement, StateHistory } from 'aurelia-store';
import { State } from './state'; // Import your State interface

interface LogMiddlewareSettings {
  logType: 'log' | 'debug' | 'warn' | 'error';
}

const customLogMiddleware = (
  currentState: StateHistory<State>,
  originalState: StateHistory<State>,
  settings: LogMiddlewareSettings
) => {
  if (settings && settings.logType) {
    console[settings.logType](
      'Current state:',
      currentState,
      'Original state:',
      originalState
    );
  } else {
    console.log('Current state:', currentState, 'Original state:', originalState);
  }
};

@inject(Store)
export class App {
  constructor(private store: Store<StateHistory<State>>) {
    this.store.registerMiddleware(
      customLogMiddleware,
      PLATFORM.moduleName('after'),
      { logType: 'debug' }
    );
  }
}
```

**Explanation:**

1. **`interface LogMiddlewareSettings`:** We define an interface to strongly type the settings object.
2. **`console[settings.logType](...)`:** We use bracket notation to dynamically access the appropriate `console` method based on the `logType` setting.
3. **`{ logType: 'debug' }`:** When registering the middleware, we pass an object with the `logType` set to `'debug'`.

Now, the `customLogMiddleware` will use `console.debug` to log the state information.

Reference to the Calling Action for Middlewares

In some cases, you might need to know which action is being dispatched within a middleware function. This can be useful for filtering actions, modifying behavior based on the action type, or logging action-specific information. The Aurelia Store plugin provides this information through the fourth argument of the middleware function.

**Signature:**

```typescript
(currentState: T, originalState?: T, settings?: any, action?: { name: string; params: any[] }) =>
  | Promise<T | undefined | void>
  | T
  | undefined
  | void;
```

**`action?: { name: string; params: any[] }`:** An object containing information about the dispatched action:

* **`name: string`:** The name of the action (either the function name or the name provided during registration).
* **`params: any[]`:** An array of the parameters that were passed to the `dispatch` method.

**Example: `gateKeeperMiddleware`**

Let's say you want to implement a middleware that prevents certain actions from being executed if a specific condition is met (e.g., a `lockActive` flag in the state).

```typescript
// app.ts
import { inject, PLATFORM } from 'aurelia-framework';
import { Store, MiddlewarePlacement, StateHistory } from 'aurelia-store';
import { State } from './state';

const gateKeeperMiddleware = (
  currentState: StateHistory<State>,
  originalState: StateHistory<State>,
  _: unknown, // settings is not used, we can use any placeholder
  action: { name: string; params: any[] }
) => {
  if (currentState.present.lockActive && action.name === 'trespasserAction') {
    console.warn('Action', action.name, 'is not allowed when lock is active.');
    return originalState; // Block the action by returning the original state
  }

  return currentState; // Allow the action to proceed
};

@inject(Store)
export class App {
  constructor(private store: Store<StateHistory<State>>) {
    this.store.registerMiddleware(gateKeeperMiddleware, PLATFORM.moduleName('before'));
  }
}
```

**Explanation:**

1. **`if (currentState.present.lockActive && action.name === 'trespasserAction')`:** We check if the `lockActive` flag in the state is `true` and if the dispatched action's name is `'trespasserAction'`.
2. **`return originalState;`:** If both conditions are met, we block the action by returning the `originalState`, preventing any further modifications.
3. **`return currentState;`:** If the conditions are not met, we allow the action to proceed by returning the `currentState`.

### Piped Actions

When dealing with piped actions (using `store.pipe`), the `action` object in middleware will have the following characteristics:

* **`action.name`:** Will be a string representing the piped actions in the format `'actionA->actionB->actionC'`.
* **`action.params`:** Will be a single array containing all the parameters from all the piped actions.
* **`action.pipedActions`:** An array of objects, each representing a piped action with its individual `name` and `params`.

**Example:**

```typescript
const pipedMiddleware = (
  currentState: StateHistory<State>,
  originalState: StateHistory<State>,
  _: unknown,
  action: { name: string; params: any[]; pipedActions: { name: string; params: any[] }[] }
) => {
  console.log('Piped action name:', action.name); // e.g., "actionA->actionB"
  console.log('All params:', action.params); // e.g., [1, "hello", true]
  console.log('Individual actions:', action.pipedActions);
  // e.g., [{ name: "actionA", params: [1] }, { name: "actionB", params: ["hello", true] }]

  return currentState;
};
```
