# Lifecycle hooks

Route components (view-models) can implement several lifecycle hooks to respond to navigation events:

## canActivate

Called before activating the route. It can be used for authorization or to check if data is available.

```typescript
canActivate(params: any, routeConfig: RouteConfig, navigationInstruction: NavigationInstruction): boolean | Promise<boolean> {
  if (params.id === '42') {
    return true;
  }
  return false;
}
```

## activate

Called when the route is activated. Use this to load data or perform setup.

```typescript
activate(params: any, routeConfig: RouteConfig, navigationInstruction: NavigationInstruction): void | Promise<void> {
  this.userId = params.id;
  return this.userService.getUser(this.userId).then(user => this.user = user);
}
```

## canDeactivate

Called before navigating away from the route. Can be used to prevent navigation if there are unsaved changes.

```typescript
canDeactivate(): boolean | Promise<boolean> {
  if (this.hasUnsavedChanges) {
    return confirm('You have unsaved changes. Are you sure you want to leave?');
  }
  return true;
}
```

## deactivate

Called when navigating away from the route. Use this for cleanup.

```typescript
deactivate(): void {
  this.subscription.dispose();
}
```
