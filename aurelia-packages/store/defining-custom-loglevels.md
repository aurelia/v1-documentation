# Defining custom loglevels

The Aurelia Store plugin provides various logging features to help you understand the flow of actions and state changes. You can customize the log levels used for different features using the `logDefinitions` option during plugin registration.

```typescript
// main.ts
import { Aurelia } from 'aurelia-framework';
import { initialState } from './state';

export function configure(aurelia: Aurelia) {
  aurelia.use.standardConfiguration().feature('resources');

  aurelia.use.plugin('aurelia-store', {
    initialState,
    logDispatchedActions: true, // Enable logging of dispatched actions
    logDefinitions: {
      dispatchedActions: 'debug', // Log dispatched actions with 'debug' level
      performanceLog: 'info', // Log performance info with 'info' level
      devToolsStatus: 'warn', // Log DevTools status with 'warn' level
    },
  });

  aurelia.start().then(() => aurelia.setRoot());
}
```

**Available Log Levels:**

You can use the following standard log levels:

* `'log'`
* `'debug'`
* `'info'`
* `'warn'`
* `'error'`

**Customizable Log Features:**

You can control the log levels for the following features:

* **`dispatchedActions`:** Logs information about dispatched actions (name, parameters).
* **`performanceLog`:** Logs performance measurements (if `measurePerformance` is enabled).
* **`devToolsStatus`:** Logs the status of the Redux DevTools integration (connected, disconnected).

**Example:**

In the example above, dispatched actions will be logged using `console.debug`, performance information will be logged using `console.info`, and DevTools status will be logged using `console.warn`.

{% hint style="info" %}
The `logDispatchedActions` option must be set to `true` to enable logging of dispatched actions, regardless of the `logDefinitions` setting.
{% endhint %}

