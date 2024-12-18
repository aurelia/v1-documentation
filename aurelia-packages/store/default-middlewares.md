# Default middlewares

The Aurelia Store plugin comes with a few built-in middleware functions that provide common functionality you might need in your applications. You can use these middleware functions directly without having to implement them yourself.

### The Logging Middleware (`logMiddleware`)

The `logMiddleware` is a simple middleware that logs the current state, original state, and dispatched action to the console. It's beneficial for debugging and understanding how your application's state changes over time.

**Usage:**

```typescript
// app.ts
import { inject, PLATFORM } from 'aurelia-framework';
import { Store, logMiddleware, MiddlewarePlacement, StateHistory } from 'aurelia-store';
import { State } from './state';

@inject(Store)
export class App {
  constructor(private store: Store<StateHistory<State>>) {
    this.store.registerMiddleware(
      logMiddleware,
      PLATFORM.moduleName('after'),
      { logType: 'log' }
    );
  }
}
```

**Settings:**

The `logMiddleware` accepts a settings object with a `logType` property, similar to the `customLogMiddleware` example we discussed earlier. You can set `logType` to one of the following:

* `'log'` (default)
* `'debug'`
* `'info'`
* `'warn'`
* `'error'`

This determines which `console` method will be used for logging.

### The Local Storage Middleware (`localStorageMiddleware`)

The `localStorageMiddleware` automatically saves the latest emitted state to the browser's `localStorage`. This allows you to persist the application's state across page refreshes or browser sessions.

**Usage:**

```typescript
// app.ts
import { inject, PLATFORM } from 'aurelia-framework';
import {
  Store,
  localStorageMiddleware,
  rehydrateFromLocalStorage,
  MiddlewarePlacement,
  StateHistory
} from 'aurelia-store';
import { State } from './state';

@inject(Store)
export class App {
  constructor(private store: Store<StateHistory<State>>) {
    // Register the middleware AFTER the initial state has been loaded
    this.store.registerAction('Rehydrate', rehydrateFromLocalStorage);

    this.store.registerMiddleware(
      localStorageMiddleware,
      PLATFORM.moduleName('after'),
      { key: 'my-aurelia-store-state' }
    );
    this.store.dispatch(rehydrateFromLocalStorage);
  }
}
```

**Settings:**

The `localStorageMiddleware` accepts a settings object with a `key` property. This determines the key that will be used to store the state in `localStorage`. The default key is `'aurelia-store-state'`.

**Rehydrating from Local Storage:**

To load the saved state from `localStorage`, you need to dispatch the `rehydrateFromLocalStorage` action. This action retrieves the state from `localStorage` and updates the store with it.

**Important Note:** It's crucial to register the `localStorageMiddleware` **after** the initial state has been loaded. Otherwise, the initial state will immediately overwrite any previously saved state in `localStorage` on the first page load. Register the middleware just after the state has been rehydrated as shown in the example above.

### 26. Execution Order

When multiple actions are dispatched, or you have a chain of middleware functions, it's essential to understand the order in which they are executed.

**Action Queue:**

Dispatched actions are queued and executed one after another. This ensures that each action receives the most up-to-date state resulting from the previous action.

**Middleware Chain:**

Middleware functions are executed in the order they are registered, based on their `MiddlewarePlacement` (`before` or `after`):

1. All `before` middleware functions are executed in registration order.
2. The dispatched action is executed.
3. All `after` middleware functions are executed in registration order.

**Interrupting Execution:**

If an action or middleware returns `false` (either synchronously or by resolving a `Promise` with `false`), the execution of the current dispatch cycle will be interrupted. This means:

* The store's state will **not** be updated.
* Any subsequent middleware functions in the chain will **not** be executed.
* If an action is piped, none of the actions in the pipe after the interruptor will be executed

This behavior can be useful for preventing state updates or middleware execution based on certain conditions.

**Asynchronous Considerations:**

If any middleware in the chain is asynchronous (i.e., returns a `Promise`), the entire dispatch process becomes asynchronous. The Aurelia Store plugin will wait for each `Promise` to resolve before proceeding to the next middleware or updating the store.
