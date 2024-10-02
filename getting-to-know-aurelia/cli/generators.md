# Generators

The Aurelia CLI includes generators to scaffold common Aurelia resources. Use the following command pattern:

```bash
au generate <resource> <name>
```

Available resources include:

* `element`
* `attribute`
* `value-converter`
* `binding-behavior`
* `task`
* `generator`

For example, to generate a custom element:

```bash
au generate element my-awesome-element
```

This creates `src/resources/elements/my-awesome-element.js` (or `.ts` if using TypeScript).

{% hint style="info" %}
By Aurelia convention, use **kebab-case** for naming elements, attributes, value converters, and binding behaviors.
{% endhint %}

You can also create custom generators:

```bash
au generate generator my-custom-generator
```
