# View Swapping and Animation

Aurelia provides strategies for controlling how views are swapped when navigating between routes. This is particularly useful when implementing animations.

## Setting Swap Order

You can set the swap order strategy on the `<router-view>` element:

```markup
<router-view swap-order="before"></router-view>
```

**Available swap order strategies are:**

* `before`: Animate the new view in before removing the current view
* `with`: Animate the new view at the same time the current view is removed
* `after`: Animate the new view in after the current view has been removed (default)

## Implementing Animations

You'll typically use Aurelia's animation module and the swap order strategy to implement animations. Here's a basic example:

```typescript
import { useView, PLATFORM } from 'aurelia-framework';
import { Router } from 'aurelia-router';

@useView(PLATFORM.moduleName('app.html'))
export class App {
  configureRouter(config, router: Router) {
    config.title = 'My App';
    config.map([
      { route: ['', 'home'], name: 'home', moduleId: PLATFORM.moduleName('home/index') },
      { route: 'users', name: 'users', moduleId: PLATFORM.moduleName('users/index') }
    ]);

    config.addPipelineStep('postcomplete', PostCompleteStep);
  }
}

class PostCompleteStep {
  run(instruction, next) {
    instruction.router.container.viewModel.setAnimationClass('slide-in-right');
    return next();
  }
}
```

Then in your CSS:

```css
.slide-in-right {
  animation: slideInRight 0.5s forwards;
}

@keyframes slideInRight {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}
```
