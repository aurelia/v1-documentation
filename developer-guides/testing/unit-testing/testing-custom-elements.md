# Testing Custom Elements

### Basic Component Test

Let's break down a comprehensive example of testing a custom element:

```javascript
import {StageComponent} from 'aurelia-testing';
import {bootstrap} from 'aurelia-bootstrapper';

describe('UserProfile Component', () => {
  let component;

  // Custom element to be tested
  class UserProfile {
    @bindable firstName;
    @bindable lastName;

    get fullName() {
      return `${this.firstName} ${this.lastName}`;
    }
  }

  beforeEach(() => {
    // Stage the component for testing
    component = StageComponent
      .withResources(PLATFORM.moduleName('user-profile'))
      .inView('<user-profile first-name.bind="firstName" last-name.bind="lastName"></user-profile>')
      .boundTo({
        firstName: 'John',
        lastName: 'Doe'
      });
  });

  it('should render full name correctly', done => {
    component.create(bootstrap).then(() => {
      const nameElement = document.querySelector('.full-name');
      expect(nameElement.textContent).toBe('John Doe');
      done();
    }).catch(done.fail);
  });

  afterEach(() => {
    component.dispose();
  });
});
```

{% hint style="info" %}
Note the use of `PLATFORM.moduleName()` for better compatibility with module loaders like Webpack.
{% endhint %}

### Detailed Binding and Property Tests

Testing bindable properties in our component.

```javascript
describe('UserProfile Bindable Properties', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources(PLATFORM.moduleName('user-profile'))
      .inView(`
        <user-profile 
          first-name.bind="firstName" 
          last-name.bind="lastName"
          age.bind="age">
        </user-profile>
      `)
      .boundTo({
        firstName: 'Jane',
        lastName: 'Smith',
        age: 30
      });
  });

  it('should update when bound properties change', done => {
    component.create(bootstrap).then(() => {
      // Initial state check
      let firstNameElement = document.querySelector('.first-name');
      let lastNameElement = document.querySelector('.last-name');
      let ageElement = document.querySelector('.age');

      expect(firstNameElement.textContent).toBe('Jane');
      expect(lastNameElement.textContent).toBe('Smith');
      expect(ageElement.textContent).toBe('30');

      // Update bound properties
      component.viewModel.firstName = 'John';
      component.viewModel.lastName = 'Doe';
      component.viewModel.age = 35;

      // Wait for bindings to update
      return new Promise(resolve => setTimeout(resolve, 50));
    }).then(() => {
      let firstNameElement = document.querySelector('.first-name');
      let lastNameElement = document.querySelector('.last-name');
      let ageElement = document.querySelector('.age');

      expect(firstNameElement.textContent).toBe('John');
      expect(lastNameElement.textContent).toBe('Doe');
      expect(ageElement.textContent).toBe('35');

      done();
    }).catch(done.fail);
  });
});

```

### Lifecycle Method Testing

Here we manually control the lifecycle so we can decide when the lifecycle methods get fired.

```javascript
describe('UserProfile Lifecycle', () => {
  let component;
  let lifecycleTracker;

  beforeEach(() => {
    // Create a mock view-model with lifecycle tracking
    class UserProfile {
      constructor() {
        this.lifecycleTracker = [];
      }

      bind() {
        this.lifecycleTracker.push('bind');
      }

      attached() {
        this.lifecycleTracker.push('attached');
      }

      detached() {
        this.lifecycleTracker.push('detached');
      }

      unbind() {
        this.lifecycleTracker.push('unbind');
      }
    }

    component = StageComponent
      .withResources(PLATFORM.moduleName('user-profile'))
      .inView('<user-profile></user-profile>')
      .boundTo(new UserProfile());
  });

  it('should trigger lifecycle methods in correct order', done => {
    // Use manuallyHandleLifecycle to control method invocation
    component.manuallyHandleLifecycle().create(bootstrap)
      .then(() => {
        // Initial state - no lifecycle methods called
        expect(component.viewModel.lifecycleTracker.length).toBe(0);

        // Manually trigger bind
        return component.bind();
      })
      .then(() => {
        expect(component.viewModel.lifecycleTracker).toContain('bind');

        // Manually trigger attached
        return component.attached();
      })
      .then(() => {
        expect(component.viewModel.lifecycleTracker).toContain('attached');

        // Manually trigger detached
        return component.detached();
      })
      .then(() => {
        expect(component.viewModel.lifecycleTracker).toContain('detached');

        // Manually trigger unbind
        return component.unbind();
      })
      .then(() => {
        expect(component.viewModel.lifecycleTracker).toContain('unbind');
        done();
      })
      .catch(done.fail);
  });
});

```

### Complex Binding Scenarios

Testing two-way and computed bindings.

```javascript
describe('Complex Binding Scenarios', () => {
  let component;

  beforeEach(() => {
    class ComplexComponent {
      @bindable firstName;
      @bindable lastName;
      
      get fullName() {
        return `${this.firstName} ${this.lastName}`;
      }

      updateFullName(newFullName) {
        const [first, last] = newFullName.split(' ');
        this.firstName = first;
        this.lastName = last;
      }
    }

    component = StageComponent
      .withResources(PLATFORM.moduleName('complex-component'))
      .inView(`
        <complex-component 
          first-name.bind="firstName" 
          last-name.bind="lastName" 
          full-name.two-way="fullName">
        </complex-component>
      `)
      .boundTo({
        firstName: 'John',
        lastName: 'Doe',
        fullName: ''
      });
  });

  it('should handle two-way binding and computed properties', done => {
    component.create(bootstrap).then(() => {
      // Test computed property
      expect(component.viewModel.fullName).toBe('John Doe');

      // Update individual properties
      component.viewModel.firstName = 'Jane';
      component.viewModel.lastName = 'Smith';

      // Wait for binding to update
      return new Promise(resolve => setTimeout(resolve, 50));
    }).then(() => {
      expect(component.viewModel.fullName).toBe('Jane Smith');

      // Test two-way binding update
      component.viewModel.updateFullName('Alice Johnson');

      // Wait for binding to update
      return new Promise(resolve => setTimeout(resolve, 50));
    }).then(() => {
      expect(component.viewModel.firstName).toBe('Alice');
      expect(component.viewModel.lastName).toBe('Johnson');

      done();
    }).catch(done.fail);
  });
});

```

### Best Practices for Component Testing

* Always use `PLATFORM.moduleName()` for resources
* Dispose of components in `afterEach()`
* Use `done()` or return a Promise for async tests
* Test various binding scenarios
* Mock external dependencies
* Check both initial state and state after updates

### Common Pitfalls to Avoid

* Don't rely on implementation details
* Avoid testing private methods
* Use meaningful test descriptions
* Handle async operations carefully
* Clean up DOM between tests
