# Layouts

Layouts in Aurelia allow you to create a consistent structure around your views, similar to master pages in traditional web frameworks.

## Basic Layout Usage

To use a layout, you can specify it on the `<router-view>` element:

```markup
<template>
  <router-view layout-view="layouts/default.html"></router-view>
</template>
```

Then, create your layout view (e.g., `layouts/default.html`):

```markup
<template>
  <header>Common Header</header>
  <slot></slot>
  <footer>Common Footer</footer>
</template>
```

## Layout View Model and Model

You can also specify a layout view model and model:

```markup
<router-view 
  layout-view="layouts/default.html"
  layout-view-model="layouts/default"
  layout-model="{ title: 'My App' }">
</router-view>
```

## Layouts in Route Configuration

Layouts can also be specified in route configuration:

```typescript
config.map([
  { 
    route: 'users',
    name: 'users',
    moduleId: PLATFORM.moduleName('users/index'),
    layoutView: PLATFORM.moduleName('layouts/default.html'),
    layoutViewModel: PLATFORM.moduleName('layouts/default'),
    layoutModel: { title: 'Users' }
  }
]);
```

## View Ports in Layouts

Layouts can define named view ports that routes can target:

```markup
<!-- layouts/two-column.html -->
<template>
  <div class="left-column">
    <slot name="left"></slot>
  </div>
  <div class="right-column">
    <slot name="right"></slot>
  </div>
</template>
```

```typescript
config.map([
  { 
    route: 'users',
    name: 'users',
    layoutView: PLATFORM.moduleName('layouts/two-column.html'),
    viewPorts: {
      left: { moduleId: PLATFORM.moduleName('users/list') },
      right: { moduleId: PLATFORM.moduleName('users/detail') }
    }
  }
]);
```
