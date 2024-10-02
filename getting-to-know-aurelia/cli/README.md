# CLI

The Aurelia CLI includes a built-in bundler that uses RequireJS or SystemJS, offering an alternative to Webpack for managing module bundling. This bundler provides flexibility and ease of use, especially for developers preferring a more straightforward setup without the complexity of Webpack.

## CLI’s Built-in Bundler

The CLI's built-in bundler is designed to simplify the bundling process by automatically handling module dependencies and optimizing the application for production. It supports both RequireJS and SystemJS module loaders, allowing developers to choose based on their project requirements.

### Setting Up the Built-in Bundler

To create a new project using the CLI's built-in bundler:

1.  **Initialize a New Project**:

    ```bash
    au new appName
    ```
2. **Choose Configuration Options**:
   * **Custom Setup**: Select the **Custom** option when prompted.
   * **Bundler Selection**: Choose between **CLI's built-in bundler with RequireJS** or **CLI's built-in bundler with SystemJS**.
   * **Transpiler**: Select your preferred transpiler (Babel or TypeScript).
   * **CSS Preprocessor**: Choose a CSS preprocessor if needed (e.g., Sass, Less).
3.  **Install Dependencies**: The CLI will prompt to install the necessary dependencies. Confirm to proceed:

    ```bash
    npm install
    ```

    or

    ```bash
    yarn install
    ```

### Project Structure

After setup, your project will include the following key directories and files:

* **aurelia\_project/**: Contains configuration files and Gulp tasks.
  * **tasks/**: Holds Gulp task definitions.
  * **aurelia.json**: Main configuration file for the bundler.
* **src/**: Your application's source code.
* **package.json**: Lists project dependencies and scripts.

## Dependency Management

Efficient dependency management is crucial for maintaining a scalable and maintainable project. The built-in bundler in Aurelia CLI simplifies this by offering both automatic and manual dependency tracing.

### Auto Tracing

The built-in bundler automatically traces JavaScript dependencies, supporting various module formats:

1. **Module Formats Supported**:
   * **CommonJS**
   * **AMD**
   * **UMD**
   * **Native ES Modules**
2. **Core Node.js Module Stubbing**: The bundler stubs core Node.js modules to ensure browser compatibility, using the same stubs as Webpack and Browserify.
3. **Aurelia View Template Tracing**: Dependencies specified in Aurelia view templates using `<require>` are automatically traced and bundled.
4.  **Automatic Installation**: For ESNext applications, the bundler can automatically install missing NPM packages during development using the `--auto-install` flag:

    ```bash
    au run --auto-install
    ```

    **Warning**:

    ```warning
    The `--auto-install` flag is not supported for TypeScript projects due to TypeScript's compilation behavior.
    ```

### Manual Tracing

While auto-tracing handles most dependencies, certain scenarios require manual intervention:

1.  **Modules Not Explicitly Required**: Some modules may not be directly imported into your code. To include them in the bundle, specify them in the `dependencies` section of `aurelia.json`:

    ```json
    "dependencies": [
      "aurelia-bootstrapper",
      "aurelia-loader-default",
      "aurelia-pal-browser",
      {
        "name": "aurelia-testing",
        "env": "dev"
      },
      "text"
    ]
    ```
2. **Legacy Modules Without Module Support**: For packages that do not support any module format, use the `prepend` or `shim` configurations to integrate them:
   *   **Prepend**: Add the script paths in the `prepend` array to include them before the module loader:

       ```json
       "bundles": [
         {
           "name": "vendor-bundle.js",
           "prepend": [
             "node_modules/jquery/dist/jquery.min.js",
             "node_modules/requirejs/require.js"
           ],
           "dependencies": [ /* ... */ ]
         }
       ]
       ```
   *   **Shim**: Wrap the legacy scripts as AMD modules:

       ```json
       "dependencies": [
         {
           "name": "tempusdominus-bootstrap-4",
           "deps": ["jquery", "bootstrap", "popper.js", "moment"],
           "wrapShim": true
         }
       ]
       ```
3.  **Conditional Bundling**: Handle environment-specific dependencies by specifying the `env` property:

    ```json
    {
      "name": "aurelia-testing",
      "env": "dev"
    }
    ```

## Advanced Usage

The built-in bundler offers advanced customization options to cater to complex project requirements. These include modifying module tracing behavior and globally configuring shim settings.

### onRequiringModule Callback

Customize how the bundler handles module tracing by implementing the `onRequiringModule` callback in `aurelia_project/tasks/build.js`:

```javascript
function writeBundles() {
  return buildCLI.dest({
    onRequiringModule: function(moduleId) {
      // Custom logic before tracing a module
      if (moduleId === 'client-info') return false; // Ignore this module
      if (moduleId === 'lodash') return ['lodash/core']; // Include specific sub-modules
      if (moduleId === 'jquery') return `define(function() { return window.jQuery });`; // Supply module implementation directly
    }
  });
}
```

**Usage Scenarios**:

1. **Ignore Specific Modules**: Prevent the bundler from including certain modules.
2. **Bundle Additional Dependencies**: Manually include dependencies that are not auto-traced.
3. **Supply Module Implementations**: Provide custom module implementations that are especially useful for legacy libraries.

### Global wrapShim Configuration

Apply `wrapShim` settings globally to all shimmed dependencies by modifying `aurelia.json`:

```json
"build": {
  "loader": {
    "type": "require",
    "configTarget": "vendor-bundle.js",
    "includeBundleMetadataInConfig": "auto",
    "config": {
      "wrapShim": true
    }
  }
}
```

{% hint style="info" %}
The `wrapShim` option delays the execution of legacy code until the module is loaded, ensuring proper initialization.
{% endhint %}

### NPM Package Alias

Alias NPM packages to use local copies or alternative versions by specifying the `path` and `main` properties:

```json
{
  "name": "vscode",
  "path": "../node_modules/monaco-languageclient/lib",
  "main": "vscode-compatibility"
}
```

**Usage**:

* Useful for projects that require patched versions of libraries or local modifications.

### Lazy Bundling of Main Files

Optimize bundle size by enabling lazy loading for certain packages:

```json
{
  "name": "lodash",
  "path": "../lodash",
  "lazyMain": true
}
```

**Behavior**:

* Only the modules explicitly imported (e.g., `import { map } from 'lodash/map';`) are bundled.
* The main file is bundled only when `import _ from 'lodash';` is used.

## CLI + Built-in Bundler Advanced

Leverage the flexibility of RequireJS and SystemJS to implement advanced bundling strategies.

### Prepend Configuration

Include scripts before the module loader to integrate non-module libraries:

```json
"bundles": [
  {
    "name": "vendor-bundle.js",
    "prepend": [
      "node_modules/jquery/dist/jquery.min.js",
      "node_modules/moment/min/moment-with-locales.min.js",
      "node_modules/popper.js/dist/umd/popper.min.js",
      "node_modules/bootstrap/dist/js/bootstrap.min.js",
      "node_modules/tempusdominus-bootstrap-4/build/js/tempusdominus-bootstrap-4.min.js",
      "node_modules/requirejs/require.js"
    ],
    "dependencies": [ /* ... */ ]
  }
]
```

**Example Usage**: Integrating `tempusdominus-bootstrap-4` with dependencies:

```javascript
// main.ts
import * as moment from 'moment';
window.moment = moment;

// app.ts
import * as $ from 'jquery';
import 'tempusdominus-bootstrap-4';

export class App {
  value = moment();

  attached() {
    $('#datetimepicker1').datetimepicker({
      date: this.value
    });

    $('#datetimepicker1').on('change.datetimepicker', e => {
      this.value = e.date;
    });
  }

  detached() {
    $('#datetimepicker1').datetimepicker('destroy');
  }
}
```

```html
<!-- app.html -->
<template>
  <require from="font-awesome/css/font-awesome.min.css"></require>
  <require from="bootstrap/css/bootstrap.min.css"></require>
  <require from="tempusdominus-bootstrap-4/build/css/tempusdominus-bootstrap-4.min.css"></require>
  <div class="container">
    <div class="row">
      <div class="col-sm-6">
        <p>Value: ${value}</p>
        <div class="form-group">
          <div class="input-group date" id="datetimepicker1" data-target-input="nearest">
            <input type="text" class="form-control datetimepicker-input" data-target="#datetimepicker1">
            <div class="input-group-append" data-target="#datetimepicker1" data-toggle="datetimepicker">
              <div class="input-group-text"><i class="fa fa-calendar"></i></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```

{% hint style="warning" %}
Avoid importing `$` from 'jquery' after it has been prepended to prevent duplicate instances.
{% endhint %}

### Shim Configuration

Wrap legacy libraries that do not support module loaders using shims:

```json
"dependencies": [
  {
    "name": "tempusdominus-bootstrap-4",
    "deps": ["jquery", "bootstrap", "popper.js", "moment"],
    "wrapShim": true
  }
]
```

**Example**: Exposing `moment` globally for legacy compatibility:

```javascript
// main.ts
import * as moment from 'moment';
window.moment = moment;

// app.ts
import * as $ from 'jquery';
import 'tempusdominus-bootstrap-4';
```

{% hint style="warning" %}
Use the `wrapShim` option to delay the execution of legacy scripts until all dependencies are loaded.
{% endhint %}

## CLI’s Built-in Bundler Cookbook

The following recipes demonstrate common integration scenarios with the CLI's built-in bundler, including popular libraries and handling specific requirements.

### jQuery Integration

1.  **Install jQuery**:

    ```bash
    npm install jquery
    ```

    or

    ```bash
    yarn add jquery
    ```
2.  **Import jQuery** in your TypeScript or JavaScript files:

    ```typescript
    import * as $ from 'jquery';
    ```

### Normalize.css

1.  **Install Normalize.css**:

    ```bash
    npm install normalize.css
    ```

    or

    ```bash
    yarn add normalize.css
    ```
2.  **Include in Your HTML Templates**:

    ```html
    <template>
      <require from="normalize.css"></require>
    </template>
    ```

### Bootstrap CSS v4

1.  **Install Bootstrap and Dependencies**:

    ```bash
    npm install jquery bootstrap popper.js
    ```

    or

    ```bash
    yarn add jquery bootstrap popper.js
    ```
2.  **Import Bootstrap JavaScript** in `main.ts`:

    ```typescript
    import 'bootstrap';
    ```
3.  **Include Bootstrap CSS** in your HTML templates:

    ```html
    <template>
      <require from="bootstrap/css/bootstrap.min.css"></require>
    </template>
    ```

### Customize Bootstrap CSS v4

1.  **Initialize a New Project with Sass**:

    ```bash
    au new demo
    ```

    * Choose **Custom** setup.
    * Select **CLI's built-in bundler with RequireJS** or **SystemJS**.
    * Choose **Babel** or **TypeScript**.
    * Select **Sass** as the CSS preprocessor.
2.  **Install Dependencies**:

    ```bash
    npm install jquery bootstrap popper.js
    ```

    or

    ```bash
    yarn add jquery bootstrap popper.js
    ```
3.  **Customize Bootstrap Variables** in `app.scss`:

    ```scss
    // app.scss
    $grid-columns: 24; // Customize to use 24 columns
    @import '../node_modules/bootstrap/scss/bootstrap';
    ```
4.  **Include Compiled CSS** in `app.html`:

    ```html
    <template>
      <require from="./app.css"></require>
    </template>
    ```

{% hint style="info" %}
Unlike Webpack, the built-in bundler processes Sass files before bundling. Use `.css` in `<require>` statements.
{% endhint %}

### Bootstrap CSS v3 (Legacy)

1.  **Install Bootstrap v3 and Dependencies**:

    ```bash
    npm install jquery bootstrap@3.3.7
    ```

    or

    ```bash
    yarn add jquery bootstrap@3.3.7
    ```
2.  **Configure `aurelia.json`**:

    ```json
    "bundles": [
      {
        "name": "vendor-bundle.js",
        "prepend": [
          "node_modules/jquery/dist/jquery.min.js",
          "node_modules/requirejs/require.js"
        ],
        "dependencies": [
          "aurelia-bootstrapper",
          "aurelia-loader-default",
          "aurelia-pal-browser",
          {
            "name": "bootstrap",
            "deps": ["jquery"],
            "path": "../node_modules/bootstrap",
            "main": "dist/js/bootstrap.min"
          }
        ]
      }
    ],
    "copyFiles": {
      "node_modules/bootstrap/dist/fonts/*": "bootstrap/fonts"
    }
    ```
3.  **Import Bootstrap JavaScript** in `main.ts`:

    ```typescript
    import 'bootstrap';
    ```
4.  **Include Bootstrap CSS** in `app.html`:

    ```html
    <template>
      <require from="bootstrap/css/bootstrap.min.css"></require>
    </template>
    ```

{% hint style="info" %}
Both `bootstrap/css/bootstrap.min.css` and `bootstrap/dist/css/bootstrap.min.css` are valid paths. Ensure the `copyFiles` target path matches your CSS inclusion.
{% endhint %}

### Font Awesome v5 Free

1.  **Install Font Awesome**:

    ```bash
    npm install @fortawesome/fontawesome-free
    ```

    or

    ```bash
    yarn add @fortawesome/fontawesome-free
    ```
2.  **Configure `aurelia.json`**:

    ```json
    "copyFiles": {
      "node_modules/@fortawesome/fontawesome-free/webfonts/*": "@fortawesome/fontawesome-free/webfonts"
    }
    ```
3.  **Include Font Awesome CSS** in `app.html`:

    ```html
    <template>
      <require from="@fortawesome/fontawesome-free/css/all.min.css"></require>
      <require from="@fortawesome/fontawesome-free/css/v4-shims.min.css"></require> <!-- Optional for v4 compatibility -->
      <i class="fas fa-cube"></i>
    </template>
    ```

### Font Awesome v4

1.  **Install Font Awesome v4**:

    ```bash
    npm install font-awesome
    ```

    or

    ```bash
    yarn add font-awesome
    ```
2.  **Configure `aurelia.json`**:

    ```json
    "copyFiles": {
      "node_modules/font-awesome/fonts/*": "font-awesome/fonts"
    }
    ```
3.  **Include Font Awesome CSS** in `app.html`:

    ```html
    <template>
      <require from="font-awesome/css/font-awesome.min.css"></require>
      <i class="fa fa-cube"></i>
    </template>
    ```

### Foundation CSS v6

1.  **Install Foundation and Dependencies**:

    ```bash
    npm install jquery what-input foundation-sites
    ```

    or

    ```bash
    yarn add jquery what-input foundation-sites
    ```
2.  **Import Foundation JavaScript** in `main.ts`:

    ```typescript
    import 'what-input';
    import 'foundation-sites';
    ```
3.  **Configure `app.ts`**:

    ```typescript
    import * as $ from 'jquery';
    import * as Foundation from 'foundation-sites';

    export class App {
      tooltip: any;

      attached() {
        this.tooltip = new Foundation.Tooltip($('#demo'));
      }

      detached() {
        if (this.tooltip) {
          this.tooltip.destroy();
          this.tooltip = null;
        }
      }
    }
    ```
4.  **Include Foundation CSS** in `app.html`:

    ```html
    <template>
      <require from="foundation-sites/css/foundation.min.css"></require>
      <span ref="demo" data-tooltip class="top" tabindex="2" title="Fancy word for a beetle.">demo</span>
    </template>
    ```

### Materialize CSS v1

1.  **Install Materialize and Dependencies**:

    ```bash
    npm install jquery materialize-css
    ```

    or

    ```bash
    yarn add jquery materialize-css
    ```
2.  **Configure `app.ts`**:

    ```typescript
    import * as materialize from 'materialize-css';

    export class App {
      modal: HTMLElement;

      attached() {
        materialize.Modal.init(this.modal);
      }

      detached() {
        const instance = materialize.Modal.getInstance(this.modal);
        if (instance) instance.destroy();
      }
    }
    ```
3.  **Include Materialize CSS** in `app.html`:

    ```html
    <template>
      <require from="materialize-css/css/materialize.min.css"></require>
      <a class="waves-effect waves-light btn modal-trigger" href="#modal1">Modal</a>
      <div ref="modal" id="modal1" class="modal">
        <div class="modal-content">
          <h4>Modal Header</h4>
          <p>A bunch of text</p>
        </div>
        <div class="modal-footer">
          <a href="#!" class="modal-close waves-effect waves-green btn-flat">Agree</a>
        </div>
      </div>
    </template>
    ```

### TypeORM with Shimmed Decorators

1.  **Install TypeORM**:

    ```bash
    npm install typeorm
    ```

    or

    ```bash
    yarn add typeorm
    ```
2.  **Configure `aurelia.json`**:

    ```json
    "bundles": [
      {
        "name": "vendor-bundle.js",
        "prepend": [ /* ... */ ],
        "dependencies": [
          /* ... */
          {
            "name": "typeorm",
            "path": "../node_modules/typeorm",
            "main": "typeorm-model-shim"
          }
        ]
      }
    ],
    "paths": {
      "entities": "../../server/src/entity"
    }
    ```
3.  **Adjust `tsconfig.json`**:

    ```json
    {
      "compilerOptions": {
        /* ... */
        "paths": {
          "entities/*": ["../../server/src/entity/*"]
        },
        "baseUrl": "src"
      }
    }
    ```
4.  **Usage in Code**:

    ```typescript
    import { Entity, PrimaryGeneratedColumn, Column } from 'entities/customer';

    @Entity()
    export class Customer {
      @PrimaryGeneratedColumn()
      id: number;

      @Column()
      name: string;
    }
    ```

{% hint style="info" %}
Ensure that the `aurelia.json` paths align with your server-side entity locations to maintain consistency between frontend and backend models.
{% endhint %}

### Copying Other Files (e.g., Fonts)

Resources not handled by JavaScript module loaders, such as fonts and images, need to be copied to the appropriate output directories.

1.  **Configure `aurelia.json`**:

    ```json
    "copyFiles": {
      "node_modules/font-awesome/fonts/*": "font-awesome/fonts",
      "node_modules/some-library/images/*": "some-library/images"
    }
    ```
2.  **Usage in CSS**:

    ```css
    /* Example in CSS */
    @font-face {
      font-family: 'FontAwesome';
      src: url('font-awesome/fonts/fontawesome-webfont.woff2') format('woff2'),
           url('font-awesome/fonts/fontawesome-webfont.woff') format('woff');
      font-weight: normal;
      font-style: normal;
    }
    ```

{% hint style="info" %}
Ensure that all copied files are referenced correctly in your CSS or HTML to load resources at runtime.
{% endhint %}

### Path Mappings

Simplify import statements by defining path mappings in `aurelia.json` and `tsconfig.json`.

1.  **Configure `aurelia.json`**:

    ```json
    "paths": {
      "root": "src",
      "resources": "resources",
      "elements": "resources/elements",
      "attributes": "resources/attributes",
      "valueConverters": "resources/value-converters",
      "bindingBehaviors": "resources/binding-behaviors",
      "models": "common/models"
    }
    ```
2.  **Configure `tsconfig.json`**:

    ```json
    {
      "compilerOptions": {
        /* ... */
        "paths": {
          "models/*": ["src/common/models/*"]
        },
        "baseUrl": "."
      }
    }
    ```
3.  **Usage in Imports**:

    ```typescript
    import { Customer } from 'models/customer';
    ```

{% hint style="info" %}
Restart your editor after updating `tsconfig.json` to ensure path mappings are recognized.
{% endhint %}

### Styling Your Application

Manage and import styles effectively within your Aurelia application.

1.  **Without CSS Preprocessor**:

    ```html
    <template>
      <require from="./styles.css"></require>
    </template>
    ```
2. **With CSS Preprocessor (e.g., Sass)**:
   * Write styles in `.scss` files.
   *   Import compiled `.css` files in your HTML templates:

       ```html
       <template>
         <require from="./styles.css"></require>
       </template>
       ```

{% hint style="info" %}
Unlike Webpack, the built-in bundler processes styles before bundling. Always reference the compiled `.css` files in your templates.
{% endhint %}

### Updating Libraries

Keep your project's dependencies up-to-date to leverage improvements and patches.

1.  **Updating a Single Library**:

    ```bash
    npm install <library-name>@latest
    ```

    or

    ```bash
    yarn add <library-name>@latest
    ```
2. **Updating Multiple Libraries**:
   *   **Define an Update Script** in `package.json`:

       ```json
       "scripts": {
         "au-update": "npm install aurelia-binding@latest aurelia-bootstrapper@latest ..."
       }
       ```
   *   **Run the Update**:

       ```bash
       npm run au-update
       ```

       or

       ```bash
       yarn run au-update
       ```

{% hint style="info" %}
Ensure updated libraries are compatible with Aurelia version 1 to prevent integration issues.
{% endhint %}

### JavaScript Minification

Optimize your application's performance by minifying JavaScript during production builds.

1.  **Default Minification Settings** in `aurelia.json`:

    ```json
    "options": {
      "minify": "stage & prod",
      "sourcemaps": "dev & stage"
    }
    ```
2.  **Custom Minification Options**:

    ```json
    "options": {
      "minify": {
        "dev": false,
        "default": {
          "indent_level": 2
        },
        "stage & prod": {
          "max-line-len": 100000
        }
      }
    }
    ```

    * **Minifier Used**: Terser ([terser GitHub](https://github.com/fabiosantoscode/terser)) supports options compatible with UglifyJS2 and Uglify-ES.

{% hint style="info" %}
Adjust minification options to balance between build performance and code optimization based on your project needs.
{% endhint %}
