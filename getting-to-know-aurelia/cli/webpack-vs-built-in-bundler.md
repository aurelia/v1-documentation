# Webpack vs built-in bundler

When creating a new project with the Aurelia CLI, you have the choice between two bundlers: **Webpack** and the CLI's **built-in bundler**. Your selection affects how your application handles module loading and bundling.

## Webpack

Webpack is a powerful and widely-used bundler with a rich ecosystem. It has built-in module loading capabilities, eliminating the need for a separate module loader. If you choose Webpack during project creation, the CLI generates a robust `webpack.config.js` file, providing a strong starting point for your application.

#### Important note

When using Webpack, wrap module references in `PLATFORM.moduleName()`.

```javascript
import {MyModule} from PLATFORM.moduleName('path/to/my-module');
```

## CLI's Built-in Bundler

The built-in bundler provided by the Aurelia CLI offers simplicity and ease of use. It's ideal for developers who prefer minimal configuration and are less familiar with Webpack. The built-in bundler supports CommonJS, AMD, UMD, and native ES modules.

With auto-tracing capabilities, the latest version of the built-in bundler automatically tracks dependencies, reducing or eliminating the need to manually configure them in `aurelia.json`.

{% hint style="info" %}
If you're migrating from an older version of the CLI bundler, most applications will work without changes. If you encounter issues, consult the [migration guide](https://discourse.aurelia.io/) or seek assistance on the Aurelia Discourse forum.
{% endhint %}
