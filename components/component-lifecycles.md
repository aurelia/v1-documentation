# Component lifecycles

Aurelia components follow a consistent lifecycle that is comprised of several stages. Each stage is associated with specific callbacks that you can implement to execute custom logic at different points in the lifecycle of a component.

## constructor

**Description**

Not an Aurelia callback. The constructor belongs to your class and is always called first before any of the Aurelia callbacks.

## created

#### Description

The `created` callback is the first lifecycle callback that a component experiences. It is invoked after the component's constructor has been executed but before any data binding has occurred. This is the perfect time to set up component properties independent of DOM or data-binding context.

#### Purpose

* Initialize component-specific settings.
* Configure class properties.
* Set up preliminary states or inject dependencies that don't involve data binding.

#### Arguments

* **`owningView`**: The view instance containing this custom element. It can be used to peek at the containing environment if needed.
* **`myView`**: The view that is being created for this component. Useful for customizing the component's view.

#### Example Usage

```javascript
created(owningView, myView) {
  this.owningView = owningView;
  this.myView = myView;
  console.log('Component created, pre-bind setup complete.');
}
```

## bind

#### Description

The `bind` callback is fired once the component is bound to its context, typically signaling that the view-model properties and data-binding structures are ready for use. This stage follows the creation of the element and is crucial for initializing state that depends on bindings.

#### Purpose

* Initialize properties based on the current context.
* Set default values or configurations that may depend on dynamic data.

#### Arguments

* **`bindingContext`**: Provides the context object to which the component is bound. It is effectively the raw data or state.
* **`overrideContext`**: Supplies additional contextual information, such as parent context, that may enhance binding operations or data access.

#### Example Usage

```javascript
bind(bindingContext, overrideContext) {
  this.context = bindingContext;
  this.additionalData = overrideContext;
  console.log('Component bound to context.');
}
```

## attached

#### Description

The `attached` callback is activated when the component's view has been physically inserted into the DOM. It is now safe to perform operations requiring direct interaction with DOM elements, such as jQuery plugin initialization or DOM measurements.

#### Purpose

* Manipulate or query DOM elements now that they are part of the live document.
* Initialize or integrate third-party libraries dependent on DOM presence.

#### Arguments

* None.

#### Example Usage

```javascript
attached() {
  console.log('Component attached, now part of DOM.');
  // Direct DOM manipulation or third-party lib setup.
}
```

## detached

#### Description

The `detached` callback is called when the component's view is removed from the live DOM. This is the point to undo any DOM manipulations or to release resources that were obtained during the `attached` phase.

#### Purpose

* Clean up DOM-related artefacts (e.g., event listeners).
* Dispose of resources, timers, or intervals.

#### Arguments

* None.

#### Example Usage

```javascript
detached() {
  console.log('Component detached and removed from DOM.');
  // Clean-up operations for DOM manipulations.
}
```

## unbind

#### Description

The `unbind` callback signifies the end of the component's association with its data context. It's an opportunity to release resources, stop data subscriptions, or carry out any cleanup linked to bindings.

#### Purpose

* Clean up resources related to data binding.
* Unsubscribe from any observables or events.

#### Arguments

* None.

#### Example Usage

```javascript
unbind() {
  console.log('Component unbound from its data context.');
  // Cleanup operations for binding-related logic.
}
```

Each of these callbacks is optional. Implement whatever makes sense for your component, but don't feel obligated to implement any of them if they aren't needed for your scenario. Usually, if you implement `bind` you will need to implement `unbind`. The same goes for `attached` and `detached`, but again, it isn't mandatory.

{% hint style="warning" %}
It is important to note that if you implement the `bind` callback function, the property changed callbacks for any `bindable` properties will not be called when the property value is initially set. The changed callback will be called for when the bound value changes.
{% endhint %}

## Execution Order of Callbacks

1. **`created()`** - After the component is created but before data-binding.
2. **`bind()`** - After data-binding establishment.
3. **`attached()`** - When the component's view is added to the DOM.
4. **`detached()`** - When the component's view is removed from the DOM.
5. **`unbind()`** - When data-binding is no longer active on the component.

## Examples

### Example 1: Managing Resources with Attached/Detached

In this example, we'll use the `attached` and `detached` hooks to manage resources like event listeners.

```javascript
export class CustomElement {
  constructor() {
    this.onResize = this.onResize.bind(this);
  }

  attached() {
    window.addEventListener('resize', this.onResize);
    this.onResize(); // Initial call to handle current state
  }

  detached() {
    window.removeEventListener('resize', this.onResize);
  }

  onResize() {
    console.log('Window resized to:', window.innerWidth, window.innerHeight);
  }
}
```

For components that need to respond to global events (e.g., window resizing or keyboard shortcuts), setting up listeners in `attached` and removing them in `detached` ensures your component is responsive and doesn't leak memory.

### Example 2: Data Fetching with Bind/Unbind

In this example, we demonstrate using the `bind` and `unbind` hooks to handle data fetching and cleanup.

```javascript
export class DataFetcher {
  data = null;
  subscription = null;

  bind() {
    this.fetchData();
  }

  unbind() {
    // Assume we have a subscription to a data source
    if (this.subscription) {
      this.subscription.unsubscribe();
    }
  }

  async fetchData() {
    try {
      const response = await fetch('https://api.example.com/data');
      this.data = await response.json();
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  }
}
```

Binding is often tied with fetching data needed for the component to display. Using `bind`, you can initiate data requests, and with `unbind`, you can stop any active data subscriptions.

### Example 3: Dynamic Initialization with Created

In this case, we'll use the `created` lifecycle to perform setup tasks that occur after the view is instantiated but before any data binding.

```javascript
export class LoggerComponent {
  constructor() {
    this.logger = null;
  }

  created(owningView, myView) {
    this.logger = new LoggerService();
    this.logger.log('Component created');
  }
}
```

You might use the `created` hook to initialize services, like logging or analytics, that don't depend on the component being bound to data or added to the DOM.
