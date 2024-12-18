# Subscribing to the stream of states

Once you have configured the Aurelia Store plugin, you can start subscribing to the stream of states in your components. This allows your components to react to changes in the application's state and update their UI accordingly.

### Injecting the Store

To access the store in your component, you need to inject it using Aurelia's dependency injection.

```typescript
// app.ts
import { inject } from 'aurelia-dependency-injection';
import { Store } from 'aurelia-store';
import { State } from './state'; // Import your State interface

@inject(Store)
export class App {
  private state: State;
  private subscription: any;

  constructor(private store: Store<State>) {}

  // ... rest of your component code ...
}
```

**Explanation:**

1. **`import { inject } from 'aurelia-dependency-injection';`**: Imports the `inject` decorator from Aurelia's dependency injection library.
2. **`import { Store } from 'aurelia-store';`**: Imports the `Store` class from the Aurelia Store plugin.
3. **`@inject(Store)`**: This decorator tells Aurelia to inject an instance of the `Store` into your component's constructor.
4. **`constructor(private store: Store<State>) {}`**: The constructor receives the injected `Store` instance. The `private` keyword automatically creates a property called `store` on your component and assigns the injected store to it.

### Creating a Subscription

The recommended place to subscribe to the state is in the `bind` lifecycle method of your component.

```typescript
// app.ts
// ... imports ...

@inject(Store)
export class App {
  private state: State;
  private subscription: any;

  constructor(private store: Store<State>) {}

  bind() {
    this.subscription = this.store.state.subscribe(
      (newState: State) => this.state = newState
    );
  }

  // ... rest of your component code ...
}
```

**Explanation:**

1. **`bind() { ... }`**: The `bind` lifecycle method is called when the component is attached to the DOM and its data bindings are ready.
2. **`this.subscription = this.store.state.subscribe(...)`**:
   * `this.store.state` accesses the `Observable` that emits the stream of states.
   * `.subscribe(...)` subscribes to the `Observable`, which means that the provided callback function will be executed every time a new state is emitted.
   * The subscription is assigned to `this.subscription` so that we can unsubscribe later.
3. **`(newState: State) => this.state = newState`**: This is the callback function that is executed for each new state. It simply assigns the new state to the component's `state` property.

#### Disposing of the Subscription

It's important to unsubscribe from the state when the component is unbound (e.g., when it's removed from the DOM). This prevents memory leaks. You should do this in the `unbind` lifecycle method.

```typescript
// app.ts
// ... imports ...

@inject(Store)
export class App {
  // ...

  unbind() {
    if (this.subscription) {
      this.subscription.unsubscribe();
    }
  }
}
```

**Explanation:**

1. **`unbind() { ... }`**: The `unbind` lifecycle method is called when the component is detached from the DOM.
2. **`this.subscription.unsubscribe();`**: This unsubscribes from the state `Observable`, preventing the callback function from being executed anymore.

### Consuming the State in the Template

Now that your component has access to the state and updates it whenever a new state is emitted, you can use the state directly in your template:

```html
<!-- app.html -->
<template>
  <h1>Frameworks</h1>
  <ul>
    <li repeat.for="framework of state.frameworks">${framework}</li>
  </ul>
</template>
```

**Explanation:**

* **`state.frameworks`**: This accesses the `frameworks` array from the current state object.
* **`repeat.for="framework of state.frameworks"`**: This Aurelia repeater iterates over the `frameworks` array and creates a list item for each framework.

With this setup, your component will automatically re-render whenever the `frameworks` array in the state changes.
