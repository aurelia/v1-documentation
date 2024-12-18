# Dispatching actions

Dispatching an action is the process of triggering it to update the state. You use the `store.dispatch()` method to dispatch actions.

### `store.dispatch()`

The `dispatch()` method takes at least one argument:

* **The action to dispatch:** This can be either the action function itself or the name of the action if it was registered with a name using `store.registerAction()`.
* **Optional arguments:** Any additional arguments you provide to `dispatch()` will be passed to the action function.

### Example

```typescript
// app.ts
import { inject } from 'aurelia-dependency-injection';
import { Store } from 'aurelia-store';
import { addFramework } from './actions'; // Import the action
import { State } from './state';

@inject(Store)
export class App {
  private state: State;
  constructor(private store: Store<State>) {
    this.store.registerAction('Add Framework', addFramework);

    this.store.state.subscribe((newState) => {
      this.state = newState;
    });
  }

  addNewFramework(name: string) {
    this.store.dispatch(addFramework, name); 
    // Or: this.store.dispatch('Add Framework', name);
  }
}
```

**Explanation:**

1. **`this.store.dispatch(addFramework, name);`:** This line dispatches the `addFramework` action, passing the `name` argument to it.
2. **`this.store.dispatch('Add Framework', name);`:** This line does the same thing but uses the registered name of the action instead of the action function itself.

### Awaiting `dispatch`

If you dispatch an asynchronous action, and you need to perform operations that depend on the state being updated _after_ the action has completed, you should `await` the `dispatch` call:

```typescript
async addNewFramework(name: string) {
  await this.store.dispatch(addFramework, name);
  console.log('Framework added:', this.state.frameworks); // This will log the updated state
}
```

By awaiting `dispatch`, you ensure that the subsequent code will only execute after the asynchronous action has finished and the state has been updated.

### Piping Multiple Actions as One

Sometimes, you might want to dispatch multiple actions in sequence to achieve a specific state update. While you could call `dispatch()` multiple times, this can lead to unnecessary intermediate state updates and make it harder to track the overall change in debugging tools. The Aurelia Store plugin provides a way to pipe multiple actions together and dispatch them as a single unit using `store.pipe()`.

#### `store.pipe()`

The `store.pipe()` method allows you to chain multiple actions together. It returns a `PipedDispatch` object, which has a `dispatch()` method to execute the entire pipeline.

#### Example

```typescript
// app.ts
import { inject } from 'aurelia-dependency-injection';
import { Store } from 'aurelia-store';
import { addFramework, setFrameworks, State } from './actions';

@inject(Store)
export class App {
  constructor(private store: Store<State>) {
    this.store.registerAction('Add Framework', addFramework);
    this.store.registerAction('Set Frameworks', setFrameworks);
  }

  addAndSetFrameworks(newFramework: string, allFrameworks: string[]) {
    this.store
      .pipe(addFramework, newFramework)
      .pipe(setFrameworks, allFrameworks)
      .dispatch();
  }
}
```

**Explanation:**

1. **`this.store.pipe(addFramework, newFramework)`:** This starts a new pipeline by adding the `addFramework` action with the `newFramework` argument.
2. **`.pipe(setFrameworks, allFrameworks)`:** This adds the `setFrameworks` action with the `allFrameworks` argument to the pipeline.
3. **`.dispatch();`:** This executes the entire pipeline as a single unit.

#### Benefits of Piping

* **Single State Transition:** Piping actions ensures that only a single state update is emitted at the end of the pipeline, even if multiple actions are involved.
* **Atomic Updates:** The state update appears as a single, atomic operation in debugging tools like the Redux DevTools.
* **Improved Readability:** Piping can make complex state updates more readable and easier to understand.

#### DevTools Display

In the Redux DevTools, piped actions will be displayed in the format `actionA->actionB->actionC`, clearly indicating the sequence of actions that were executed.

#### Error Handling

If any action in the pipeline throws an error or returns a promise that rejects, the rest of the pipeline will not be executed, and the error will be propagated. If you are awaiting the `dispatch()` of a piped action, you will receive the error there.

### Using the `dispatchify` Higher-Order Function

Sometimes, you might not have direct access to the `Store` instance, especially in child components that are designed to be reusable and independent of the specific state management implementation. In such cases, the `dispatchify` higher-order function can be a valuable tool.

#### What is `dispatchify`?

`dispatchify` is a function that takes an action as an argument and returns a new function. This new function, when called, will automatically obtain the `Store` instance and dispatch the provided action with any arguments passed to it. It essentially "wraps" your action, making it easier to dispatch from components that don't have a direct reference to the store.

#### Example:

Let's say you have a `framework-list` component that displays a list of frameworks and a `framework-item` component that represents a single framework and has a button to add a new one. You want to keep the `framework-item` component "dumb" or presentational, meaning it doesn't know anything about the store or how actions are dispatched.

**`actions.ts`:**

```typescript
import { State } from './state';

export const addFramework = (state: State, frameworkName: string) => {
  const newState = Object.assign({}, state);
  newState.frameworks = [...newState.frameworks, frameworkName];
  return newState;
};
```

**`app.ts`:**

```typescript
import { inject } from 'aurelia-dependency-injection';
import { Store } from 'aurelia-store';
import * as actions from './actions';
import { State } from './state';

@inject(Store)
export class App {
  constructor(private store: Store<State>) {
    this.store.registerAction('Add framework', actions.addFramework);
  }
}
```

**`app.html`:**

```html
<template>
  <require from="./framework-list"></require>

  <framework-list></framework-list>
</template>
```

**`framework-list.ts`:**

```typescript
import { bindable } from 'aurelia-framework';
import { connectTo, dispatchify } from 'aurelia-store';
import * as actions from './actions';
import { State } from './state';

@connectTo()
export class FrameworkList {
  @bindable state: State;
  addFramework: (name: string) => Promise<void>;

  constructor() {
    // Wrap the addFramework action with dispatchify
    this.addFramework = dispatchify(actions.addFramework);
  }
}
```

**`framework-list.html`:**

```html
<template>
  <require from="./framework-item"></require>
  <framework-item add.bind="addFramework"></framework-item>
  <ul>
    <li repeat.for="framework of state.frameworks">${framework}</li>
  </ul>
</template>
```

**`framework-item.ts`:**

```typescript
import { bindable } from 'aurelia-framework';

export class FrameworkItem {
  @bindable add: (name: string) => Promise<void>;
  newFrameworkName: string;

  addFramework() {
    this.add(this.newFrameworkName);
    this.newFrameworkName = ''; // Clear the input field
  }
}
```

**`framework-item.html`:**

```html
<template>
  New framework name:
  <input value.bind="newFrameworkName" />
  <button click.trigger="addFramework()">Add</button>
</template>
```

**Explanation:**

1. In `framework-list.ts`, we use `dispatchify(actions.addFramework)` to create a new function `addFramework`. This function is now ready to be passed down to child components.
2. In `framework-item.ts`, the `add` function (which is the `addFramework` function passed from the parent) is called directly from the template when the button is clicked. The `framework-item` component doesn't need to know anything about the store or how to dispatch actions.

#### Presentational vs. Container Components

This approach allows you to clearly separate your components into:

* **Presentational (or "dumb") components:** These components are only concerned with how things look. They receive data and callbacks (like the `add` function) via properties and don't interact with the store directly. `framework-item` is an example.
* **Container (or "smart") components:** These components are aware of the store and handle the interaction with it. They connect to the state, dispatch actions, and pass data and callbacks down to presentational components. `framework-list` and `App` are examples.
