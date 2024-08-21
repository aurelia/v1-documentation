# Event binding

Event binding in Aurelia provides a seamless way to handle DOM events within your application. Attaching event listeners directly to your view templates allows you to easily respond to user interactions such as clicks, key presses, and more. This guide will delve into the specifics of event binding in Aurelia, offering detailed explanations and examples to enhance your understanding and usage of this feature.

## Understanding Event Binding <a href="#understanding-event-binding" id="understanding-event-binding"></a>

Aurelia simplifies the process of binding events to methods in your view model. It uses a straightforward syntax that allows you to specify the type of event and the corresponding method to invoke when that event occurs.

### Event Binding Syntax <a href="#event-binding-syntax" id="event-binding-syntax"></a>

The general syntax for event binding in Aurelia 2 is as follows:

```html
<element event.trigger="methodName(argument1, argument2, ...)">
```

* `element` represents the HTML element to which you want to attach the event listener.
* `event` is the event you want to listen for (e.g., `click`, `keypress`).
* `.trigger` the binding command tells Aurelia to listen for the specified event and call the method when the event is fired. You can also use `.delegate` for instances where are are multiple click events within a common ancestor.
* `methodName` is the method's name in your view model that will be called when the event occurs?
* `argument1`, `argument2`, ... are optional arguments you can pass to the method.

### Event Binding Commands <a href="#event-binding-commands" id="event-binding-commands"></a>

Aurelia offers three primary commands for event binding:

1. `trigger`: directly attaches an event handler to the element. The specified expression runs when the event occurs.
2. `delegate`: attaches a single event handler to the document (or nearest shadow DOM boundary). It manages all events of the specified type during the bubbling phase, correctly routing them back to their original targets to run the associated expression.
3. `capture`: similar to `delegate`, but operates during the capturing phase. It attaches a single handler to the document (or nearest shadow DOM boundary), managing events of the specified type and routing them to their original targets to execute the associated expression.

**Click event binding**

To listen for a click event on a button and call a method named `handleClick`, you would write:

```html
<button click.trigger="handleClick()">Click me!</button>
```

When the button is clicked, your view-model's `handleClick` method will be executed.

**Click event binding (delegate)**

```html
<button type="button" click.delegate="select('yes')">Yes</button>
<button type="button" click.delegate="select('no')">No</button>
```

#### Passing Event Data <a href="#passing-event-data" id="passing-event-data"></a>

You can pass the event object itself or other custom data to your event handler method. For instance, to pass the event object to the `handleClick` method, you would modify the binding like this:

```html
<button click.trigger="handleClick($event)">Click me!</button>
```

In your view model, the `handleClick` method would accept the event object as a parameter:

```javascript
export class MyViewModel {
  handleClick(event) {
    // Handle the click event
  }
}
```

Aurelia will automatically call [preventDefault()](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault) on events handled with `delegate` or `trigger` binding. Most of the time, this is the behavior you want. To turn this off, return `true` from your event handler function.
