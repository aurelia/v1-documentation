# Advanced Testing

Advanced testing in Aurelia goes beyond simple component rendering. This section covers complex testing strategies that address real-world scenarios developers encounter when building robust applications. We'll explore techniques for mocking dependencies, working with dependency injection, and handling more intricate testing requirements.

### Mocking Dependencies

#### Why Mock Dependencies?

Mocking dependencies is crucial for isolating the component you're testing from external services, databases, or complex logic. This approach allows you to:

* Control test environments
* Simulate different scenarios
* Avoid external system dependencies
* Speed up test execution

### Dependency Mocking Example

```javascript
import {StageComponent} from 'aurelia-testing';
import {bootstrap} from 'aurelia-bootstrapper';

// Mock Service
class MockUserService {
  constructor() {
    this.users = [
      { id: 1, name: 'John Doe' },
      { id: 2, name: 'Jane Smith' }
    ];
  }

  async getUsers() {
    return Promise.resolve(this.users);
  }

  async getUserById(id) {
    return Promise.resolve(
      this.users.find(user => user.id === id)
    );
  }
}

describe('User List Component with Mocked Service', () => {
  let component;
  let mockUserService;

  beforeEach(() => {
    // Create mock service
    mockUserService = new MockUserService();

    // Stage component with mocked service
    component = StageComponent
      .withResources(PLATFORM.moduleName('user-list'))
      .inView('<user-list></user-list>')
      .bootstrap(aurelia => {
        aurelia.use.standardConfiguration();
        
        // Register mock service in the dependency injection container
        aurelia.container.registerInstance(UserService, mockUserService);
      });
  });

  it('should render users from mocked service', done => {
    component.create(bootstrap).then(() => {
      // Get rendered user elements
      const userElements = document.querySelectorAll('.user-item');
      
      expect(userElements.length).toBe(2);
      expect(userElements[0].textContent).toContain('John Doe');
      expect(userElements[1].textContent).toContain('Jane Smith');
      
      done();
    }).catch(done.fail);
  });

  afterEach(() => {
    component.dispose();
  });
});
```

### Dependency Injection Testing

#### Complex Dependency Resolution

When testing components with multiple dependencies, you can manually configure the dependency injection container:

```javascript
describe('Complex Component with Multiple Dependencies', () => {
  let component;
  let container;
  let mockLogger;
  let mockAuthService;
  let mockDataService;

  beforeEach(() => {
    // Create mock dependencies
    mockLogger = {
      log: jasmine.createSpy('log'),
      error: jasmine.createSpy('error')
    };

    mockAuthService = {
      isAuthenticated: jasmine.createSpy('isAuthenticated').and.returnValue(true),
      getCurrentUser: jasmine.createSpy('getCurrentUser').and.returnValue({
        id: 1,
        username: 'testuser'
      })
    };

    mockDataService = {
      fetchData: jasmine.createSpy('fetchData').and.returnValue(
        Promise.resolve([{ id: 1, name: 'Test Data' }])
      )
    };

    // Create a new dependency injection container
    container = new Container();

    // Register mock dependencies
    container.registerInstance(Logger, mockLogger);
    container.registerInstance(AuthService, mockAuthService);
    container.registerInstance(DataService, mockDataService);

    // Stage the component with custom container configuration
    component = StageComponent
      .withResources(PLATFORM.moduleName('complex-component'))
      .inView('<complex-component></complex-component>')
      .bootstrap(aurelia => {
        aurelia.use.standardConfiguration();
        
        // Replace container with our configured container
        aurelia.container = container;
      });
  });

  it('should resolve and use all dependencies correctly', done => {
    component.create(bootstrap).then(() => {
      // Verify dependency interactions
      expect(mockAuthService.isAuthenticated).toHaveBeenCalled();
      expect(mockDataService.fetchData).toHaveBeenCalled();
      expect(mockLogger.log).toHaveBeenCalled();

      done();
    }).catch(done.fail);
  });
});
```

### Async Testing Patterns

#### Handling Asynchronous Operations

Testing components with async operations requires careful approach:

```javascript
describe('Async Component Testing', () => {
  let component;
  let mockAsyncService;

  beforeEach(() => {
    mockAsyncService = {
      fetchDataWithDelay: (delay) => {
        return new Promise((resolve) => {
          setTimeout(() => {
            resolve([
              { id: 1, name: 'Async Item 1' },
              { id: 2, name: 'Async Item 2' }
            ]);
          }, delay);
        });
      }
    };

    component = StageComponent
      .withResources(PLATFORM.moduleName('async-component'))
      .inView('<async-component></async-component>')
      .bootstrap(aurelia => {
        aurelia.container.registerInstance(AsyncService, mockAsyncService);
      });
  });

  it('should handle long-running async operations', done => {
    component.create(bootstrap)
      .then(() => {
        // Use waitForElement to ensure async content is loaded
        return component.waitForElement('.async-content');
      })
      .then((asyncContentElement) => {
        expect(asyncContentElement).toBeTruthy();
        expect(asyncContentElement.children.length).toBe(2);
        done();
      })
      .catch(done.fail);
  });
});
```

### Event and Interaction Testing

#### Simulating User Interactions

```javascript
describe('Interactive Component Testing', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources(PLATFORM.moduleName('interactive-component'))
      .inView(`
        <interactive-component 
          on-submit.trigger="handleSubmit($event)">
        </interactive-component>
      `)
      .boundTo({
        handleSubmit: jasmine.createSpy('handleSubmit')
      });
  });

  it('should trigger event handlers', done => {
    component.create(bootstrap)
      .then(() => {
        // Find submit button and trigger click
        const submitButton = document.querySelector('.submit-button');
        submitButton.click();

        // Verify event handler was called
        expect(component.viewModel.handleSubmit).toHaveBeenCalled();
        done();
      })
      .catch(done.fail);
  });
});
```

### Best practices

* Always mock external services and complex dependencies
* Use dependency injection to control component context
* Handle async operations carefully
* Verify both synchronous and asynchronous behaviors
* Test edge cases and error scenarios
* Keep tests isolated and focused
