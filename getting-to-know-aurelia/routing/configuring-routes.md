# Configuring routes

Routes are configured using the `config.map()` method. Each route is an object with several possible properties:

* `route`: The URL pattern to match. Can be a string, array of strings, or a regular expression.
* `name`: A unique name for the route, used in generating URLs.
* `moduleId`: The module to load for this route. This is typically a path to a view-model file.
* `nav`: If true, includes this route in the router's navigation model (useful for generating menus).
* `title`: The title to use for this route. Will be combined with the router's title.
* `settings`: An object for storing arbitrary data associated with the route.
* `viewPorts`: An object defining which components should load into named router views.

## Basic Route Configuration

Here's an example with various route configurations:

```typescript
config.map([
  { route: ['', 'home'], name: 'home', moduleId: 'home/index' },
  { route: 'users', name: 'users', moduleId: 'users/index', nav: true, title: 'Users' },
  { route: 'users/:id', name: 'userDetail', moduleId: 'users/detail' },
  { route: /^\/blog\/\d{4}\/\d{2}\/\d{2}\//, name: 'blog-post', moduleId: 'blog/post' },
  { 
    route: 'files*path', 
    name: 'files',
    moduleId: 'files/index',
    nav: true,
    title: 'Files',
    href: '#files'
  }
]);
```

## Using PLATFORM.moduleName for Webpack

When using Webpack with Aurelia, it's crucial to use `PLATFORM.moduleName()` for the `moduleId` property. This helps Webpack understand the dependencies and include them in the bundle correctly.

First, import PLATFORM from aurelia-framework or aurelia-pal:

```typescript
import { PLATFORM } from 'aurelia-framework';
```

Then, wrap each `moduleId` value with `PLATFORM.moduleName()`:

```typescript
config.map([
  { route: ['', 'home'], name: 'home', moduleId: PLATFORM.moduleName('home/index') },
  { route: 'users', name: 'users', moduleId: PLATFORM.moduleName('users/index'), nav: true, title: 'Users' },
  { route: 'users/:id', name: 'userDetail', moduleId: PLATFORM.moduleName('users/detail') },
  { 
    route: 'files*path', 
    name: 'files',
    moduleId: PLATFORM.moduleName('files/index'),
    nav: true,
    title: 'Files',
    href: '#files'
  }
]);
```

This approach ensures that Webpack can properly analyze and include all necessary modules in your bundle. It's especially important when using dynamic imports or lazy loading in your Aurelia application.

## Advanced Configuration with PLATFORM.moduleName

For more complex scenarios, such as lazy loading or conditional module loading, you can use `PLATFORM.moduleName()` with additional options:

```typescript
config.map([
  {
    route: 'admin',
    name: 'admin',
    moduleId: PLATFORM.moduleName('admin/index', 'admin'),
    nav: true,
    title: 'Admin'
  }
]);
```

In this example, the 'admin' chunk name is specified as the second argument to `PLATFORM.moduleName()`. This can be used to control how Webpack splits your code into chunks for lazy loading.

{% hint style="warning" %}
Remember, using `PLATFORM.moduleName()` is crucial when working with Webpack, as it allows for proper static analysis and code splitting in your Aurelia application.
{% endhint %}
