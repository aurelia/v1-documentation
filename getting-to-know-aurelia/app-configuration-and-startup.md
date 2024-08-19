---
description: How to get Aurelia started.
---

# App configuration and startup

Aurelia offers flexibility in configuring your application's startup process. You can choose between two main approaches:

1. Quick setup: Uses sensible defaults for a streamlined launch.
2. Verbose setup: Provides granular control over framework-specific behaviors.

Choose the method that best suits your project's needs and complexity.

Aurelia, like most platforms, has an entry point for code execution. The Quick Start introduces the `aurelia-app` attribute, which, when placed on an HTML element, triggers Aurelia's bootstrapper to load and databind `app.js/ts` and `app.html`, then inject them into the specified DOM element.

As your project grows, you may need to configure the framework or run code before displaying content. To achieve this, assign a value to the aurelia-app attribute, pointing to a configuration module. This module should export a configure function that Aurelia calls, passing an Aurelia object for framework configuration and UI management.

## Standard configuration

Here's an example configuration file demonstrating standard setup, equivalent to using `aurelia-app` without a value:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import {Aurelia} from 'aurelia-framework';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import {Aurelia} from 'aurelia-framework';

export function configure(aurelia: Aurelia): void {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}
{% endtabs %}

**For a quick setup with default settings, you have two simple steps:**

1. Call `standardConfiguration()` to set up the standard plugins.
2. Use `developmentLogging()` to enable console logging in debug mode.

This approach gives you a solid starting point without any fuss.

{% hint style="warning" %}
Certain Aurelia modules rely on the Platform Abstraction Layer (PAL) being initialized, which typically occurs during `aurelia.start()`. If you need these modules in your configuration, you'll have to manually initialize the PAL beforehand. For step-by-step instructions on how to do this, check out [this GitHub issue](https://github.com/aurelia/pal-browser/issues/17) that details the process of initializing the PAL before bootstrapping.
{% endhint %}

The `use` property on the Aurelia instance provides access to a `FrameworkConfiguration` object. This object offers various methods for fine-tuning your Aurelia setup. For instance, if you prefer to manually configure standard plugins and logging instead of using helper methods like `standardConfiguration()`, you can do so directly through the `FrameworkConfiguration`instance.&#x20;

**Here's an example:**

{% tabs %}
{% tab title="Javascript" %}
```javascript
import { LogManager } from 'aurelia-framework';
import { ConsoleAppender } from 'aurelia-logging-console';

LogManager.addAppender(new ConsoleAppender());
LogManager.setLevel(LogManager.logLevel.debug);

export function configure(aurelia) {
  aurelia.use
    .defaultBindingLanguage()
    .defaultResources()
    .history()
    .router()
    .eventAggregator();

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import { Aurelia, LogManager } from 'aurelia-framework';
import { ConsoleAppender } from 'aurelia-logging-console';

LogManager.addAppender(new ConsoleAppender());
LogManager.setLevel(LogManager.logLevel.debug);

export function configure(aurelia: Aurelia): void {
  aurelia.use
    .defaultBindingLanguage()
    .defaultResources()
    .history()
    .router()
    .eventAggregator();

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}
{% endtabs %}

This code sets up the core features of Aurelia, including:

* Default data-binding syntax (.bind, .trigger, etc.)
* Standard view resources (repeat, if, compose, etc.)
* Browser history integration
* Routing system
* Event aggregator for app-wide messaging

Aurelia's modular design allows you to customize this setup. For instance, if you don't need routing or the event aggregator but want debug logging, you can easily adjust the configuration like this:

{% tabs %}
{% tab title="Javascript" %}
```javascript
export function configure(aurelia) {
  aurelia.use
    .defaultBindingLanguage()
    .defaultResources()
    .developmentLogging();

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import { Aurelia } from 'aurelia-framework';

export function configure(aurelia: Aurelia): void {
  aurelia.use
    .defaultBindingLanguage()
    .defaultResources()
    .developmentLogging();

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}
{% endtabs %}

After configuring the framework, kick things off by calling `aurelia.start()`. This method returns a promise. When resolved, you're all set – the framework, including all plugins, is ready to go. At this point, it's safe to interact with services and begin rendering.

## Rendering the root component

To set the root component, use aurelia.setRoot(). By default, it uses the element with the aurelia-app attribute as your app's DOM host and app.ts/app.html as the root component source. Need something different? No problem. You can easily customize these settings:

{% tabs %}
{% tab title="Javascript" %}
```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  aurelia.start().then(() => aurelia.setRoot('my-root', document.getElementById('some-element'));
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import { Aurelia } from 'aurelia-framework';

export function configure(aurelia: Aurelia): void {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  aurelia.start().then(() => aurelia.setRoot('my-root', document.getElementById('some-element'));
}
```
{% endtab %}
{% endtabs %}

The `my-root.js/ts`/`my-root.html` files are loaded as the root component and inserted into the `some-element` in the HTML. This sets up the initial structure for your Aurelia application.

{% hint style="info" %}
The content of the app host element, the one marked with `aurelia-app` or passed to `Aurelia.prototype.setRoot`, will be replaced when `Aurelia.prototype.setRoot` completes.
{% endhint %}

{% hint style="warning" %}
When using the `<body>` element as the app host, bear in mind that any content added prior to the completion of `Aurelia.prototype.setRoot` will be removed.
{% endhint %}

## Bootstrapping older browsers

Aurelia primarily targets modern, evergreen browsers like Chrome, Firefox, IE11, and Safari 8+. However, we extend support to IE9 and above with additional polyfills.

**Install required polyfills:**

```sh
npm install -D raf mutationobserver-shim
```

Load these polyfills before initializing Aurelia.

Webpack users: Create an `ie-polyfill.js` file to import the polyfills.

{% code title="ie-polyfill.js" overflow="wrap" %}
```javascript
import 'mutationobserver-shim/MutationObserver'; // IE10 MutationObserver polyfill
import 'raf/polyfill'; // IE9 requestAnimationFrame polyfill
```
{% endcode %}

Once you've created the file, add it as the first entry in your aurelia-bootstrapper bundle. You'll find the bundle configuration in webpack.config.js. It should look something like this:

{% code title="webpack.config.js" %}
```javascript
entry: {
    'app': ['./ie-polyfill', 'aurelia-bootstrapper'],
```
{% endcode %}

This approach maintains Aurelia's performance for most users while accommodating those who need legacy browser support.

## Manual bootstrapping

While we've been using the aurelia-app attribute to bootstrap our application declaratively, it's not your only option. Aurelia also offers a manual bootstrapping process, giving you more control over your app's initialization.

When using Webpack, you can streamline your setup by replacing the aurelia-bootstrapper-webpack package. Instead, use the ./src/main entry file in the aurelia-bootstrapper bundle defined in webpack.config.js. Then, call the bootstrapper manually. This approach gives you more control over the bootstrapping process while keeping your configuration lean.

{% tabs %}
{% tab title="Javascript" %}
```javascript
import { bootstrap } from 'aurelia-bootstrapper';

bootstrap(async aurelia => {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  await aurelia.start();
  aurelia.setRoot(PLATFORM.moduleName('app'), document.body);
});
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import { Aurelia } from 'aurelia-framework';
import { bootstrap } from 'aurelia-bootstrapper';

bootstrap(async (aurelia: Aurelia) => {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  await aurelia.start();
  aurelia.setRoot(PLATFORM.moduleName('app'), document.body);
});
```
{% endtab %}
{% endtabs %}

The function you pass to the `bootstrap` method is the same as the `configure` function from the examples above.

## Making resources global

Aurelia views are self-contained, requiring you to import or require components much like you would in ES2015/TypeScript modules. However, for frequently used components, this can become repetitive. To address this, Aurelia allows you to declare certain "view resources" as global.

The `defaultResources()` configuration helper does this for common elements like repeat, if, and compose, making them available in all views. You can apply the same principle to your custom components.

For example, to make a custom element named `my-component` (located in your project's resources subfolder) globally accessible, you would:

{% tabs %}
{% tab title="Javascript" %}
```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .globalResources('resources/my-component');

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import { Aurelia } from 'aurelia-framework';

export function configure(aurelia: Aurelia): void {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .globalResources('resources/my-component');

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endtab %}
{% endtabs %}

## Organizing your app with features

n complex applications, you may have groups of components or related functionality that form a distinct "feature". These features are often managed by specific teams or developers. To facilitate this organization, Aurelia provides the "feature" concept.

**Features allow developers to:**

* Manage configuration and resources independently
* Maintain clear separation of concerns
* Simplify overall application structure

**Implementing a Feature**

1. Create a folder for your feature (e.g., 'my-feature')
2. Place all related components and code within this folder
3. Create an index.js/index.ts file at the root of the feature folder
4. Export a single 'configure' function from index.js/ts

**Example structure:**

```
my-feature/
├── components/
│   └── my-component.ts
├── services/
│   └── my-service.ts
└── index.ts
```

{% tabs %}
{% tab title="Javascript" %}
{% code title="my-feature/index.js" %}
```javascript
export function configure(config) {
  config.globalResources(['./my-component', './my-component-2', 'my-component-3', 'etc.']);
}
```
{% endcode %}
{% endtab %}

{% tab title="TypeScript" %}
{% code title="my-feature/index.ts" %}
```typescript
import { FrameworkConfiguration } from 'aurelia-framework';

export function configure(config: FrameworkConfiguration): void {
  config.globalResources(['./my-component', './my-component-2', 'my-component-3', 'etc.']);
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

The `configure` method receives a `FrameworkConfiguration` instance, identical to the one used with `aurelia.use`. This allows the feature to customize your app as needed. Remember to configure resources using paths relative to `index.js/ts`.

To enable this feature in your app, here's an example configuration:

{% tabs %}
{% tab title="Javascript" %}
{% code title="main.js" %}
```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .feature('my-feature');

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endcode %}
{% endtab %}

{% tab title="TypeScript" %}
{% code title="main.ts" %}
```typescript
import { Aurelia } from 'aurelia-framework';

export function configure(aurelia: Aurelia): void {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .feature('my-feature');

  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
When using Webpack, the syntax for enabling a feature is a little different. Instead of calling `.feature('my-feature');`, you will want to use the `PLATFORM.moduleName(...)` helper that allows the Aurelia Webpack plugin to understand dynamic module references. In this case, your syntax will look like `.feature(PLATFORM.moduleName('my-feature/index'));` Notice that in addition to the use of the `PLATFORM.moduleName(...)` helper, the index file must be directly referenced.
{% endhint %}

## Installing plugins

While features are built into your application, plugins are third-party add-ons you install via your package manager.

1.  Install the plugin package:

    ```sh
    npm install my-plugin
    ```
2.  Configure it in your app:

    ```javascript
    export function configure(aurelia) {
      aurelia.use
        .standardConfiguration()
        .developmentLogging()
        .plugin('my-plugin', pluginConfiguration);

      aurelia.start().then(() => aurelia.setRoot());
    }
    ```

To use a plugin, provide its installation name to the plugin API. If the plugin needs configuration, check its docs for specifics. As the second parameter, you can pass a config object or callback.

Let's look at a real example: adding and configuring the dialog plugin with a callback. Here, we're using a DialogConfiguration type:

```javascript
export function configure(aurelia) {s
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .plugin('aurelia-dialog', config => {
      config.useDefaults();
      config.settings.lock = true;
      config.settings.centerHorizontalOnly = false;
      config.settings.startingZIndex = 5;
      config.settings.keyboard = true;
    });

  aurelia.start().then(() => aurelia.setRoot());
}
```

## Leveraging Progressive Enhancement

Aurelia isn't limited to replacing DOM portions with root components. It can also progressively enhance existing HTML, offering flexibility in rendering applications.

**This approach is particularly useful when:**

1. Generating server-side HTML, including Aurelia custom elements, for improved SEO.
2. Gradually integrating Aurelia into an existing traditional web app.

In these scenarios, Aurelia can "bring to life" the pre-rendered HTML by activating the components' rich behavior in the browser.

To achieve this, use the `enhance` method instead of the `aurelia-app` attribute. Here's how you can progressively enhance the entire body of your HTML page using manual bootstrapping (NPM example):

```html
<!doctype html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <my-component message="Enhance Me"></my-component>
  </body>
</html>
```

```javascript
import 'aurelia-bootstrapper';
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from 'aurelia-pal';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .globalResources(PLATFORM.moduleName('resources/my-component'));

  aurelia.start().then(() => {
    aurelia.enhance();
  });
}
```

This method allows for seamless integration of Aurelia's dynamic capabilities with server-rendered or existing HTML, providing a smooth transition path for your web applications.

### Customized progressive enhancement

For enhance to recognize and upgrade components in your HTML, you must declare them as global resources. We've demonstrated this with the my-component example above.

You can also optionally specify a data-binding context or target a specific DOM section for enhancement.

```html
<!doctype html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <my-component message.bind="message"></my-component>
  </body>
</html>
```

```javascript
import 'aurelia-bootstrapper';
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from 'aurelia-pal';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .globalResources(PLATFORM.moduleName('resources/my-component'));

  const viewModel = {
    message: 'Enhanced'
  };

  aurelia.start().then(() => {
    aurelia.enhance(viewModel, document.body);
  });
}
```

Sometimes you need to enhance several elements on a page that aren't directly related. This often happens when refactoring an existing non-Aurelia app piece by piece.

While you can't use aurelia.enhance multiple times on the same content, there's a workaround. You can directly use the templating engine's enhance method instead. This approach gives you more flexibility when upgrading your application incrementally.

```html
<!doctype html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <my-component message="Enhance Me"></my-component>
    <div class="42">Some legacy code that you don't want to enhance</div>
    <your-component message.bind="message"></your-component>
  </body>
</html>
```

```javascript
import { TemplatingEngine } from 'aurelia-framework';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .globalResources('resources/my-component', 'resources/your-component');

  aurelia.start().then(a => {
    let templatingEngine = a.container.get(TemplatingEngine);

    templatingEngine.enhance({
      container: a.container,
      element: document.querySelector('my-component'),
      resources: a.resources
    });

    templatingEngine.enhance({
      container: a.container,
      element: document.querySelector('your-component'),
      resources: a.resources,
      bindingContext: {
        message: 'Enhance Me as well'
      }
    });
  });
}
```

## Customizing conventions

Customizing Aurelia's bootstrap process is straightforward once you've set up your main configure method and linked it with aurelia-app. This flexibility allows you to tailor various aspects of your application's startup. Among these, tweaking Aurelia's conventions is a common task for developers. Let's explore how you can make these adjustments to suit your project's needs.

### Configuring the View Location convention

Aurelia uses View Strategies to connect views with their corresponding view models. When a component doesn't specify its own strategy, the ViewLocator service employs the ConventionalViewStrategy by default.

The ConventionalViewStrategy maps view-model module IDs to view IDs. For instance, a view model with the module ID "welcome.js" would be paired with a view at "welcome.html."

Need a different convention? You can customize the mapping logic during bootstrap:

1. Import the ViewLocator
2. Replace its convertOriginToViewUrl method with your own implementation

```javascript
import { ViewLocator } from 'aurelia-framework';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  ViewLocator.prototype.convertOriginToViewUrl = (origin) => {
    let moduleId = origin.moduleId;
    ...
    return "view.html";
  };

  aurelia.start().then(a => a.setRoot());
}
```

Replace "..." with your custom mapping logic to return the desired view path.

For Webpack users with templating engines like Jade, you'll need to adjust Aurelia to recognize .jade files instead of .html. This is because Webpack preserves the original sourcemaps, leaving transpilation to loader plugins. To configure Aurelia's ViewLocator for Jade, use:

```javascript
import { ViewLocator } from 'aurelia-framework';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  ViewLocator.prototype.convertOriginToViewUrl = (origin) => {
    let moduleId = origin.moduleId;
    let id = (moduleId.endsWith('.js') || moduleId.endsWith('.ts')) ? moduleId.substring(0, moduleId.length - 3) : moduleId;
    return id + '.jade';
  };

  aurelia.start().then(a => a.setRoot());
}
```

### Configuring the Fallback View location strategy

While tweaking the ConventionalViewStrategy's mapping logic is useful, you can go a step further by replacing the entire fallback view strategy. This is done by overriding the createFallbackViewStrategy method in the ViewLocator.

Here's how you might implement this:

```javascript
import { ViewLocator } from 'aurelia-framework';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  ViewLocator.prototype.createFallbackViewStrategy = (origin) => {
    return new CustomViewStrategy(origin);
  };

  aurelia.start().then(a => a.setRoot());
}
```

## Logging

Aurelia uses a straightforward logging system that's inactive by default. To enable logging, you'll need to set up an appender. The examples above demonstrate how to configure an appender that sends log data to the console. For your reference, here's the code snippet again:

```javascript
import { LogManager } from 'aurelia-framework';
import { ConsoleAppender } from 'aurelia-logging-console';

LogManager.addAppender(new ConsoleAppender());
LogManager.setLevel(LogManager.logLevel.debug);

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration;

  aurelia.start().then(() => aurelia.setRoot());
}
```

You can also see how to set the log level. Values for the `logLevel` include: `none`, `error`, `warn`, `info` and `debug`.

The above example uses our provided `ConsoleAppender`, but you can easily create your own appenders. Simply implement a class that matches the `Appender` interface from the logging library.
