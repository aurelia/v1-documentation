# Viewports

Aurelia supports named viewports, allowing you to define multiple `<router-view>` elements in your template. This is particularly useful for complex layouts where you want to load different components into different areas of the page.

In your template:

```markup
<template>
  <div class="left-panel">
    <router-view name="left"></router-view>
  </div>
  <div class="main-content">
    <router-view name="main"></router-view>
  </div>
  <div class="right-panel">
    <router-view name="right"></router-view>
  </div>
</template>
```

Then in your route configuration:

```typescript
config.map([
  { 
    route: 'dashboard', 
    name: 'dashboard', 
    viewPorts: {
      left: { moduleId: 'dashboard/nav' },
      main: { moduleId: 'dashboard/main' },
      right: { moduleId: 'dashboard/widgets' }
    } 
  }
]);
```

You can also set defaults for viewports:

```typescript
config.useViewPortDefaults({
  right: { moduleId: 'common/rightPanel' }
});
```

This will load the 'common/rightPanel' module into the 'right' viewport for any route that doesn't explicitly specify a different module for that viewport.
