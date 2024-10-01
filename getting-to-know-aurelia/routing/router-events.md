# Router events

You can subscribe to various router events using Aurelia's event aggregator. This is useful for things like showing loading indicators, tracking page views, or responding to navigation errors.

```typescript
import { EventAggregator } from 'aurelia-event-aggregator';
import { RouterEvent, NavigationInstruction } from 'aurelia-router';

export class App {
  static inject = [EventAggregator];

  constructor(private ea: EventAggregator) {
    ea.subscribe(RouterEvent.Processing, (event: { instruction: NavigationInstruction }) => {
      console.log('Navigation started', event.instruction);
      // Show loading indicator
    });

    ea.subscribe(RouterEvent.Complete, (event: { instruction: NavigationInstruction }) => {
      console.log('Navigation completed', event.instruction);
      // Hide loading indicator
    });

    ea.subscribe(RouterEvent.Success, (event: { instruction: NavigationInstruction }) => {
      console.log('Navigation succeeded', event.instruction);
      // Track page view
    });

    ea.subscribe(RouterEvent.Error, (event: { instruction: NavigationInstruction, result: any }) => {
      console.error('Navigation failed', event.instruction, event.result);
      // Show error message
    });
  }
}
```

**Available events include:**

* `RouterEvent.Processing`: Fired when navigation starts.
* `RouterEvent.Complete`: Fired when navigation completes (regardless of success or failure).
* `RouterEvent.Success`: Fired when navigation completes successfully.
* `RouterEvent.Error`: Fired when navigation fails.
* `RouterEvent.Canceled`: Fired when navigation is canceled.

These events provide detailed information about the navigation process, allowing you to create rich, responsive user experiences around your application's routing.
