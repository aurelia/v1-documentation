# Binding behaviors

Binding behaviors are view resources that work alongside value converters, custom attributes, and custom elements. They're similar to value converters in that you use them declaratively in binding expressions to influence how bindings work.

The key distinction is that binding behaviors have full access to the binding instance throughout its lifecycle. Value converters, on the other hand, can only intercept values moving between the model and view.

This expanded access allows binding behaviors to modify the binding's behavior, opening up a range of powerful possibilities. Let's explore some of these scenarios below.

I apologize for the lack of detail. You're right, more comprehensive information is needed. Let me provide a more thorough explanation of binding behaviors in Aurelia.

## Built-in Binding Behaviors

Aurelia provides several built-in binding behaviors that address common scenarios. Let's explore each in detail:

### throttle

The `throttle` binding behavior limits the rate at which a binding updates.

```html
<input value.bind="searchTerm & throttle:500">
```

In this example, the binding will update at most once every 500 milliseconds. This is particularly useful for scenarios where you want to reduce the frequency of updates for rapidly changing values, such as search inputs or sliders.

If no value is specified, `throttle` defaults to 200 milliseconds:

```html
<input value.bind="searchTerm & throttle">
```

The throttle behavior works by updating the binding immediately for the first change, then waiting for the specified duration before allowing another update. Any changes during this waiting period are ignored.

The first thing you probably noticed in the example above is the `&` symbol used to declare binding behavior expressions. Binding behavior expressions use the same syntax pattern as value converter expressions:

* Binding behaviors can accept arguments: `firstName & myBehavior:arg1:arg2:arg3`
* A binding expression can contain multiple binding behaviors: `firstName & behavior1 & behavior2:arg1`.
* Binding expressions can also include a combination of value converters and binding behaviors: `${foo | upperCase | truncate:3 & throttle & anotherBehavior:arg1:arg2}`.

### debounce

While similar to `throttle`, the `debounce` binding behavior delays updates and resets the timer on each change.

```html
<input value.bind="searchTerm & debounce:300">
```

This waits 300 milliseconds after the last change before updating the binding. It's ideal for scenarios where you want to wait for user input to stabilize before triggering an action, such as autocomplete suggestions.

Like `throttle`, if no value is specified, `debounce` defaults to 200 milliseconds:

```html
<input value.bind="searchTerm & debounce">
```

The key difference between `throttle` and `debounce` is that `debounce` will wait for the specified duration of inactivity before updating, while `throttle` updates immediately and then ignores changes for the specified duration.

### updateTrigger

The `updateTrigger` binding behavior allows you to specify which events trigger a binding update.

```html
<input value.bind="name & updateTrigger:'blur'">
```

By default, input elements update on the `change` event. The `updateTrigger` behavior lets you override this. Common values include:

* `'blur'`: Updates when the element loses focus
* `'input'`: Updates on every keystroke (for text inputs)
* `'change'`: The default behavior for most elements

You can also specify multiple events:

```html
<input value.bind="name & updateTrigger:['blur','change']">
```

This behavior is particularly useful for form validation scenarios where you might want to validate on blur rather than on every keystroke.

### signal

The `signal` binding behavior enables manual triggering of updates using a signal.

```html
<div textContent.bind="message & signal:'refresh'"></div>
```

To use `signal`, you need to inject the `BindingSignaler` into your view-model:

```javascript
import {BindingSignaler} from 'aurelia-templating-resources';

export class MyViewModel {
  static inject = [BindingSignaler];
  
  constructor(signaler) {
    this.signaler = signaler;
  }

  refreshMessage() {
    this.message = "Updated message";
    this.signaler.signal('refresh');
  }
}
```

The `signal` behavior is useful when you need fine-grained control over when bindings update, especially for bindings that don't naturally trigger updates (like computed properties).

### oneTime

The `oneTime` binding behavior sets up a one-way binding that updates only once when the view is bound.

```html
<div textContent.bind="staticMessage & oneTime"></div>
```

This behavior is crucial for optimizing performance with static content. Once set, the binding disconnects, freeing up resources. Use this for any content you know won't change after initial rendering.

### self

The `self` binding behavior binds to the element itself rather than a property.

```html
<div ref="myElement & self"></div>
```

This allows direct access to the DOM element in your view-model:

```javascript
export class MyViewModel {
  attached() {
    console.log(this.myElement); // Logs the actual DOM element
  }
}
```

The `self` behavior is particularly useful when you need to manipulate the DOM directly or when working with third-party libraries that require DOM element references.

## Custom Binding Behaviors

Aurelia allows you to create custom binding behaviors for specialized functionality. Here's a more detailed look at creating a custom behavior:

```javascript
export class CustomBehavior {
  bind(binding, scope, bindingContext) {
    // Store the original updateTarget function
    this.originalUpdate = binding.updateTarget;

    // Replace with custom logic
    binding.updateTarget = (value) => {
      // Perform custom operations here
      let modifiedValue = value.toUpperCase();
      
      // Call the original update with the modified value
      this.originalUpdate(modifiedValue);
    };
  }

  unbind(binding, scope) {
    // Restore the original updateTarget function
    binding.updateTarget = this.originalUpdate;
  }
}
```

This example creates a custom behavior that converts bound values to uppercase.

To use this custom behavior, you need to register it in your app's main configuration:

```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .feature('resources');

  aurelia.use.globalResources('./custom-behavior');

  aurelia.start().then(() => aurelia.setRoot());
}
```

Then you can use it in your templates:

```html
<input value.bind="someValue & customBehavior">
```

Custom behaviors allow you to encapsulate complex binding logic, making your templates cleaner and more maintainable.

## Combining Binding Behaviors

Aurelia allows chaining multiple binding behaviors:

```html
<input value.bind="searchTerm & debounce:300 & throttle:500">
```

Behaviors are applied from left to right. In this example, `debounce` is applied first, followed by `throttle`. This means the binding will wait for 300ms of inactivity, then ensure updates happen at most every 500ms.

Understanding the order of application is crucial when combining behaviors to achieve the desired effect.
