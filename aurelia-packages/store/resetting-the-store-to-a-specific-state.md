# Resetting the store to a specific state

There might be situations where you need to reset the store to a specific state, bypassing the regular action dispatch and middleware pipeline. The `store.resetToState` method allows you to do this.

{% hint style="warning" %}
Use this method with care, as it can introduce side effects if not handled properly. It's primarily intended for scenarios like restoring an initial state or implementing time-traveling debugging.
{% endhint %}

### Usage:

```typescript
import { inject } from 'aurelia-dependency-injection';
import { Store } from 'aurelia-store';
import { initialState, State } from './state';

@inject(Store)
export class App {
  constructor(private store: Store<State>) {}

  resetToInitialState() {
    this.store.resetToState(initialState);
  }
}
```

This code will immediately replace the current state with the `initialState`, without going through the regular action dispatch process. Any active subscriptions will receive the new state.

{% hint style="warning" %}
States reset using `resetToState` are not tracked by the Redux DevTools, so it might affect your debugging workflow if you rely heavily on time-travelling.
{% endhint %}

