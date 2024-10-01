# Getting started

The router in Aurelia 1 is a powerful and flexible system for managing navigation in your single-page applications (SPAs). It allows you to create a multi-view application with deep linking and powerful navigation features.&#x20;

Here's a step-by-step guide to get you started with routing in Aurelia:

## Set Up Your App Component

Your app component will serve as your application's main container and host the router.

In `src/app.html`:

```markup
<template>
  <h1>${router.title}</h1>
  
  <nav>
    <ul>
      <li repeat.for="nav of router.navigation">
        <a href.bind="nav.href">${nav.title}</a>
      </li>
    </ul>
  </nav>

  <router-view></router-view>
</template>
```

**This template includes:**

* A title that the router will set
* A navigation menu that will be automatically populated based on your route configuration
* A `<router-view>` element, which serves as a placeholder where route components will be rendered

## Configure the Router

In `src/app.ts`, implement the `configureRouter()` method:

```typescript
import { RouterConfiguration, Router } from 'aurelia-router';
import { PLATFORM } from 'aurelia-framework';

export class App {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router): void {
    this.router = router;
    config.title = 'My Aurelia App';
    config.map([
      { route: ['', 'home'], name: 'home', moduleId: PLATFORM.moduleName('home/index'), nav: true, title: 'Home' },
      { route: 'users', name: 'users', moduleId: PLATFORM.moduleName('users/index'), nav: true, title: 'Users' }
    ]);
  }
}
```

**This configuration:**

* Sets the base title for your application
* Defines two routes: a home route and a user route
* Uses `PLATFORM.moduleName()` for webpack compatibility
* Marks both routes as `nav: true` so they appear in the navigation menu

## Create Route Components

For each route, create a corresponding component. For example:

In `src/home/index.html`:

```markup
<template>
  <h2>Welcome to the Home Page</h2>
</template>
```

In `src/home/index.ts`:

```typescript
export class Home {
  // Component logic here
}
```

Create similar files for the Users route.

## The Basics

* The `<router-view>` element in your app template is where route components will be rendered.
* The `configureRouter()` method is where you set up your routes and any router-specific configuration.
* Each route in the `config.map()` array defines a different "page" or view in your application.
* The `moduleId` specifies which component should be loaded for each route.
* Setting `nav: true` includes the route in the `router.navigation` array, which you can use to build navigation menus.

{% hint style="info" %}
The `<router-view>` element is required when using the router.
{% endhint %}

