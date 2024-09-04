# Custom attributes

Custom attributes in Aurelia allow you to extend HTML elements with additional functionality. This guide covers various ways to create and use custom attributes.

## Creating Custom Attributes

### Convention-based Custom Attributes

Aurelia uses naming conventions to recognize custom attributes automatically.

```javascript
export class UppercaseCustomAttribute {
  static inject = [Element];

  constructor(element) {
    this.element = element;
  }

  valueChanged(newValue) {
    this.element.textContent = newValue.toUpperCase();
  }
}
```

Usage:

```markup
<span uppercase.bind="name"></span>
```

###

Explicit Custom Attributes

Use the `@customAttribute` decorator for explicit declaration.

```javascript
import {customAttribute, inject} from 'aurelia-framework';

@customAttribute('uppercase')
@inject(Element)
export class UppercaseCustomAttribute {
  constructor(element) {
    this.element = element;
  }

  valueChanged(newValue) {
    this.element.textContent = newValue.toUpperCase();
  }
}
```

## Single Value Binding

By default, custom attributes use a single-value binding.

```javascript
@customAttribute('font-size')
export class FontSizeCustomAttribute {
  valueChanged(newValue) {
    this.element.style.fontSize = newValue;
  }
}
```

Usage:

```markup
<div font-size.bind="fontSize">Resizable text</div>
```

## Accessing the Element

Inject the `Element` to access the DOM element.

```javascript
@customAttribute('tooltip')
@inject(Element)
export class TooltipCustomAttribute {
  constructor(element) {
    this.element = element;
  }

  valueChanged(newValue) {
    this.element.title = newValue;
  }
}
```

## Multi-value Bindable Properties

Use `@bindable` for multiple properties.

```javascript
import {customAttribute, bindable} from 'aurelia-framework';

@customAttribute('border')
export class BorderCustomAttribute {
  @bindable width = '1px';
  @bindable color = 'black';
  @bindable style = 'solid';

  bind() {
    this.element.style.border = `${this.width} ${this.style} ${this.color}`;
  }
}
```

Usage:

```markup
<div border="width: 2px; color: red; style: dashed;">Bordered content</div>
```

## Responding to Bindable Property Changes

Implement `*Changed` methods to respond to property changes.

```javascript
@customAttribute('highlight')
export class HighlightCustomAttribute {
  @bindable color = 'yellow';

  colorChanged(newValue, oldValue) {
    this.element.style.backgroundColor = newValue;
  }
}
```

## Options Binding

Use options binding for complex configurations.

```javascript
@customAttribute('popover')
export class PopoverCustomAttribute {
  @bindable({ defaultBindingMode: bindingMode.oneTime }) options = {};

  bind() {
    // Initialize popover with options
  }
}
```

Usage:

```markup
<button popover.bind="{ title: 'Info', content: 'Click for details' }">Info</button>
```

## Specifying a Primary Property

Use `primaryProperty` for a default property.

```javascript
import {customAttribute, bindable} from 'aurelia-framework';

@customAttribute('tooltip')
export class TooltipCustomAttribute {
  @bindable({ primaryProperty: true }) content = '';
  @bindable placement = 'top';

  bind() {
    // Initialize tooltip
  }
}
```

Usage:

```markup
<button tooltip="Click me!">Hover for tooltip</button>
```

## Lifecycle Hooks

Implement lifecycle hooks for fine-grained control.

```javascript
@customAttribute('fade-in')
export class FadeInCustomAttribute {
  attached() {
    this.element.style.opacity = '0';
    setTimeout(() => {
      this.element.style.transition = 'opacity 0.5s';
      this.element.style.opacity = '1';
    }, 0);
  }

  detached() {
    this.element.style.opacity = '';
    this.element.style.transition = '';
  }
}
```

## Caveats and Considerations

### Naming Conflicts

When using convention-based custom attributes, be cautious of naming conflicts with built-in HTML attributes or other libraries.

```javascript
// This might conflict with the native 'style' attribute
export class StyleCustomAttribute { ... }
```

To avoid conflicts, consider using a prefix or the `@customAttribute` decorator with a unique name.

### Unexpected Behavior with One-time Bindings

One-time bindings (`.one-time`) won't update the attribute if the bound value changes after initial binding.

```html
<div my-attribute.one-time="dynamicValue"></div>
```

Use `.bind` or `.to-view` for values that might change.

### Attribute vs. Element Semantics

Custom attributes modify existing elements, while custom elements create new elements. Be mindful of this distinction when deciding between the two.

```markup
<!-- Attribute: modifies existing element -->
<div my-attribute></div>

<!-- Element: creates new element -->
<my-element></my-element>
```

### Case Sensitivity in Templates

Custom attribute names in Aurelia 1 are case-insensitive and follow a specific convention:

* Define custom attribute classes using PascalCase, typically with a 'CustomAttribute' suffix.
* Use kebab-case in HTML templates when applying the custom attribute.
* Aurelia does not automatically convert between different cases.

```javascript
// Definition: PascalCase
export class MyCustomAttributeCustomAttribute {
  // ...
}
```

Usage:

```markup
<!-- Usage: kebab-case -->
<div my-custom-attribute="value"></div>
```

Be aware that using camelCase or PascalCase in templates will not work:

```markup
<!-- These won't work -->
<div myCustomAttribute="value"></div>
<div MyCustomAttribute="value"></div>
```

### Unbinding and Memory Leaks

Ensure proper cleanup in the `unbind()` lifecycle method to prevent memory leaks, especially when working with third-party libraries.

```javascript
@customAttribute('third-party-plugin')
export class ThirdPartyPluginAttribute {
  unbind() {
    // Clean up third-party plugin
  }
}
```
