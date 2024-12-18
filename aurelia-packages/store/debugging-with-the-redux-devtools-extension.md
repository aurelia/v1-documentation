# Debugging with the Redux devtools extension

The Redux DevTools Extension is a powerful browser extension that provides a visual interface for inspecting and debugging the state of your Redux or Redux-like applications, including those using the Aurelia Store plugin.

**Benefits:**

* **State Inspection:** View the current state of your application, as well as the history of state changes.
* **Action Replay:** Replay actions to see how they affected the state.
* **Time-Travel Debugging:** Step backward and forward through the state history to understand how the application reached a particular state.
* **Dispatch Actions:** Dispatch actions directly from the DevTools interface.

**Integration:**

The Aurelia Store plugin automatically integrates with the Redux DevTools Extension if it's installed in your browser. No additional configuration is required in most cases.

**Enabling Redux Devtools:** To enable the Redux Devtools, you need to set the `devToolsOptions` to `true` during the plugin registration.

```js
aurelia.use.plugin('aurelia-store', {
    initialState,
    devToolsOptions: true
  });
```

**Usage:**

1. Install the Redux DevTools Extension for your browser (Chrome, Firefox, etc.).
2. Enable `devToolsOptions` during plugin registration.
3. Open your Aurelia application.
4. Open the browser's developer tools and select the "Redux" tab.

You should now see the Redux DevTools interface, which will display the state and actions of your Aurelia Store application.

{% hint style="info" %}
The Redux DevTools Extension might not support all the features of the Aurelia Store plugin, such as middleware or history navigation with `StateHistory`. However, it still provides a valuable tool for inspecting and debugging the core state management aspects of your application.
{% endhint %}

