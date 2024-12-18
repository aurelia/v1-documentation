# Defining custom devToolsOptions

The Aurelia Store plugin seamlessly integrates with the Redux DevTools browser extension, providing a powerful way to debug and visualize your application's state changes. You can customize the behavior of the Redux DevTools integration by passing a `devToolsOptions` object during plugin registration.

```typescript
// main.ts
import { Aurelia } from 'aurelia-framework';
import { initialState } from './state';

export function configure(aurelia: Aurelia) {
  aurelia.use.standardConfiguration().feature('resources');

  aurelia.use.plugin('aurelia-store', {
    initialState,
    devToolsOptions: {
      // Your custom options here
      serialize: false, // Example: Disable state serialization
    },
  });

  aurelia.start().then(() => aurelia.setRoot());
}
```

**Available Options:**

The `devToolsOptions` object accepts the same options as the `redux-devtools-extension` itself. You can find the complete list of options in the [official documentation](https://github.com/zalmoxisus/redux-devtools-extension/blob/master/docs/API/Arguments.md).

**Example: Disabling State Serialization**

By default, the Redux DevTools extension serializes the state before sending it to the extension. This can be problematic if your state contains non-serializable objects (e.g., functions, circular references). You can disable serialization by setting the `serialize` option to `false`:

```typescript
devToolsOptions: {
  serialize: false,
},
```

**Other Common Options:**

* **`name`:** Sets the instance name in the Redux DevTools. Useful if you have multiple stores or applications on the same page.
* **`maxAge`:** Specifies the maximum number of actions to hold in the history (similar to the `limit` option for the built-in history).
* **`latency`:** Sets the maximum delay between a state change and the DevTools update.
