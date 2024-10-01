# Router configuration

Aurelia's router allows you to configure how routes are defined, but also react when navigting throughout the application.

A basic router configuration looks like this:

```typescript
import {RouterConfiguration, Router} from 'aurelia-router';

export class App {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router): void {
    this.router = router;
    config.title = 'Aurelia';
    config.map([
      { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
      { route: 'users',            name: 'users',      moduleId: 'users/index', nav: true, title: 'Users' },
      { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
      { route: 'files/*path',      name: 'files',      moduleId: 'files/index', nav: 0,    title: 'Files', href:'#files' }
    ]);
  }
}
```

* `config.map()` adds route(s) to the router. Although only `route`, `name`, `moduleId`, `href` and `nav` are shown above there are other properties that can be included in a route. The interface name for a route is `RouteConfig`. You can also use `config.mapRoute()` to add a single route.
* `route` - is the pattern to match against incoming URL fragments. It can be a string or array of strings. The route can contain parameterized routes or wildcards as well.
  * Parameterized routes match against a string with a `:token` parameter (ie: 'users/:id/detail'). An object with the token parameter's name is set as property and passed as a parameter to the route view-model's `activate()` function.
  * A parameter can be made optional by appending a question mark `:token?` (ie: `users/:id?/detail` would match both `users/3/detail` and `users/detail`). When an optional parameter is missing from the url, the property passed to `activate()` is `undefined`.
  * Wildcard routes are used to match the "rest" of a path (ie: files/\*path matches files/new/doc or files/temp). An object with the rest of the URL after the segment is set as the `path` property and passed as a parameter to `activate()` as well.
* `name` - is a friendly name that you can use to reference the route with, particularly when using route generation.
* `moduleId` - is the id (usually a relative path) of the module that exports the component that should be rendered when the route is matched.
* `href` - is a conditionally optional property. If it is not defined then route is used. If route has segments then href is required as in the case of files because the router does not know how to fill out the parameterized portions of the pattern.
* `nav` - is a boolean or number property. When set to true the route will be included in the router's navigation model. When specified as number, the value will be used in sorting the routes. This makes it easier to create a dynamic menu or similar elements. The navigation model will be available as array of `NavModel` in `router.navigation`. These are the available properties in `NavModel`:
  * `isActive` flag which will be true when the associated route is active.
  * `title` which will be prepended in html title when the associated route is active.
  * `href` can be used on `a` tag.
  * `config` is the object defined in `config.map`.
  * `settings` is equal to the property `settings` of `config` object.
  * `router` is a reference for AppRouter.
  * Other properties includes `relativeHref` and `order`.
* `title` - is the text to be displayed as the document's title (appears in your browser's title bar or tab). It is combined with the `router.title` and the title from any child routes.
* `titleSeparator` - is the text that will be used to join the `title` and any active `route.title`s. The default value is `' | '`.

## Navigation States

The router contains a number of additional properties that indicate the current status of router navigation. These properties are only set on the base router, i.e. not in child routers. Additionally, these properties are all with respect to browser history which extends past the lifecycle of the router itself.

* `router.isNavigating`: `true` if the router is currently processing a navigation.
* `router.isNavigatingFirst`: `true` if the router is navigating into the app for the first time in the browser session.
* `router.isNavigatingNew`: `true` if the router is navigating to a page instance not in the browser session history. This is triggered when the user clicks a link or the navigate function is called explicitly.
* `router.isNavigatingForward`: `true` if the router is navigating forward in the browser session history. This is triggered when the user clicks the forward button in their browser.
* `router.isNavigatingBack`: `true` if the router is navigating back in the browser session history. This is triggered when the user clicks the back button in their browser or when the `navigateBack()` function is called.
* `router.isNavigatingRefresh`: `true` if the router is navigating due to a browser refresh.
* `router.isExplicitNavigation`: `true` if the router is navigating due to explicit call to navigate function(s).
* `router.isExplicitNavigationBack`: `true` if the router is navigating due to explicit call to navigateBack function.

## Webpack Configuration

When using Webpack, it is necessary to help Aurelia create a path that the loader can resolve. This is done by wrapping the `moduleId` string with `PLATFORM.moduleName()`. You can import `PLATFORM` into your project from either `aurelia-framework` or `aurelia-pal`.

```typescript
import { PLATFORM } from "aurelia-framework";

export class App {
  configureRouter(config: RouterConfiguration, router: Router): void {
    config.map([
      { route: ['', 'home'],   name: 'home',    moduleId: PLATFORM.moduleName('home') }
    ]);
  }
}
```

## Options

### Push State

Set `config.options.pushState` to `true` to activate the push state and add a base tag to the head of your HTML document. If you're using JSPM, RequireJS or a similar module loader, you will also need to configure it with a base URL, corresponding to your base tag's `href`. Finally, set the `config.options.root` to match your base tag's setting.

```typescript
import {RouterConfiguration, Router} from 'aurelia-router';

export class App {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router): void {
    this.router = router;
    config.title = 'Aurelia';
    config.options.pushState = true;
    config.options.root = '/';
    config.map([
      { route: ['', 'home'], name: 'home',  moduleId: 'home/index' },
      { route: 'users',      name: 'users', moduleId: 'users/index', nav: true, title: 'Users' }
    ]);
  }
}
```

{% hint style="warning" %}
PushState requires server-side support. This configuration is different depending on your server setup. For example, if you are using Webpack DevServer, you'll want to set the `devServer` `historyApiFallback` option to `true`. If you are using ASP.NET Core, you'll want to call `routes.MapSpaFallbackRoute` in your startup code. See you preferred server technology's documentation for more information on how to allow 404s to be handled on the client with push state.
{% endhint %}
