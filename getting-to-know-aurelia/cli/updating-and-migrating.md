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

The auto-tracing bundler (available in CLI version 1.0.0-beta.1+) automatically manages JavaScript dependencies, significantly reducing manual configuration requirements.

#### Prerequisites

```bash
# Update to CLI version 1.0.0-beta.1 or higher
npm update -g aurelia-cli
```

#### Migration Steps

**1. Update CLI Version:**

```bash
# Check current version
au -v

# Update to latest
npm update -g aurelia-cli

# Verify update
au -v
```

**2. Simplify aurelia.json Dependencies:**

Most existing applications will work without modifications. However, you can remove many explicit dependency configurations:

**Before (Manual Dependencies):**
```json
{
  "dependencies": [
    "aurelia-bootstrapper",
    "aurelia-loader-default", 
    "aurelia-pal-browser",
    "aurelia-animator-css",
    "aurelia-binding",
    "aurelia-dependency-injection",
    "aurelia-event-aggregator",
    "aurelia-framework",
    "aurelia-history",
    "aurelia-history-browser",
    "aurelia-loader",
    "aurelia-logging",
    "aurelia-metadata",
    "aurelia-pal",
    "aurelia-path",
    "aurelia-polyfills",
    "aurelia-route-recognizer",
    "aurelia-router",
    "aurelia-task-queue",
    "aurelia-templating",
    "aurelia-templating-binding",
    "text"
  ]
}
```

**After (Auto-tracing):**
```json
{
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
}
```

**3. Enable Build Caching:**

Add cache configuration to improve build performance:

```json
{
  "build": {
    "options": {
      "cache": true
    }
  }
}
```

**4. Add New Task Files:**

The auto-tracing bundler includes cache management tasks. You may need to copy new task files from a fresh CLI project:

```bash
# Create a temporary new project to copy task files
au new temp-project

# Copy new task files to your existing project
cp temp-project/aurelia_project/tasks/clear-cache.js aurelia_project/tasks/
cp temp-project/aurelia_project/tasks/clear-cache.json aurelia_project/tasks/

# Remove temporary project
rm -rf temp-project
```

**5. Optional: Install Gulp Cache Plugin**

For enhanced transpilation caching:

```bash
npm install --save-dev gulp-cache
```

#### What Auto-tracing Handles

**Automatic Module Format Detection:**
- CommonJS modules
- AMD modules  
- UMD modules
- Native ES modules

**Core Node.js Module Stubbing:**
- Uses same stubs as Webpack and Browserify
- Ensures browser compatibility

**Aurelia View Template Dependencies:**
- Automatically traces `<require>` statements
- Includes referenced modules in bundles

**NPM Package Resolution:**
- Handles main field resolution
- Processes package.json configurations
- Manages peer dependencies

#### Migration Benefits

**1. Simplified Configuration:**
- Fewer manual dependency entries
- Reduced aurelia.json maintenance
- Automatic NPM package handling

**2. Performance Improvements:**
- Build caching reduces compilation time
- More efficient dependency resolution
- Better incremental builds

**3. Plugin Compatibility:**
- Standard NPM installation for all plugins
- No CLI-specific installation procedures
- Consistent behavior across bundlers

#### Verification Steps

After migration, verify everything works correctly:

```bash
# Clear any existing caches
au clear-cache

# Test development build
au run

# Test production build  
au build --env prod

# Run tests if configured
au test
```

#### Troubleshooting Auto-tracing Migration

**Build Errors After Migration:**

1. **Clear all caches:**
   ```bash
   au clear-cache
   rm -rf node_modules
   npm install
   ```

2. **Check for incompatible dependencies:**
   - Remove legacy dependency configurations
   - Ensure NPM packages are compatible versions

3. **Verify aurelia.json syntax:**
   - JSON must be valid
   - Check for trailing commas or syntax errors

**Missing Modules:**

If the auto-tracer doesn't detect certain modules:

```json
{
  "dependencies": [
    "aurelia-bootstrapper",
    "aurelia-loader-default", 
    "aurelia-pal-browser",
    "text",
    {
      "name": "missing-module",
      "path": "../node_modules/missing-module",
      "main": "index"
    }
  ]
}
```

{% hint style="success" %}
The auto-tracing bundler maintains maximum backward compatibility with existing CLI bundler projects while providing modern dependency management.
{% endhint %}

### Update 3rd Party Plugin Installation Guide

When updating or installing 3rd party plugins:

1.  **Install via NPM:**

    ```bash
    npm install <plugin-name> --save
    ```
2. **Configure in `aurelia.json`:**
   * Add necessary entries in the `dependencies` array.
   * Update `copyFiles` if additional assets are required.
