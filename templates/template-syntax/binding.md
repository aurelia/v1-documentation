# Binding

Binding in Aurelia allows data from the view-model to drive template behavior. The most basic example of binding is linking a text box to the view model using `value.bind`. What if we let the user decide who they want to greet, and whether to say Hello or Goodbye?

{% code title="greeter.html" %}
```markup
<template>
  <label for="nameField">
    Who to greet?
  </label>
  <input type="text" value.bind="name" id="nameField">
  <label for="arrivingBox">
    Arriving?
  </label>
  <input type="checkbox" checked.bind="arriving" id="arrivingBox">
  <p>
    ${arriving ? 'Hello' : 'Goodbye'}, ${name}!
  </p>
</template>
```
{% endcode %}

{% code title="greeter.js" %}
```javascript
export class Greeter {
  constructor() {
    this.name = 'John Doe';
    this.arriving = true;
  }
}
```
{% endcode %}

The example showcases a text box for entering a name and a checkbox to indicate arrival status. The initial values, "John Doe" for the name and a checked box for arrival, are set in the view-model. As you modify the name or checkbox state, the greeting updates accordingly.

Key to this functionality is Aurelia's binding syntax: `value.bind` and `checked.bind`. The dot (.) in these attributes is crucial. It signals Aurelia to link the attribute on the left with the action on the right. This binding mechanism is fundamental to understanding how Aurelia connects your view and view-model.

## Binding focus

We can also use two-way data binding to communicate whether or not an element has focus:

{% code title="bind-focus.html" %}
```markup
<template>
  <input focus.bind="hasFocus">
  ${hasFocus}
</template>
```
{% endcode %}

When we click the input field, we see "true" printed. When we click elsewhere, it changes to "false".

## Binding scopes using with

We can also declare that certain parts of our markup will be referencing properties of an object in the view model, as below:

{% code title="bind-with.html" %}
```markup
<template>
  <p with.bind="first">
    <input type="text" value.bind="message">
  </p>
  <p with.bind="second">
    <input type="text" value.bind="message">
  </p>
</template>
```
{% endcode %}

{% code title="bind-with.js" %}
```javascript
export class Hello {
  constructor() {
    this.first = {
      message : 'Hello'
    };
    this.second = {
      message : 'Goodbye'
    }
  }
}
```
{% endcode %}

Using `with` is basically shorthand for "I'm working on properties of this object", which lets you reuse code as necessary.
