# Dependency Injection Testing

Dependency Injection (DI) is a core concept in Aurelia that allows for loose coupling and easier testing. Understanding how to test DI-related scenarios is crucial for robust application development.

### Basic Dependency Injection Mocking

#### Creating Mock Dependencies

```javascript
describe('Basic Dependency Injection Mocking', () => {
  // Mock service to be injected
  class MockUserService {
    constructor() {
      this.users = [
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' }
      ];
    }

    getUsers() {
      return Promise.resolve(this.users);
    }

    getUserById(id) {
      return Promise.resolve(
        this.users.find(user => user.id === id)
      );
    }
  }

  // Component that depends on the service
  class UserListViewModel {
    static inject = [UserService];

    constructor(userService) {
      this.userService = userService;
      this.users = [];
    }

    activate() {
      return this.userService.getUsers()
        .then(users => this.users = users);
    }
  }

  let container;
  let mockUserService;
  let userListViewModel;

  beforeEach(() => {
    // Create a new dependency injection container
    container = new Container();

    // Create mock service
    mockUserService = new MockUserService();

    // Register mock service in the container
    container.registerInstance(UserService, mockUserService);

    // Resolve view-model with mocked dependencies
    userListViewModel = container.get(UserListViewModel);
  });

  it('should inject mock service', async () => {
    await userListViewModel.activate();

    expect(userListViewModel.users.length).toBe(2);
    expect(userListViewModel.users[0].name).toBe('John Doe');
  });
});
```

### Advanced Dependency Resolution

#### Complex Dependency Graphs

```javascript
describe('Complex Dependency Resolution', () => {
  // Multiple interdependent services
  class LoggerService {
    log(message) {
      console.log(message);
    }
  }

  class CacheService {
    constructor(loggerService) {
      this.logger = loggerService;
    }

    set(key, value) {
      this.logger.log(`Caching ${key}`);
      return true;
    }
  }

  class DataService {
    static inject = [CacheService, LoggerService];

    constructor(cacheService, loggerService) {
      this.cache = cacheService;
      this.logger = loggerService;
    }

    processData(data) {
      this.logger.log('Processing data');
      this.cache.set('lastProcessed', data);
      return data;
    }
  }

  let container;
  let mockLogger;
  let mockCache;
  let dataService;

  beforeEach(() => {
    // Create container
    container = new Container();

    // Create mock dependencies
    mockLogger = {
      log: jasmine.createSpy('log')
    };

    mockCache = {
      set: jasmine.createSpy('set').and.returnValue(true)
    };

    // Register mock dependencies
    container.registerInstance(LoggerService, mockLogger);
    container.registerInstance(CacheService, mockCache);

    // Resolve data service
    dataService = container.get(DataService);
  });

  it('should resolve complex dependency graph', () => {
    const testData = { id: 1, name: 'Test' };
    
    // Process data
    dataService.processData(testData);

    // Verify dependency interactions
    expect(mockLogger.log).toHaveBeenCalledWith('Processing data');
    expect(mockCache.set).toHaveBeenCalledWith('lastProcessed', testData);
  });
});
```

### Custom Injection Strategies

#### Conditional Dependency Injection

```javascript
describe('Custom Injection Strategies', () => {
  // Interface-like base class
  class DatabaseService {
    connect() {}
    query() {}
  }

  // Concrete implementations
  class MySqlService extends DatabaseService {
    connect() { return 'MySQL Connected'; }
    query() { return [{ id: 1, name: 'MySQL Result' }]; }
  }

  class PostgreSqlService extends DatabaseService {
    connect() { return 'PostgreSQL Connected'; }
    query() { return [{ id: 1, name: 'PostgreSQL Result' }]; }
  }

  // Conditional service resolver
  class DatabaseServiceResolver {
    static resolve(container, environment) {
      switch(environment) {
        case 'production':
          return container.get(MySqlService);
        case 'development':
        default:
          return container.get(PostgreSqlService);
      }
    }
  }

  // View-model using dynamic service
  class DataViewModel {
    static inject = [Container];

    constructor(container) {
      this.databaseService = DatabaseServiceResolver.resolve(
        container, 
        process.env.NODE_ENV || 'development'
      );
    }

    getData() {
      return this.databaseService.query();
    }
  }

  let container;
  let dataViewModel;

  beforeEach(() => {
    container = new Container();

    // Register concrete implementations
    container.registerSingleton(MySqlService, MySqlService);
    container.registerSingleton(PostgreSqlService, PostgreSqlService);

    // Create view-model
    dataViewModel = container.get(DataViewModel);
  });

  it('should resolve correct database service based on environment', () => {
    const data = dataViewModel.getData();

    expect(data).toBeTruthy();
    expect(data[0].name).toContain('PostgreSQL Result');
  });
});
```

### Dynamic Dependency Registration

#### Runtime Dependency Modification

```javascript
describe('Dynamic Dependency Registration', () => {
  class ConfigurationService {
    constructor(settings = {}) {
      this.settings = settings;
    }

    getSetting(key) {
      return this.settings[key];
    }
  }

  class DynamicConfigurationResolver {
    static resolve(container, initialSettings = {}) {
      // Remove any existing registrations
      container.unregister(ConfigurationService);

      // Register new instance with dynamic settings
      const newInstance = new ConfigurationService(initialSettings);
      container.registerInstance(ConfigurationService, newInstance);

      return newInstance;
    }
  }

  let container;

  beforeEach(() => {
    container = new Container();
  });

  it('should dynamically update dependency configuration', () => {
    // Initial configuration
    const initialConfig = { apiUrl: 'http://initial.com' };
    const firstInstance = DynamicConfigurationResolver.resolve(
      container, 
      initialConfig
    );

    expect(firstInstance.getSetting('apiUrl')).toBe('http://initial.com');

    // Update configuration
    const updatedConfig = { apiUrl: 'http://updated.com' };
    const secondInstance = DynamicConfigurationResolver.resolve(
      container, 
      updatedConfig
    );

    expect(secondInstance.getSetting('apiUrl')).toBe('http://updated.com');
  });
});
```
