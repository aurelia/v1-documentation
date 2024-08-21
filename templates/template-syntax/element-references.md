# Element references

The `ref` binding command lets you create references to DOM elements in your view. Its basic syntax is simple: `ref="expression"`. The DOM element is assigned to the specified expression when the view is data-bound. This powerful feature gives you direct access to elements, enabling more complex interactions and manipulations when needed.

```markup
<template>
  <input type="text" ref="nameInput"> ${nameInput.value}
</template>
```

The `ref` command has several qualifiers you can use in conjunction with custom elements and attributes:

* `element.ref="expression"`: create a reference to the DOM element (same as `ref="expression"`).
* `attribute-name.ref="expression"`: create a reference to a custom attribute's view model.
* `view-model.ref="expression"`: create a reference to a custom element's view model.
* `view.ref="expression"`: reference a custom element's view instance (not an HTML Element).
* `controller.ref="expression"`: create a reference to a custom element's controller instance.
