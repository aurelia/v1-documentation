# Helpful Properties, Functions, and Advanced Querying

Aurelia's testing utilities provide a rich set of properties and functions that go beyond basic component rendering. This section explores the advanced capabilities of the ComponentTester and auxiliary testing functions.

### ComponentTester Properties

#### Core Properties

ComponentTester provides several key properties that help in component testing:

*   `element`

    * **What it is**: The root HTML element of the rendered component
    * **Use cases**:
      * Direct DOM manipulation checks
      * Accessing rendered component's root element
      * Verifying element attributes or classes

    ```javascript
    describe('Element Property', () => {
      let component;

      beforeEach(() => {
        component = StageComponent
          .withResources(PLATFORM.moduleName('my-component'))
          .inView('<my-component></my-component>');
      });

      it('provides access to the rendered element', done => {
        component.create(bootstrap).then(() => {
          // Check element existence
          expect(component.element).toBeTruthy();

          // Verify element attributes
          expect(component.element.className).toContain('expected-class');

          // Check element properties
          expect(component.element.tagName).toBe('DIV');

          done();
        });
      });
    });
    ```
*   `viewModel`

    * **What it is**: The actual instance of the component's view-model
    * **Key benefits**:
      * Direct access to component's internal state
      * Ability to call methods and check properties
      * Simulate interactions programmatically

    ```javascript
    describe('ViewModel Property', () => {
      let component;

      beforeEach(() => {
        class TestComponent {
          someMethod() { return 'test'; }
          @bindable propertyToTest;
        }

        component = StageComponent
          .withResources(PLATFORM.moduleName('test-component'))
          .inView('<test-component></test-component>')
          .boundTo(new TestComponent());
      });

      it('provides access to view-model methods and properties', done => {
        component.create(bootstrap).then(() => {
          // Call view-model methods
          expect(component.viewModel.someMethod()).toBe('test');

          // Set and check bindable properties
          component.viewModel.propertyToTest = 'new value';
          expect(component.viewModel.propertyToTest).toBe('new value');

          done();
        });
      });
    });
    ```

#### Lifecycle Control Methods

Lifecycle control methods allow precise management of component lifecycle:

*   `manuallyHandleLifecycle()`

    * **Purpose**: Take full control of component lifecycle methods
    * **Key benefits**:
      * Step-by-step lifecycle testing
      * Precise timing control
      * Simulate different lifecycle scenarios

    ```javascript
    describe('Manual Lifecycle Control', () => {
      let component;
      let lifecycleTracker;

      beforeEach(() => {
        class TrackingComponent {
          lifecycleSteps = [];

          bind() { this.lifecycleSteps.push('bind'); }
          unbind() { this.lifecycleSteps.push('unbind'); }
          attached() { this.lifecycleSteps.push('attached'); }
          detached() { this.lifecycleSteps.push('detached'); }
        }

        component = StageComponent
          .withResources(PLATFORM.moduleName('tracking-component'))
          .inView('<tracking-component></tracking-component>')
          .boundTo(new TrackingComponent());
      });

      it('allows step-by-step lifecycle method testing', done => {
        component
          .manuallyHandleLifecycle()
          .create(bootstrap)
          .then(() => component.bind())
          .then(() => component.attached())
          .then(() => {
            const viewModel = component.viewModel;
            expect(viewModel.lifecycleSteps).toEqual(['bind', 'attached']);
            done();
          });
      });
    });
    ```

#### Key Lifecycle Methods

* `bind()`: Manually trigger data binding
* `unbind()`: Manually remove data bindings
* `attached()`: Simulate component attachment to DOM
* `detached()`: Simulate component removal from DOM

### Element Waiting Utilities

#### `waitForElement` and `waitForElements`

Waiting utilities help manage asynchronous rendering:

* **`waitForElement`**
  * Waits for a single element to be present or absent
  * Configurable timeout and polling interval
  * Useful for testing async content loading
* **`waitForElements`**
  * Waits for multiple elements
  * Similar configuration options to `waitForElement`

```javascript
describe('Element Waiting Utilities', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources(PLATFORM.moduleName('async-component'))
      .inView('<async-component></async-component>');
  });

  it('waits for element presence', done => {
    component.create(bootstrap)
      .then(() => component.waitForElement('.async-content', {
        timeout: 2000,    // 2 seconds max wait
        interval: 100     // check every 100ms
      }))
      .then(element => {
        expect(element).toBeTruthy();
        done();
      });
  });

  it('waits for element absence', done => {
    component.create(bootstrap)
      .then(() => component.waitForElement('.loading-indicator', {
        present: false,
        timeout: 2000
      }))
      .then(() => {
        // Element disappeared
        done();
      });
  });
});
```

### Advanced Querying with `waitFor`

#### Flexible Waiting Mechanism

* **`waitFor`**
  * Most flexible waiting utility
  * Works with custom conditions
  * Supports complex querying scenarios

```javascript
import { waitFor } from 'aurelia-testing';

describe('Advanced Waiting Mechanisms', () => {
  it('waits for custom condition', done => {
    waitFor(() => {
      const items = document.querySelectorAll('.dynamic-list-item');
      return items.length > 3 ? items : null;
    }, {
      timeout: 3000,  // Maximum wait time
      interval: 200   // Polling frequency
    })
    .then(items => {
      expect(items.length).toBeGreaterThan(3);
      done();
    });
  });
});
```
