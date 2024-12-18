# Configuring your app

To use the Aurelia Store plugin, you need to register it in your application's main configuration file (typically `main.ts` or `main.js`).

Here's how you do it:

```typescript
// main.ts
import { Aurelia } from 'aurelia-framework';
import { initialState } from './state'; // Import your initial state

export function configure(aurelia: Aurelia) {
  aurelia.use
    .standardConfiguration()
    .feature('resources'); // Assuming you have custom resources in a 'resources' folder

  // ... other configuration ...

  aurelia.use.plugin('aurelia-store', { initialState }); // Register the plugin and provide the initial state

  aurelia.start().then(() => aurelia.setRoot());
}
```

**Explanation:**

1. **`import { initialState } from './state';`:** Import the `initialState` object that you defined earlier.
2. **`aurelia.use.plugin('aurelia-store', { initialState });`:**
   * This line registers the Aurelia Store plugin with your Aurelia application.
   * The `{ initialState }` object provides the initial state to the store. This is the starting point for your application's state.

With this configuration in place, the Aurelia Store plugin is ready to use, and your application's state will be initialized with the `initialState` you provided.
