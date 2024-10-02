# Updating & migrating

## Updating

To keep the Aurelia CLI up-to-date:

```bash
npm update -g aurelia-cli
```

{% hint style="info" %}
Regular updates ensure access to the latest features and bug fixes.
{% endhint %}

## Migrating

### From CLI Bundler to Webpack

To migrate your project from the CLI's built-in bundler to Webpack:

1.  **Install Webpack Dependencies:**

    ```bash
    npm install --save-dev webpack webpack-cli aurelia-webpack-plugin
    ```
2.  **Generate Webpack Configuration:**

    ```bash
    au migrate-to-webpack
    ```
3. **Update Project Files:**
   * Replace `aurelia_project/tasks` with Webpack configurations.
   * Adjust `package.json` scripts accordingly.

{% hint style="info" %}
Backup your project before migration.
{% endhint %}

### Upgrade to Auto-tracing Bundler

Ensure your Aurelia CLI is updated to version `1.0.0-beta.1` or above to utilize the auto-tracing bundler.

```bash
npm update -g aurelia-cli
```

**Hint:** Auto-tracing enhances dependency management by reducing manual configurations.

### Update 3rd Party Plugin Installation Guide

When updating or installing 3rd party plugins:

1.  **Install via NPM:**

    ```bash
    npm install <plugin-name> --save
    ```
2. **Configure in `aurelia.json`:**
   * Add necessary entries in the `dependencies` array.
   * Update `copyFiles` if additional assets are required.
