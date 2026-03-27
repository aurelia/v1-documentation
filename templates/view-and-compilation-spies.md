# View and Compilation Spies

If you've installed the `aurelia-testing` plugin, you have access to two special custom attributes for debugging templates: `view-spy` and `compile-spy`.

### Installation

```bash
npm install aurelia-testing
```

Register the plugin in your `main.js` or `main.ts`:

```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin('aurelia-testing');  // Enables view-spy and compile-spy
}
```

{% hint style="warning" %}
The `aurelia-testing` plugin is intended for development and debugging. Remove it from your production configuration to avoid unnecessary overhead.
{% endhint %}

## view-spy

The `view-spy` attribute logs Aurelia's internal `View` object to the browser console when the element is bound. This gives you access to the view's binding context, overrideContext, and all associated bindings.

{% code title="hello.html" %}
```markup
<template>
  <p view-spy>Hello!</p>
</template>
```
{% endcode %}

Open your browser's developer console, and you'll see the `View` object logged. You can inspect:

- **`bindingContext`** — The view-model bound to the template
- **`overrideContext`** — Contextual properties like `$parent`, `$index`, etc.
- **`bindings`** — All active data bindings on the element
- **`children`** — Any child views (e.g., from repeaters or compositions)

This is particularly useful when debugging data binding issues or verifying that the correct data is reaching your template.

## compile-spy

The `compile-spy` attribute logs the compiler's `TargetInstruction` to the console during compilation. This reveals how Aurelia interpreted your template markup — what bindings, behaviors, and custom elements it detected.

{% code title="hello.html" %}
```markup
<template>
  <p compile-spy>Hello!</p>
</template>
```
{% endcode %}

The logged `TargetInstruction` includes:

- **`expressions`** — Binding expressions found on the element
- **`behaviorInstructions`** — Custom attributes and behaviors applied
- **`contentExpression`** — Instructions for the element's text content

## Using Both Together

You can apply both attributes to the same element for comprehensive debugging:

{% code title="debug-example.html" %}
```markup
<template>
  <div view-spy compile-spy>
    <p>${message}</p>
    <input value.bind="name">
  </div>
</template>
```
{% endcode %}

This logs both the compiled instructions (how Aurelia parsed the template) and the runtime view (the live binding state), giving you a complete picture of what's happening.

## Debugging Repeaters

Spies are especially helpful when debugging `repeat.for` to inspect each iteration's binding context:

```markup
<template>
  <ul>
    <li repeat.for="item of items" view-spy>${item.name}</li>
  </ul>
</template>
```

Each repeated item logs its own `View` object, letting you verify that `$index`, `$first`, `$last`, and other contextual properties are correct.
