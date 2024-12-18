# Handling side-effects with middleware

Middleware provides a powerful mechanism for handling side-effects in your Aurelia Store applications. Middleware functions are executed before or after each dispatched action, allowing you to intercept and modify the state, perform asynchronous operations, log actions, or integrate with external services.

**Concept:**

Middleware in Aurelia Store is similar to middleware in other frameworks like Express.js. It allows you to create a chain of functions that are executed in a specific order, with each function having the opportunity to modify the state or perform other actions.

**Middleware Function Signature:**

A middleware function has the following signature:

```typescript
(currentState: T, originalState?: T, settings?: any, action?: { name: string; params: any[] }) =>
  | Promise<T | undefined | void>
  | T
  | undefined
  | void;
```

**Arguments:**

* **`currentState: T`:** The current state before the action is applied (or the result of the previous middleware in the chain).
* **`originalState?: T`:** The state before any middleware was applied. This is useful for comparing the state before and after the middleware chain or resetting to the original state in specific cases.
* **`settings?: any`:** An optional object that can be used to pass configuration settings to the middleware.
* **`action?: { name: string; params: any[] }`:** An object containing the name and parameters of the dispatched action. This is useful for filtering or modifying behavior based on the specific action being dispatched.

**Return Value:**

* **`T` or `Promise<T>`:** A new state object, which will be passed to the next middleware in the chain or used to update the store if it's the last middleware.
* **`undefined` or `void`:** If the middleware returns `undefined` or nothing (void), the previous state (`currentState`) will be passed to the next middleware or used to update the store.
* **`Promise<void>`** If the middleware returns a Promise that resolves with `undefined` or nothing (void), the previous state will be passed on as the resolved value.

**Registration:**

You register middleware using the `store.registerMiddleware` method:

```typescript
store.registerMiddleware(middlewareFunction, MiddlewarePlacement, settings?);
```

**Arguments:**

* **`middlewareFunction`:** The middleware function to register.
* **`MiddlewarePlacement`:** An enum that specifies whether the middleware should be executed `before` or `after` the action.
* **`settings?`:** An optional object containing settings for the middleware.

**Unregistration:**

You can unregister middleware using the `store.unregisterMiddleware` method:

```typescript
store.unregisterMiddleware(middlewareFunction);
```

**Example:**

```typescript
// app.ts
import { inject, PLATFORM } from 'aurelia-framework';
import { Store, MiddlewarePlacement, logMiddleware, StateHistory } from 'aurelia-store';
import { State } from './state';

const customMiddleware = (
    currentState: StateHistory<State>, 
    originalState: StateHistory<State>, 
    settings: any, 
    action: any
) => {
  console.log('Custom middleware executed!');
  console.log('Current state:', currentState);
  console.log('Original state:', originalState);
  console.log('Settings:', settings);
  console.log('Action:', action);

  // Modify the state or perform side-effects here

  return currentState; // Pass the current state to the next middleware
};

@inject(Store)
export class App {
  constructor(private store: Store<StateHistory<State>>) {
    this.store.registerMiddleware(
      customMiddleware, 
      PLATFORM.moduleName('before'), 
      { logType: 'debug' }
    );
    this.store.registerMiddleware(logMiddleware, PLATFORM.moduleName('after'), {
      logType: 'log',
    });
  }
}
```

**Asynchronous Middleware:**

Middleware can also be asynchronous. If a middleware function returns a `Promise`, the Aurelia Store plugin will wait for the `Promise` to resolve before proceeding to the next middleware or updating the store.

**Warning:** As soon as you have one async middleware registered, all action dispatches will essentially be async as well.

**Key Use Cases for Middleware:**

* **Logging:** Log actions, state changes, or other debugging information.
* **Asynchronous Operations:** Perform API calls, database operations, or other asynchronous tasks before or after an action.
* **Authentication/Authorization:** Check user permissions or authentication status before allowing an action to proceed.
* **State Validation:** Validate the state before or after an action to ensure data integrity.
* **Integration with External Services:** Interact with third-party services or libraries.

Middleware provides a flexible and powerful way to extend the functionality of the Aurelia Store plugin and handle various side effects in your application.
