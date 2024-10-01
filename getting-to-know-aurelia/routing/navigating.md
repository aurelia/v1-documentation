# Navigating

Aurelia provides several ways to navigate between routes:

## Programmatic Navigation

Use the `router.navigate()` method:

```typescript
this.router.navigate('users');
```

You can also pass parameters:

```typescript
this.router.navigate('users/42');
```

Or use named routes with parameters:

```typescript
this.router.navigateToRoute('userDetail', { id: 42 });
```

## Declarative Navigation

In your templates, you can use the `route-href` attribute:

```markup
<a route-href="route: users">Users</a>
<a route-href="route: userDetail; params.bind: { id: user.id }">View User</a>
```

You can also use the `router-view` element's `route` attribute for creating navigation menus:

```markup
<ul>
  <li repeat.for="nav of router.navigation">
    <a href.bind="nav.href">${nav.title}</a>
  </li>
</ul>
```
