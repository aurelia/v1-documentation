# Bindable properties

In Aurelia, bindable properties provide a way to define attributes on custom elements or custom attributes that can receive data from their consumer. These properties allow seamless communication between components and their consumers, facilitating data flows within your application.

Bindable properties can be configured with different binding modes to control the direction of data flow between the component and its consumers. This section will explore how to define bindable properties and the available binding modes and provide examples to illustrate usage.

## Defining Bindable Properties

To define a bindable property in a custom element, we use the `bindable` decorator. This decorator can be applied to a class field to indicate that it can receive data from a parent component.

### Basic Example

```javascript
import { bindable } from 'aurelia-framework';

export class MyCustomElement {
  @bindable myProperty;

  // Additional component logic...
}
```

In this example, `myProperty` is a bindable property defined in `MyCustomElement`. This allows the property to be bound from a parent element like so:

```markup
<my-custom-element my-property.bind="parentValue"></my-custom-element>
```

## Binding Modes

Aurelia provides several binding modes to control the data flow direction between parent and child components:

1. **`one-way`:** Data flows from the parent to the child. This is the default binding mode.
2. **`two-way`:** Data flows both from parent to child and from child back to parent.
3. **`one-time`:** Data is transferred from parent to child only once when the component is initialized.
4. **`from-view`:** Data flows from the child to the parent.

## Specifying Binding Modes

You can specify a binding mode using the `bindingMode` property:

```javascript
import { bindable, bindingMode } from 'aurelia-framework';

export class MyCustomElement {
  @bindable({ defaultBindingMode: bindingMode.twoWay }) myTwoWayProperty;

  // Additional component logic...
}
```

### Examples

**One-Way Binding (Default)**

```javascript
export class MyCustomElement {
  @bindable myProperty;
}

<!-- Parent -->
<my-custom-element my-property.bind="parentValue"></my-custom-element>
```

In one-way binding, `myProperty` will be updated when `parentValue` changes, but not vice versa.

**Two-Way Binding**

```javascript
export class MyCustomElement {
  @bindable({ defaultBindingMode: bindingMode.twoWay }) myTwoWayProperty;
}

<!-- Parent -->
<my-custom-element my-two-way-property.two-way="parentValue"></my-custom-element>
```

In two-way binding, `myTwoWayProperty` and `parentValue` are kept in sync. Changes to either property will reflect on the other.

**One-Time Binding**

```javascript
export class MyCustomElement {
  @bindable({ defaultBindingMode: bindingMode.oneTime }) myOneTimeProperty;
}

<!-- Parent -->
<my-custom-element my-one-time-property.one-time="parentValue"></my-custom-element>
```

With one-time binding, `myOneTimeProperty` is set to the initial value of `parentValue`, and does not update on future changes.

**From-View Binding**

```javascript
export class MyCustomElement {
  @bindable({ defaultBindingMode: bindingMode.fromView }) myFromViewProperty;
}

<!-- Parent -->
<my-custom-element my-from-view-property.from-view="parentValue"></my-custom-element>
```

`myFromViewProperty` updates `parentValue` only when `myFromViewProperty` changes within `MyCustomElement`.

## Binding Mode Configuration

Here is a concise reference to use while configuring bindable properties:

```javascript
import { bindable, bindingMode } from 'aurelia-framework';

export class MyCustomElement {
  // One-way (default)
  @bindable myOneWayProperty;

  // Two-way
  @bindable({ defaultBindingMode: bindingMode.twoWay }) myTwoWayProperty;

  // One-time
  @bindable({ defaultBindingMode: bindingMode.oneTime }) myOneTimeProperty;

  // From-view
  @bindable({ defaultBindingMode: bindingMode.fromView }) myFromViewProperty;
}
```

Bindable properties in Aurelia provide a robust mechanism for data flow between custom elements and their consumers. Through different binding modes, developers can fine-tune how data is shared across components, enabling a wide array of use cases.
