# Dynamic composition

Embracing the DRY (Don't Repeat Yourself) Principle doesn't mean we should tightly couple our view and view-model pairs. Thankfully, Aurelia offers a solution: a custom element that dynamically combines an HTML template, a view-model, and optional initialization data. This flexible approach allows for more modular and reusable code, making your development process smoother and more efficient.

{% code title="compose-template.html" %}
```markup
<template>
    <compose view-model="hello"
        view="./hello.html"
        model.bind="{ target : 'World' }" ></compose>
</template>
```
{% endcode %}

{% code title="hello.html" %}
```markup
<template>
  Hello, ${friend}!
</template>
```
{% endcode %}

{% code title="hello.js" %}
```javascript
export class Hello {
  activate(model) {
    this.friend = model.target;
  }
}
```
{% endcode %}

Note that the view-model we're composing into has an `activate` method. When we use `model.bind`, the contents are passed to `activate`. We then pull the exact value that we need out of the passed model and assign it.
