# Two-way binding and state management

Two-way binding is a powerful feature in Aurelia, but it can conflict with the principles of unidirectional data flow that are central to state management with the Aurelia Store plugin.

### The Problem with Two-Way Binding to State Objects

If you directly two-way bind to properties of objects in your state, changes made in the UI will directly mutate the state, bypassing actions and potentially leading to unpredictable behavior and making it difficult to track changes.

### Recommended Approach:

1. **Create a separate model:** Instead of directly binding to state properties, create a separate model in your component to hold the values you want to bind to.
2. **Initialize the model:** In your component's `bind` or `attached` lifecycle method, initialize the model with the values from the state. Make sure to create a copy of any objects or arrays to avoid direct mutations.
3. **Dispatch actions on changes:** When the user interacts with the UI and changes the model's values, dispatch an action to update the state.
4. **Update the model after dispatch (if necessary):** If the action modifies the state in a way that's not automatically reflected in your model (e.g., server-side validation or modification), update the model after the action has been dispatched.

#### Example:

```typescript
/*
  * state: {
  *  user: {
  *    firstName: string;
  *    lastName: string;
  *  }
  * }
  */
import { inject } from 'aurelia-dependency-injection';
import { Store } from 'aurelia-store';
import { State } from './state';

interface UserFormModel {
  firstName: string;
  lastName: string;
}

function updateUser(state: State, model: UserFormModel) {
  const newState = Object.assign({}, state);
  newState.user = Object.assign({}, model); // Update the user object in the state
  return newState;
}

@inject(Store)
export class UserProfile {
  model: UserFormModel;
  state: State;

  constructor(private store: Store<State>) {
    this.store.registerAction('updateUser', updateUser);
  }

  bind() {
    this.subscription = this.store.state.subscribe((newState) => {
      this.state = newState;
      // Create a copy to avoid direct mutations
      this.model = {
        ...newState.user
      }
    });
  }

  unbind() {
      this.subscription.unsubscribe();
  }

  async updateUser() {
    await this.store.dispatch(updateUser, this.model);
  }
}
```

```html
<template>
  <div>
    <label>First Name:</label>
    <input type="text" value.bind="model.firstName" />
  </div>
  <div>
    <label>Last Name:</label>
    <input type="text" value.bind="model.lastName" />
  </div>
  <button click.trigger="updateUser()">Update</button>

  <div>
    <b>Model:</b> First Name = ${model.firstName}, Last Name = ${model.lastName}
  </div>
  <div>
    <b>State:</b> First Name = ${state.user.firstName}, Last Name =
    ${state.user.lastName}
  </div>
</template>
```

**Explanation:**

1. We define a `UserFormModel` interface to represent the data we want to bind to in the UI.
2. The `updateUser` action takes the current state and the `model` as arguments and returns a new state with the updated user information.
3. In the `bind` method, we initialize the `model` with a copy of the `user` object from the state.
4. The `updateUser` method dispatches the `updateUser` action with the current `model` values.
5. We use one-way binding (`value.bind`) in the template to display the model's values.

**Alternative:** You can also use one-time, to-view, or one-way bindings and handle updates manually, but the core principle of avoiding direct two-way binding to state objects remains the same.

By following this approach, you maintain a clear separation between UI and application states, ensuring that all state changes go through actions and making your application more predictable and maintainable.
