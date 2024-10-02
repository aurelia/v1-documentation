# Webpack

The Aurelia CLI offers full support for Webpack as a bundler, suitable for both ESNext and TypeScript applications.

## PLATFORM.moduleName

When referencing modules by string in a Webpack-configured Aurelia application, wrap the module names using `PLATFORM.moduleName`. This informs Webpack about Aurelia's dynamic loading behavior.

**Example:**

```javascript
// src/resources/index.js
import { PLATFORM } from 'aurelia-framework';

export function configure(config) {
  config.globalResources([
    PLATFORM.moduleName('./elements/my-awesome-element')
  ]);
}

// router configuration
export function configureRouter(config, router) {
  this.router = router;
  config.map([
    {
      route: '',
      name: 'home',
      title: 'Home',
      nav: true,
      moduleId: PLATFORM.moduleName('./home')
    }
  ]);
}
```

**Hint:** The `aurelia-webpack-plugin` processes `PLATFORM.moduleName`, bridging Webpack with Aurelia's dynamic behavior.

## Running the Application

To run your Aurelia application with Webpack:

```bash
au run
```

**Common Flags:**

*   `--hmr`: Enables Hot Module Reload.

    ```bash
    au run --hmr
    ```
*   `--analyze`: Opens the Webpack Bundle Analyzer.

    ```bash
    au run --analyze
    ```

**Configuration:**

Default settings for `au run` are located in the `platform` section of `aurelia.json`:

```json
"platform": {
  "id": "web",
  "displayName": "Web",
  "port": 8080,
  "hmr": false,
  "open": false,
  "output": "dist"
}
```

Adjust settings like `hmr`, `open`, and `port` as needed.

## Simplified Webpack

From Aurelia CLI v1.1.0 onwards, Webpack usage has been simplified:

* **Decoupled Bundler:** Webpack operates independently from the CLI's task files.
* **Commands:**
  * `au run` maps to `npm start`.
  * `au build` maps to `npm run build:dev`.

**Environment Configuration:**

Environment files are located in the `config/` folder, managed by the `app-settings-loader` Webpack plugin.

{% hint style="info" %}
This simplification is applicable only to new applications. Existing Webpack setups remain unaffected.
{% endhint %}

## Deploying Your App

To prepare your application for deployment:

1.  **Create a Production Build:**

    ```bash
    au build --env prod
    ```
2. **Deployment Output:**
   * **Non-ASP.NET Core:** Contents reside in the `dist` folder. Copy this to your web server.
   * **ASP.NET Core:** Build output is located in `wwwroot/dist`. Deploy the ASP.NET Core application as per [Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/publishing/?tabs=aspnetcore2x).

## Build Options

Webpack configurations are exported as functions based on parameters:

```javascript
module.exports = ({ production, server, extractCss, coverage } = {}) => ({
  // Webpack configuration
});
```

These parameters are influenced by the `build.options` in `aurelia.json`:

```json
"build": {
  "options": {
    "server": "dev",
    "extractCss": "prod",
    "coverage": false
  }
}
```

* **Production Flag:** Determined by the `--env` flag (`prod` for production builds).

## Installing 3rd Party Dependencies

Webpack intelligently bundles 3rd party dependencies. To install a new dependency:

```bash
npm install <library> --save
# or
yarn add <library>
```

{% hint style="info" %}
Most dependencies require minimal or no additional configuration. Refer to the [Webpack documentation](https://webpack.js.org/concepts/) for advanced configurations.
{% endhint %}

## Unit Testing

Depending on your setup during `au new`, you may have one of the following test runners:

*   **Jest:**

    ```bash
    au jest
    au jest --watch
    ```
*   **Karma:**

    ```bash
    au karma
    au karma --watch
    ```
*   **Protractor:**

    ```bash
    nps e2e
    ```

    _Ensure `nps` is installed globally:_

    ```bash
    npm install nps -g
    ```

## ASP.NET Core

When integrating with ASP.NET Core:

1.  **Set Environment Variable:**

    ```bash
    export ASPNETCORE_ENVIRONMENT=Development
    ```

    _Refer to_ [_Microsoft Docs_](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments#setting-the-environment) _for detailed instructions._
2. **Configure Project:**
   * Use Visual Studio to create the ASP.NET Core project.
   * Integrate Aurelia using `au new --here` within the project directory.

**Hint:** Prevent Visual Studio from compiling TypeScript by adding `<TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>` to your `.xproj` file.
