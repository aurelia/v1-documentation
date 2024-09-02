# Template compilation using processContent

The `processContent` function is a powerful feature in Aurelia 1 that allows developers to manipulate the content of a custom element's view before it's compiled. This feature is crucial for scenarios where you need to preprocess templates, implement custom syntax, or modify the DOM structure before Aurelia's standard compilation process.

#### Syntax

```javascript
@processContent(contentProcessorFunction)
```

### Implementing processContent

#### Basic Structure

To use `processContent`, add it as a decorator to your custom element class:

```javascript
import { customElement, processContent } from 'aurelia-framework';

@customElement('my-element')
@processContent((compiler, resources, node, instruction) => {
  // Your content processing logic here
  return true;
})
export class MyElement {
  // Your element logic
}
```

#### Parameters

The `processContent` function receives four parameters:

1. `compiler`: The `ViewCompiler` instance, used for compiling views.
2. `resources`: The `ViewResources` instance, containing view-related resources.
3. `node`: The DOM node representing the view (usually a `DocumentFragment`).
4. `instruction`: The `BehaviorInstruction` for the custom element.

#### Return Value

* `true`: Aurelia will compile the content normally after your processing.
* `false`: Aurelia will not compile the content, leaving it as-is.

### Use Cases and Examples

#### 1. Preprocessing Text Content

Replace placeholders in the template:

```javascript
@customElement('date-display')
@processContent((compiler, resources, node, instruction) => {
  const content = node.textContent;
  node.textContent = content.replace(/\{\{TODAY\}\}/g, new Date().toLocaleDateString());
  return true;
})
export class DateDisplay {}
```

#### 2. Modifying DOM Structure

Add or remove elements from the template:

```javascript
@customElement('enhanced-list')
@processContent((compiler, resources, node, instruction) => {
  const list = node.querySelector('ul');
  if (list) {
    const wrapper = document.createElement('div');
    wrapper.className = 'list-wrapper';
    list.parentNode.insertBefore(wrapper, list);
    wrapper.appendChild(list);
  }
  return true;
})
export class EnhancedList {}
```

#### 3. Implementing Custom Syntax

Process custom attribute syntax:

```javascript
@customElement('custom-button')
@processContent((compiler, resources, node, instruction) => {
  const button = node.querySelector('button');
  if (button && button.hasAttribute('custom-color')) {
    const color = button.getAttribute('custom-color');
    button.style.backgroundColor = color;
    button.removeAttribute('custom-color');
  }
  return true;
})
export class CustomButton {}
```

#### 4. Conditional Compilation

Selectively compile parts of the template:

```javascript
@customElement('conditional-element')
@processContent((compiler, resources, node, instruction) => {
  const shouldCompile = node.hasAttribute('compile');
  if (!shouldCompile) {
    // Remove all bindable attributes to prevent compilation
    Array.from(node.attributes).forEach(attr => {
      if (attr.name.startsWith('bind-')) {
        node.removeAttribute(attr.name);
      }
    });
  }
  return shouldCompile;
})
export class ConditionalElement {}
```

### Advanced Techniques

#### Accessing ViewResources

Use the `resources` parameter to register custom elements or attributes:

```javascript
@processContent((compiler, resources, node, instruction) => {
  resources.registerElement('custom-tag', CustomTagClass);
  return true;
})
```

#### Working with the ViewCompiler

Utilize the `compiler` to manually compile parts of the view:

```javascript
@processContent((compiler, resources, node, instruction) => {
  const customPart = node.querySelector('.custom-part');
  if (customPart) {
    const factory = compiler.compile(customPart, resources);
    // Use the factory as needed
  }
  return true;
})
```

### Best Practices

1. Performance Considerations:
   * Use `processContent` judiciously, as it runs for each instance of the custom element.
   * Keep manipulations focused and efficient.
2. Maintainability:
   * Document the purpose and behavior of your `processContent` function.
   * Consider extracting complex logic into separate functions for clarity.
3. Compatibility:
   * Be aware that DOM manipulations may affect Aurelia's binding system.
   * Test thoroughly, especially with complex templates or bindings.
4. Alternatives:
   * Consider using value converters, custom attributes, or composition for simpler cases.

### Conclusion

The `processContent` function is a powerful tool in Aurelia 1 for customizing template compilation. It offers unparalleled flexibility in manipulating views before they're processed by Aurelia's standard compilation. While powerful, it should be used thoughtfully, considering both performance and maintainability implications.
