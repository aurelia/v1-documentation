# Template syntax

Aurelia 1's templating system strikes a perfect balance between simplicity and power. It enhances HTML with dynamic capabilities, allowing you to create interactive UIs without sacrificing readability.

At its heart, Aurelia 1 templates are HTML files supercharged with data-binding and custom attributes. They seamlessly connect with your JavaScript or TypeScript code, creating a responsive, data-driven interface. This tight integration means your UI reacts naturally to user actions and data changes.

From your first Aurelia 1 project, you'll appreciate how the templating syntax feels familiar yet powerful. Whether building a complex component or displaying data, Aurelia 1's approach keeps your code clean and expressive. It's designed to make your development process more intuitive and efficient, letting you focus on creating great user experiences.

### Features of Aurelia Templating <a href="#features-of-aurelia-templating" id="features-of-aurelia-templating"></a>

* **Two-Way Data Binding:** Aurelia's robust data binding system ensures a seamless data flow between your application's model and the view, keeping both in sync without extra effort.
* **Custom Elements and Attributes:** Extend your HTML with custom elements and attributes that encapsulate complex behaviors, promoting code reuse and modularity.
* **Adaptive Dynamic Composition:** Dynamically render components and templates based on your application's state or user interactions, enabling the creation of flexible and adaptive UIs.
* **Rich Templating Syntax:** Utilize Aurelia's powerful templating syntax for iterating over data, conditionally rendering parts of your UI, and easily handling events.
* **Expression and Interpolation:** Effortlessly bind data to your templates and manipulate attributes with Aurelia's straightforward expression syntax.

## Quick Example

Every Aurelia template is wrapped in a `<template>` tag and paired with a view-model:

```html
<template>
  <h1>${greeting}</h1>
  <input value.bind="name">
  <p>Hello, ${name}!</p>
</template>
```

```javascript
export class Welcome {
  greeting = ‘Welcome to Aurelia';
  name = ‘World';
}
```

## In This Section

- [Binding](binding.md) — Core data binding with `.bind`, `.one-way`, `.two-way`, and `.one-time`
- [Attribute binding](attribute-binding.md) — Bind to HTML attributes and properties
- [Event binding](event-binding.md) — Handle DOM events with `.trigger` and `.delegate`
- [Delegate vs trigger](delegate-vs-trigger.md) — When to use event delegation vs direct binding
- [Text interpolation](text-interpolation.md) — String interpolation with `${}` expressions
- [Element content](element-content.md) — Control element text and HTML content
- [Function references](function-references.md) — Pass function references in templates
- [Element references](element-references.md) — Get direct access to DOM elements with `ref`
- [Contextual properties](contextual-properties.md) — Special properties like `$this`, `$parent`, and `$index`
- [Expression syntax](expression-syntax.md) — The expression language used in bindings
- [Computed properties](computed-properties.md) — Derived properties and the `@computedFrom` decorator
- [Let element](let-element.md) — Declare template-local variables with `<let>`
- [Binding engine internals](binding-engine-internals.md) — How Aurelia's binding system works under the hood
