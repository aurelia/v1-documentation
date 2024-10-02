# 404 pages

Handling non-existing routes gracefully is essential for a good user experience and SEO.

## Configuration

Map unknown routes to a dedicated 404 module in your router configuration:

```javascript
import { PLATFORM } from 'aurelia-pal';
import { Router, RouterConfiguration } from 'aurelia-router';

export class App {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router) {
    config.title = 'Aurelia';
    config.options.pushState = true;
    config.options.root = '/';

    // Define your routes here
    config.map([
      { route: ['', 'home'], name: 'home', moduleId: PLATFORM.moduleName('home'), nav: true, title: 'Home' },
      // Other routes
    ]);

    // Map unknown routes to the 404 page
    config.mapUnknownRoutes(PLATFORM.moduleName('not-found'));

    this.router = router;
  }
}
```

## 404 Module

Create a `not-found` module to display a user-friendly 404 page.

```javascript
export class NotFound {
  constructor() {
    this.message = 'Page not found';
  }
}
```

```html
<template>
  <h1>${message}</h1>
  <p>The page you are looking for does not exist.</p>
</template>
```

## Server Handling

With SSR, when a user requests a non-existing route, the server will render the `not-found` module just like any other page, ensuring consistent behavior across client and server.
