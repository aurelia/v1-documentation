# Redirecting routes

You can set up redirects in your route configuration to automatically navigate users from one route to another.

## Basic Redirect

```typescript
config.map([
  { route: '', redirect: 'home' },
  { route: 'home', name: 'home', moduleId: PLATFORM.moduleName('home/index') }
]);
```

This will redirect the root URL to the 'home' route.

## Redirect in Route Config

You can also use the `redirect` property in more complex route configurations:

```typescript
config.map([
  { 
    route: 'legacy-route', 
    name: 'legacy', 
    redirect: 'new-route'
  },
  {
    route: 'new-route',
    name: 'new',
    moduleId: PLATFORM.moduleName('new/index')
  }
]);
```
