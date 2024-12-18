# Error Propagation with Middlewares

By default, if an error occurs within a middleware function, the Aurelia Store plugin will swallow the error to prevent it from disrupting the application flow and maintain a continued state stream. However, in some cases, you might want to explicitly handle errors or stop the propagation of the state in case of an error.

**`propagateError` Option:**

You can control error propagation behavior using the `propagateError` option during plugin registration:

```typescript
// main.ts
aurelia.use.plugin('aurelia-store', {
  initialState,
  propagateError: true, // Enable error propagation
});
```

**`propagateError: true`:** When set to `true`, errors thrown within middleware will be re-thrown, potentially causing the application to crash if not handled properly. This can be useful during development for identifying and fixing errors in middleware.

**`propagateError: false` (default):** When set to `false` (or not specified), errors in middleware will be caught and logged to the console, but they will not stop the execution of the middleware chain or the updating of the store. The state propagation continues, providing the last successfully created state to the next middleware in the chain.

**Example:**

```typescript
// app.ts
import { inject, PLATFORM } from 'aurelia-framework';
import { Store, MiddlewarePlacement, StateHistory } from 'aurelia-store';
import { State } from './state';

const errorThrowingMiddleware = (currentState: StateHistory<State>) => {
  throw new Error('Something went wrong in the middleware!');
};

@inject(Store)
export class App {
  constructor(private store: Store<StateHistory<State>>) {
    this.store.registerMiddleware(
      errorThrowingMiddleware, 
      PLATFORM.moduleName('before')
    );
  }
}
```

**With `propagateError: false` (default):**

* The error will be logged to the console: `Error: Something went wrong in the middleware!`.
* The application will continue to run.
* The middleware chain will continue, and the next middleware will receive the state as it was before the error occurred.

**With `propagateError: true`:**

* The error will be logged to the console: `Error: Something went wrong in the middleware!`.
* The error will be re-thrown, potentially causing the application to crash if not handled by a `try...catch` block.
* The middleware chain execution will stop.

**Recommendation:**

* During development, it can be helpful to set `propagateError` to `true` to identify and fix errors in your middleware quickly.
* In production, it's generally recommended to keep `propagateError` set to `false` (or not specify it) to prevent unexpected crashes and provide a smoother user experience. You should handle errors gracefully within your middleware or use other error-handling mechanisms to deal with potential issues.
