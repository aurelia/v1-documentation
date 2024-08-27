# The as-element (element aliasing)

When building complex tables in Aurelia, you might need custom elements to behave like standard HTML elements. For instance, you may want a custom element to function as a or within a table structure. The as-element attribute is perfect for this scenario. It allows your custom elements to seamlessly integrate into the table layout while maintaining their unique functionality.&#x20;

This approach is particularly useful when generating dynamic table rows from data.

{% code title="as-element.html" %}
```markup
<template>
  <require from="./hello-row.html"></require>
  <table>
    <tr as-element="hello-row">
  </table>
</template>
```
{% endcode %}

{% code title="hello-row.html" %}
```markup
<template>
  <td>Hello</td>
  <td>World</td>
</template>
```
{% endcode %}

The `as-element` attribute tells Aurelia that we want the content of the table row to be exactly what our `hello-row` template wraps. The way different browsers render tables means this may be necessary sometimes.

{% hint style="danger" %}
Warning: Internet explorer does not allow `<template>` elements inside `<table>` or `<select>`
{% endhint %}
