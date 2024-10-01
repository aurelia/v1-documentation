# Fallback Routes

Fallback routes are used when navigation is rejected, such as when a user doesn't have permission to access a route.

```typescript
config.fallbackRoute('home');
```

This sets 'home' as the fallback route. If navigation is rejected and there's no previous location to return to, the router will navigate to the 'home' route.
