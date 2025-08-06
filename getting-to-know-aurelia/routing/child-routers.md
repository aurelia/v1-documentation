# Child Routers

Aurelia supports nested routing through child routers, allowing you to create hierarchical navigation structures. This is useful for organizing complex applications with multiple levels of navigation.

## Basic Child Router Setup

To create a child router, implement `configureRouter()` in any component that needs nested routing. The component must have a `<router-view>` element in its template.

### Parent Router Configuration

```typescript
// app.ts
import { RouterConfiguration, Router } from 'aurelia-router';

export class App {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router): void {
    this.router = router;
    config.title = 'My App';
    config.map([
      { route: ['', 'welcome'], name: 'welcome', moduleId: 'welcome', nav: true, title: 'Welcome' },
      { route: 'users*', name: 'users', moduleId: 'users/users', nav: true, title: 'Users' }
    ]);
  }
}
```

```html
<!-- app.html -->
<template>
  <nav>
    <a repeat.for="nav of router.navigation" href.bind="nav.href">${nav.title}</a>
  </nav>
  <router-view></router-view>
</template>
```

### Child Router Component

```typescript
// users/users.ts
import { RouterConfiguration, Router } from 'aurelia-router';

export class Users {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router): void {
    this.router = router;
    config.map([
      { route: ['', 'list'], name: 'list', moduleId: './list', nav: true, title: 'User List' },
      { route: 'detail/:id', name: 'detail', moduleId: './detail', nav: false, title: 'User Detail' },
      { route: 'create', name: 'create', moduleId: './create', nav: true, title: 'Create User' }
    ]);
  }
}
```

```html
<!-- users/users.html -->
<template>
  <div class="users-nav">
    <a repeat.for="nav of router.navigation" href.bind="nav.href">${nav.title}</a>
  </div>
  <router-view></router-view>
</template>
```

## Route Pattern Matching

Notice the `*` wildcard in the parent route `'users*'`. This tells the parent router to match `/users` and any path that starts with `/users/`, passing the remaining segments to the child router.

### URL Structure Examples

With the above configuration:
- `/users` → Users component, child router shows list
- `/users/list` → Users component, child router shows list  
- `/users/detail/123` → Users component, child router shows detail for user 123
- `/users/create` → Users component, child router shows create form

## Multiple Levels of Nesting

You can nest routers multiple levels deep:

```typescript
// users/detail.ts
export class UserDetail {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router): void {
    this.router = router;
    config.map([
      { route: ['', 'info'], name: 'info', moduleId: './info', nav: true, title: 'Info' },
      { route: 'settings', name: 'settings', moduleId: './settings', nav: true, title: 'Settings' },
      { route: 'history', name: 'history', moduleId: './history', nav: true, title: 'History' }
    ]);
  }

  activate(params: any) {
    this.userId = params.id;
  }
}
```

This creates URLs like `/users/detail/123/settings`.

## Child Router Navigation

### Programmatic Navigation

Each child router has its own navigation methods:

```typescript
export class Users {
  router: Router;

  // Navigate within this child router
  goToUserDetail(userId: string) {
    this.router.navigateToRoute('detail', { id: userId });
  }

  // Navigate to parent router route
  goToWelcome() {
    this.router.parent.navigateToRoute('welcome');
  }
}
```

### Accessing Parent Router

Child routers maintain a reference to their parent:

```typescript
export class ChildComponent {
  router: Router;

  someMethod() {
    // Access parent router
    const parentRouter = this.router.parent;
    
    // Navigate in parent
    parentRouter.navigateToRoute('welcome');
    
    // Access root/app router
    let rootRouter = this.router;
    while (rootRouter.parent) {
      rootRouter = rootRouter.parent;
    }
  }
}
```

## Child Router with ViewPorts

Child routers can also use multiple viewports:

```typescript
// users/users.ts
export class Users {
  configureRouter(config: RouterConfiguration, router: Router): void {
    config.map([
      { 
        route: 'dashboard', 
        name: 'dashboard',
        viewPorts: {
          content: { moduleId: './dashboard' },
          sidebar: { moduleId: './user-sidebar' }
        }
      }
    ]);
  }
}
```

```html
<!-- users/users.html -->
<template>
  <div class="layout">
    <router-view name="sidebar" class="sidebar"></router-view>
    <router-view name="content" class="content"></router-view>
  </div>
</template>
```

## Router Lifecycle in Child Routers

Child router components follow the same lifecycle as regular components:

```typescript
export class ChildRouterComponent {
  // Called when child router is navigated to
  canActivate(params: any, routeConfig: any, navigationInstruction: any) {
    // Authorization logic
    return true;
  }

  activate(params: any, routeConfig: any, navigationInstruction: any) {
    // Setup logic
    console.log('Child router activated');
  }

  canDeactivate() {
    // Can we leave this child router?
    return confirm('Are you sure you want to leave?');
  }

  deactivate() {
    // Cleanup logic
    console.log('Child router deactivated');
  }
}
```

## Navigation States in Child Routers

Navigation state properties are only available on the root `AppRouter`, not on child routers:

```typescript
export class ChildComponent {
  router: Router;

  checkNavigationState() {
    // These properties are undefined on child routers
    console.log(this.router.isNavigating); // undefined
    
    // Access through root router
    let rootRouter = this.router;
    while (rootRouter.parent) {
      rootRouter = rootRouter.parent;
    }
    console.log(rootRouter.isNavigating); // true/false
  }
}
```

## Best Practices

### 1. Use Descriptive Route Names
```typescript
config.map([
  { route: 'settings/profile', name: 'profile-settings', moduleId: './profile-settings' },
  { route: 'settings/security', name: 'security-settings', moduleId: './security-settings' }
]);
```

### 2. Handle Deep Linking
Ensure child routes work correctly when accessed directly via URL:

```typescript
export class UserDetail {
  activate(params: any) {
    this.userId = params.id;
    // Load user data even when deep linking directly to this route
    return this.loadUserData(this.userId);
  }
}
```

### 3. Consider Router Hierarchy
Plan your router hierarchy to match your application's logical structure:

```
App Router
├── Home
├── Products*
│   ├── List
│   ├── Category/:id*
│   │   ├── Items
│   │   └── Filters
│   └── Detail/:id
└── Admin*
    ├── Users*
    │   ├── List
    │   └── Detail/:id
    └── Settings
```

### 4. Use Webpack PLATFORM.moduleName

When using Webpack, wrap moduleId values in child routers too:

```typescript
import { PLATFORM } from 'aurelia-framework';

export class Users {
  configureRouter(config: RouterConfiguration, router: Router): void {
    config.map([
      { route: ['', 'list'], name: 'list', moduleId: PLATFORM.moduleName('./list') },
      { route: 'detail/:id', name: 'detail', moduleId: PLATFORM.moduleName('./detail') }
    ]);
  }
}
```

Child routers provide powerful capabilities for organizing complex Aurelia applications with hierarchical navigation structures, enabling clean separation of concerns and intuitive URL schemes.