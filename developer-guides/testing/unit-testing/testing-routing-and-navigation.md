# Testing Routing and Navigation

Routing is a critical part of any single-page application, and Aurelia provides robust routing capabilities. Testing routes and navigation requires specialized techniques to simulate and verify routing behavior without actually changing the browser's location.

### Mocking the Router

#### Router Mocking Strategies

```javascript
describe('Router Mocking', () => {
  let mockRouter;

  beforeEach(() => {
    // Create a comprehensive mock router
    mockRouter = {
      // Navigation methods
      navigate: jasmine.createSpy('navigate'),
      navigateBack: jasmine.createSpy('navigateBack'),
      navigateToRoute: jasmine.createSpy('navigateToRoute'),

      // Current navigation state
      currentInstruction: null,
      currentNavigationTracker: null,

      // Route configuration
      routes: [
        { route: 'home', moduleId: 'home' },
        { route: 'users', moduleId: 'users' }
      ],

      // Simulation methods
      createNavigationInstruction: (url) => ({
        fragment: url,
        queryString: '',
        params: {}
      }),

      // Additional router-like methods
      isNavigating: false,
      isActive: true
    };
  });

  it('can mock router navigation', () => {
    mockRouter.navigate('/home');
    expect(mockRouter.navigate).toHaveBeenCalledWith('/home');
  });
});
```

### Testing Route Configuration

#### Route Configuration Verification

```javascript
describe('Route Configuration Testing', () => {
  let router;

  beforeEach(() => {
    // Setup a test router configuration
    class AppRouter {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'Test App';
        config.map([
          { route: ['', 'home'], name: 'home', moduleId: 'home/index' },
          { route: 'users', name: 'users', moduleId: 'users/index' },
          { 
            route: 'user/:id', 
            name: 'user-detail', 
            moduleId: 'users/detail',
            nav: true
          }
        ]);
      }
    }

    // Create router instance
    router = new AppRouter();
    router.configureRouter({}, { routes: [] });
  });

  it('should configure routes correctly', () => {
    const configuredRoutes = router.router.routes;

    expect(configuredRoutes.length).toBe(3);
    
    // Verify specific route configurations
    const homeRoute = configuredRoutes.find(r => r.name === 'home');
    expect(homeRoute).toBeTruthy();
    expect(homeRoute.moduleId).toBe('home/index');
    expect(homeRoute.route).toContain('');
    expect(homeRoute.route).toContain('home');

    const userDetailRoute = configuredRoutes.find(r => r.name === 'user-detail');
    expect(userDetailRoute).toBeTruthy();
    expect(userDetailRoute.route).toBe('user/:id');
    expect(userDetailRoute.nav).toBe(true);
  });
});
```

### Navigation Behavior Testing

#### Simulating Navigation

```javascript
describe('Navigation Behavior', () => {
  let router;
  let navigationTracker;

  beforeEach(() => {
    // Create a more advanced router mock
    class MockRouter {
      constructor() {
        this.currentInstruction = null;
        this.navigationHistory = [];
      }

      navigate(url, options = {}) {
        const instruction = this.createNavigationInstruction(url);
        this.currentInstruction = instruction;
        this.navigationHistory.push(instruction);
        return Promise.resolve(instruction);
      }

      createNavigationInstruction(url) {
        return {
          fragment: url,
          queryString: '',
          params: this.parseParams(url)
        };
      }

      parseParams(url) {
        // Simple param parsing
        const paramMatch = url.match(/\/(\w+)$/);
        return paramMatch ? { id: paramMatch[1] } : {};
      }
    }

    router = new MockRouter();
  });

  it('should track navigation', async () => {
    // Simulate navigation
    await router.navigate('/users/123');

    // Verify navigation details
    expect(router.currentInstruction.fragment).toBe('/users/123');
    expect(router.currentInstruction.params.id).toBe('123');
    expect(router.navigationHistory.length).toBe(1);
  });

  it('should handle multiple navigations', async () => {
    await router.navigate('/home');
    await router.navigate('/users');
    await router.navigate('/users/456');

    expect(router.navigationHistory.length).toBe(3);
    expect(router.currentInstruction.fragment).toBe('/users/456');
  });
});
```

### Advanced Route Testing

#### Conditional Navigation and Guards

```javascript
describe('Navigation Guards and Conditions', () => {
  let authService;
  let router;

  beforeEach(() => {
    // Mock authentication service
    authService = {
      isAuthenticated: jasmine.createSpy('isAuthenticated').and.returnValue(false),
      login: jasmine.createSpy('login')
    };

    // Create a router with navigation guard
    class SecureRouter {
      constructor(authService) {
        this.authService = authService;
        this.currentInstruction = null;
      }

      async canAccess(instruction) {
        if (!this.authService.isAuthenticated()) {
          // Redirect to login if not authenticated
          this.currentInstruction = null;
          this.authService.login();
          return false;
        }
        this.currentInstruction = instruction;
        return true;
      }

      async navigate(url) {
        const instruction = { fragment: url };
        const canNavigate = await this.canAccess(instruction);
        return canNavigate ? instruction : null;
      }
    }

    router = new SecureRouter(authService);
  });

  it('should prevent navigation when not authenticated', async () => {
    const result = await router.navigate('/admin');

    expect(authService.isAuthenticated).toHaveBeenCalled();
    expect(authService.login).toHaveBeenCalled();
    expect(result).toBeNull();
    expect(router.currentInstruction).toBeNull();
  });

  it('should allow navigation when authenticated', async () => {
    authService.isAuthenticated.and.returnValue(true);

    const result = await router.navigate('/admin');

    expect(authService.isAuthenticated).toHaveBeenCalled();
    expect(authService.login).not.toHaveBeenCalled();
    expect(result.fragment).toBe('/admin');
  });
});
```

### Testing Navigation Components

#### Navigation View-Model Testing

```javascript
describe('Navigation View-Model', () => {
  let navigationViewModel;
  let mockRouter;

  beforeEach(() => {
    mockRouter = {
      navigate: jasmine.createSpy('navigate'),
      currentInstruction: null
    };

    class NavigationViewModel {
      constructor(router) {
        this.router = router;
      }

      navigateTo(route) {
        this.router.navigate(route);
      }

      getCurrentRoute() {
        return this.router.currentInstruction 
          ? this.router.currentInstruction.fragment 
          : null;
      }
    }

    navigationViewModel = new NavigationViewModel(mockRouter);
  });

  it('should navigate to specified route', () => {
    navigationViewModel.navigateTo('/users');

    expect(mockRouter.navigate).toHaveBeenCalledWith('/users');
  });

  it('should track current route', () => {
    mockRouter.currentInstruction = { fragment: '/home' };

    expect(navigationViewModel.getCurrentRoute()).toBe('/home');
  });
});
```
