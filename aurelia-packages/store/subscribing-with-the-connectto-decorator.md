# Subscribing with the connectTo decorator

### Basic Usage

```typescript
// app.ts
import { connectTo } from 'aurelia-store';
import { State } from './state';

@connectTo()
export class App {
  state: State;
}
```

**Explanation:**

* **`import { connectTo } from 'aurelia-store';`**: Imports the `connectTo` decorator.
* **`@connectTo()`**: This decorator automatically connects the component to the store. It creates a `state` property on the component and assigns the latest state to it whenever a new state is emitted. The decorator overrides the component's `bind` and `unbind` lifecycle hooks to manage the subscription to the store's state. It will also make a `_stateSubscription` property available on your component where the subscription is kept and which is used during the `unbind` for cleanup.

### Custom Selectors

You can provide a custom selector function to `connectTo` if you only want to subscribe to a specific part of the state:

```typescript
// app.ts
import { connectTo } from 'aurelia-store';
import { State } from './state';
import { pluck } from 'rxjs/operators';

@connectTo({
  selector: (store) => store.state.pipe(pluck('frameworks'))
})
export class App {
  state: string[]; // state will now only contain the frameworks array
}
```

**Explanation:**

* **`selector: (store) => store.state.pipe(pluck('frameworks'))`**: This selector function uses the RxJS `pluck` operator to extract only the `frameworks` property from the state.
* **`state: string[];`**: The `state` property will now only contain the `frameworks` array, not the entire state object.

### Multiple Selectors and Targets

You can also use multiple selectors to subscribe to different parts of the state and assign them to different properties:

```typescript
// app.ts
import { connectTo } from 'aurelia-store';
import { State } from './state';
import { pluck } from 'rxjs/operators';

@connectTo({
  selector: {
    frameworks: (store) => store.state.pipe(pluck('frameworks')),
    isLoading: (store) => store.state.pipe(pluck('isLoading'))
  }
})
export class App {
  frameworks: string[];
  isLoading: boolean;
}
```

**Explanation:**

* **`selector: { ... }`**: An object containing multiple selector functions.
* **`frameworks: (store) => ...`**: Selects the `frameworks` property and assigns it to the `frameworks` property on the component.
* **`isLoading: (store) => ...`**: Selects the `isLoading` property and assigns it to the `isLoading` property on the component.

You can also specify a custom target property:

```typescript
@connectTo({
  target: 'appState',
  selector: {
    frameworks: (store) => store.state.pipe(pluck('frameworks')),
    isLoading: (store) => store.state.pipe(pluck('isLoading'))
  }
})
export class App {
  appState: {
    frameworks: string[];
    isLoading: boolean;
  }
}
```

When using `target` with multiple selectors, the `appState` property would contain an object with `frameworks` and `isLoading` properties.

### Custom Setup and Teardown

You can customize the lifecycle methods used for setup and teardown:

```typescript
@connectTo({
  selector: (store) => store.state.pipe(pluck('frameworks')),
  setup: 'attached',
  teardown: 'detached'
})
export class App {
  // ...
}
```

**Explanation:**

* **`setup: 'attached'`**: The subscription will be created in the `attached` lifecycle method.
* **`teardown: 'detached'`**: The subscription will be disposed of in the `detached` lifecycle method.

### Change Handling

The `connectTo` decorator provides several ways to handle state changes:

* **`stateChanged(newState, oldState)`**: This method is called automatically when the state changes. It receives the new state and the old state as arguments.

```typescript
@connectTo()
export class App {
  state: State;

  stateChanged(newState: State, oldState: State) {
    console.log('State changed from', oldState, 'to', newState);
  }
}
```

* **`targetChanged(newState, oldState)`**: If you specify a `target` property, this method will be called instead of `stateChanged`.

```typescript
@connectTo({
  target: 'appState'
})
export class App {
  appState: State;

  appStateChanged(newState: State, oldState: State) {
    console.log('appState changed from', oldState, 'to', newState);
  }
}
```

* **`onChanged: 'methodName'`**: You can specify a custom method to be called when the state changes.

```typescript
@connectTo({
  onChanged: 'myCustomChangeHandler'
})
export class App {
  state: State;

  myCustomChangeHandler(newState: State, oldState: State) {
    console.log('State changed in my custom handler', newState, oldState);
  }
}
```

* **`propertyChanged(propName, newState, oldState)`:** Called after any other change handler. Receives the changed property name (or 'state' for the whole state object), the new state and the old state.

```typescript
@connectTo()
export class App {
  state: State;

  propertyChanged(propName: string, newState: State, oldState: State) {
    console.log(`Property ${propName} changed from`, oldState, 'to', newState);
  }
}
```

**Note:** The change handler methods are called _before_ the target property is updated. This allows you to compare the new and old states and potentially make further changes.
