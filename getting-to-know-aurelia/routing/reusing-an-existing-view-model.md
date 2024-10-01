# Reusing an Existing View Model

By default, Aurelia creates a new view model instance for each navigation. However, you can control this behavior using the `determineActivationStrategy` hook.

```typescript
import { activationStrategy } from 'aurelia-router';

export class MyViewModel {
  determineActivationStrategy() {
    return activationStrategy.replace;
  }
}
```

**Available strategies are:**

* `activationStrategy.replace`: Create a new instance of the view model
* `activationStrategy.invokeLifecycle`: Reuse the existing view model, but call the lifecycle hooks

You can also specify the activation strategy in the route config:

```typescript
config.map([
  { 
    route: 'reused',
    name: 'reused',
    moduleId: PLATFORM.moduleName('reused/index'),
    activationStrategy: activationStrategy.invokeLifecycle
  }
]);
```

This approach is useful when you want to reuse a view model but still refresh its data or state when navigating to it.
