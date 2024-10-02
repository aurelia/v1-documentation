# Task queue

Aurelia's Task Queue is a powerful tool that allows developers to schedule tasks at different phases of the application's lifecycle. By leveraging the Task Queue, you can optimize rendering, manage asynchronous operations, and ensure that your application's state remains consistent and performant.

## Understanding the Task Queue

The Task Queue in Aurelia manages task execution by categorizing them into microtasks and microtasks. This distinction helps determine the appropriate timing for task execution, ensuring that important operations are performed at the right moment within the application's rendering cycle.

### Task Queue Methods

Aurelia's Task Queue provides two primary methods for scheduling tasks:

#### `queueMicroTask`

* **Description:** Schedules a microtask to be executed after the current task but before the next rendering cycle.
* **Use Case:** Ideal for tasks that need to run before the DOM updates, such as updating application state or preparing data for rendering.

#### `queueTask`

* **Description:** Schedules a macrotask to be executed after the current task and after the next rendering cycle.
* **Use Case:** Suitable for non-urgent tasks that can be performed after the DOM has been updated, such as logging or cleanup operations.

### Differences Between `queueMicroTask` and `queueTask`

| Feature              | `queueMicroTask`                                      | `queueTask`                                         |
| -------------------- | ----------------------------------------------------- | --------------------------------------------------- |
| **Execution Timing** | After the current task, before the next rendering     | After the current task and after the next rendering |
| **Use Case**         | Critical operations that must occur before DOM update | Non-critical operations that can occur post-render  |
| **Priority**         | Higher priority                                       | Lower priority                                      |

## When to Use `queueMicroTask` vs `queueTask`

Choosing between `queueMicroTask` and `queueTask` depends on the nature and urgency of the task:

* **Use `queueMicroTask` when:**
  * You need to update application state that affects the UI before the next render.
  * Performing validations or computations that should precede DOM updates.
* **Use `queueTask` when:**
  * Executing operations that do not impact the immediate UI rendering.
  * Performing cleanup, logging, or other auxiliary tasks post-render.

### How the Task Queue Works

Under the hood, Aurelia's Task Queue leverages the browser's event loop to manage task execution:

1. **Microtasks:** Managed using mechanisms like `Promise.resolve().then()`, ensuring they run immediately after the call stack clears before the next rendering cycle.
2. **Macrotasks:** Managed using `setTimeout` or `setImmediate`, allowing tasks to be executed after the current rendering cycle is complete.

This separation ensures that critical UI updates are handled promptly while less urgent tasks do not block the rendering process.

## Examples

### Using `queueMicroTask`

```javascript
import { TaskQueue } from 'aurelia-framework';

export class Example {
  static inject = [TaskQueue];

  constructor(taskQueue) {
    this.taskQueue = taskQueue;
  }

  updateState() {
    this.taskQueue.queueMicroTask(() => {
      this.state = this.calculateNewState();
      // This update occurs before the next DOM render
    });
  }

  calculateNewState() {
    // Compute new state logic
    return { /* new state */ };
  }
}

```

### Using `queueTask`

```javascript
import { TaskQueue } from 'aurelia-framework';

export class Example {
  static inject = [TaskQueue];

  constructor(taskQueue) {
    this.taskQueue = taskQueue;
  }

  performCleanup() {
    this.taskQueue.queueTask(() => {
      this.cleanupResources();
      // This cleanup occurs after the DOM has been updated
    });
  }

  cleanupResources() {
    // Cleanup logic
  }
}
```

## Best Practices

* **Prioritize Critical Updates:** Use `queueMicroTask` for operations that impact the immediate UI to ensure a smooth user experience.
* **Defer Non-Essential Tasks:** Utilize `queueTask` for tasks that do not need to block the rendering cycle, enhancing application performance.
* **Consistent Task Scheduling:** Maintain consistency in how and where you schedule tasks to keep your application's behavior predictable and manageable.
