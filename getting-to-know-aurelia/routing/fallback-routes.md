# Fallback Routes

Fallback routes handle situations where navigation cannot be completed — such as when a user tries to access a route that doesn't exist, navigation is rejected by a pipeline step, or a user lacks permission to view a route.

## Setting a Fallback Route

Configure a fallback route in your router configuration using `config.fallbackRoute()`:

```javascript
export class App {
  configureRouter(config, router) {
    config.fallbackRoute('home');

    config.map([
      { route: ['', 'home'], name: 'home', moduleId: 'home/index' },
      { route: 'about', name: 'about', moduleId: 'about/index' }
    ]);
  }
}
```

When a user navigates to an unrecognized route (e.g., `/nonexistent`), the router redirects them to the `home` route instead of showing a blank page.

## How Fallback Routes Work

The fallback route is triggered in these scenarios:

1. **Unknown routes** — The user navigates to a URL that doesn't match any configured route
2. **Rejected navigation** — A router pipeline step (such as an authorization check) cancels navigation and there is no previous location to return to
3. **Failed module loading** — The `moduleId` for a route cannot be resolved

## Using Fallback with mapUnknownRoutes

For more control over unknown routes, use `mapUnknownRoutes()` alongside or instead of `fallbackRoute()`:

```javascript
export class App {
  configureRouter(config, router) {
    config.map([
      { route: ['', 'home'], name: 'home', moduleId: 'home/index' }
    ]);

    // Redirect all unknown routes to a specific module
    config.mapUnknownRoutes('not-found');
  }
}
```

You can also provide a function for dynamic handling:

```javascript
config.mapUnknownRoutes(instruction => {
  // Log the unknown route attempt
  console.warn(`Unknown route: ${instruction.fragment}`);

  // Return a route configuration
  return { moduleId: 'not-found', route: '' };
});
```

## Combining with Authorization

Fallback routes work well with router pipeline authorization. When an `authorize` step rejects navigation, the fallback route catches the user:

```javascript
export class App {
  configureRouter(config, router) {
    config.fallbackRoute('login');
    config.addPipelineStep('authorize', AuthorizeStep);

    config.map([
      { route: 'login', name: 'login', moduleId: 'auth/login' },
      { route: 'dashboard', name: 'dashboard', moduleId: 'dashboard/index', settings: { auth: true } }
    ]);
  }
}
```

{% hint style="info" %}
The `fallbackRoute` value should match a route name or route pattern that is defined in your route map. If the fallback route itself is invalid, the router will not be able to recover from failed navigation.
{% endhint %}

## Creating a Custom 404 Page

For a better user experience, create a dedicated "not found" component:

```javascript
// not-found.js
export class NotFound {
  activate(params, routeConfig, navigationInstruction) {
    this.attemptedUrl = navigationInstruction.fragment;
  }
}
```

```html
<!-- not-found.html -->
<template>
  <h1>Page Not Found</h1>
  <p>The page <strong>${attemptedUrl}</strong> could not be found.</p>
  <a route-href="route: home">Return to Home</a>
</template>
```

```javascript
// In your router config
config.mapUnknownRoutes('not-found');
```
