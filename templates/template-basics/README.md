# Template basics

Aurelia's templating system strikes a perfect balance between simplicity and power. Whether you're building a basic website or a complex application, you'll find it intuitive and flexible. In this guide, we'll cover:

1. Creating static templates
2. Nesting templates within parent structures
3. Data binding and manipulation via view-models
4. Working with conditionals, repeaters, and events

By the end, you'll have a solid grasp of Aurelia's templating capabilities, enabling you to craft dynamic and responsive user interfaces easily.

## A simple template

Every Aurelia template needs a \<template> wrapper. Let's start with the simplest example – a component that displays "Hello World":

{% code title="hello-world.html" %}
```markup
<template>
  <p>
    Hello, World!
  </p>
</template>
```
{% endcode %}

This straightforward approach sets the foundation for more complex templates you'll create as you dive deeper into Aurelia.

All Aurelia templates work with a view-model, so let's create one:

{% code title="hello.js" %}
```javascript
export class Hello {
  constructor() {
    this.name = 'John Doe';
  }
}
```
{% endcode %}

Let's bind the `name` property in our view-model into our template using Aurelia's string interpolation:

{% code title="hello.html" %}
```markup
<template>
  <p>
    Hello, ${name}!
  </p>
</template>
```
{% endcode %}

Aurelia's templating system minimizes context switching between JavaScript and markup. It leverages ES2015's string interpolation syntax (`${}`) to insert values into your templates seamlessly.

For example, `${name}` in your template will display the value of the name property. It's that straightforward. Need more complex logic? No problem. You can include expressions within the `${}` brackets, giving you flexibility without sacrificing readability.

{% code title="greeter.html" %}
```markup
<template>
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
    setTimeout(
      () => this.arriving = false,
      5000);
  }
}
```
{% endcode %}

The template uses a ternary operator to display "Hello" when `arriving` is true, and "Goodbye" when it's false. Our view-model initializes `arriving` to true, then switches it to false after 5 seconds.

When you run this, you'll initially see "Hello, John Doe!" followed by "Goodbye, John Doe!" after 5 seconds. Aurelia automatically re-evaluates the string interpolation when `arriving` changes.

Don't worry about performance – Aurelia's observable-based binding system reacts to changes instantly without dirty-checking. This means your app stays snappy, even as you add complexity to your templates and view-models.
