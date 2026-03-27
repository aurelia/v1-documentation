# Configuration with aurelia.json

The `aurelia_project/aurelia.json` file centralizes the configuration for your project, including build tasks, paths, and bundling settings. It defines how the CLI interacts with your application, and you can customize it to suit your needs.

## Key Sections

### build

Controls the build process, including the target output and minification settings:

```json
{
  "build": {
    "targets": [
      {
        "id": "web",
        "displayName": "Web",
        "output": "scripts",
        "index": "index.html",
        "baseDir": ".",
        "minify": "stage & prod",
        "sourcemap": "dev & stage"
      }
    ]
  }
}
```

### platform

Settings specific to the target platform (e.g., web, Electron):

```json
{
  "platform": {
    "id": "web",
    "displayName": "Web",
    "port": 9000,
    "hmr": false
  }
}
```

### transpiler

Configures the transpiler for your project (Babel or TypeScript):

```json
{
  "transpiler": {
    "id": "typescript",
    "displayName": "TypeScript",
    "fileExtension": ".ts",
    "dtsSource": ["./custom_typings/**/*.d.ts"],
    "source": "src/**/*.ts"
  }
}
```

### paths

Directory aliases used throughout the configuration:

```json
{
  "paths": {
    "root": "src",
    "resources": "resources",
    "elements": "resources/elements",
    "attributes": "resources/attributes",
    "valueConverters": "resources/value-converters",
    "bindingBehaviors": "resources/binding-behaviors"
  }
}
```

### bundles

Defines how your application files are bundled for deployment. The built-in bundler uses this to determine which modules go into which bundle files:

```json
{
  "bundles": [
    {
      "name": "app-bundle.js",
      "source": ["**/*.{js,json,css,html}"]
    },
    {
      "name": "vendor-bundle.js",
      "prepend": ["node_modules/bluebird/js/browser/bluebird.core.js"],
      "dependencies": [
        "aurelia-bootstrapper",
        "aurelia-loader-default",
        "aurelia-pal-browser"
      ]
    }
  ]
}
```

{% hint style="info" %}
The `bundles` section is only used by the built-in bundler. If you're using Webpack, bundle configuration is handled in your `webpack.config.js` instead.
{% endhint %}

## Common Customizations

### Changing the Dev Server Port

```json
{
  "platform": {
    "port": 8080
  }
}
```

### Adding External Dependencies

For the built-in bundler, add third-party libraries to the `dependencies` array in your vendor bundle:

```json
{
  "dependencies": [
    "lodash",
    {
      "name": "moment",
      "path": "../node_modules/moment",
      "main": "moment"
    }
  ]
}
```

While the CLI handles most configurations automatically, you can edit `aurelia.json` for advanced customization. See the [built-in bundler](built-in-bundler.md) and [Webpack](webpack.md) sections for bundler-specific configuration details.
