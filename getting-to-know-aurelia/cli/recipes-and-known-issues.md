# Recipes & known issues

## Aurelia CLI Bundler + Docker

**Issue:** File watcher not detecting changes from host when running Aurelia CLI inside Docker containers.

**Root Cause:** Docker's file system layering can prevent proper file change notifications from reaching the container.

**Solution:**

1.  **Modify File Watching to Use Polling:**

    Update `aurelia_project/tasks/watch.js` (or `watch.ts`):

    ```javascript
    // watch.js
    const gulp = require('gulp');
    const watch = require('gulp-watch');

    function watchPath(path, tasks) {
      return watch(path, { 
        usePolling: true,  // Enable polling for Docker
        interval: 1000     // Check every second
      }, gulp.series(tasks));
    }

    function watch() {
      return watchPath('src/**/*.{js,html,css}', 'build');
    }

    module.exports = {
      default: watch,
      watchPath
    };
    ```

2.  **Alternative Docker Configuration:**

    ```dockerfile
    FROM node:14
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    
    # Enable file watching with polling
    ENV CHOKIDAR_USEPOLLING=true
    ENV CHOKIDAR_INTERVAL=1000
    
    RUN au build --env prod
    CMD ["npm", "start"]
    ```

3.  **Development Docker Compose:**

    ```yaml
    # docker-compose.yml
    version: '3.8'
    services:
      aurelia-app:
        build: .
        ports:
          - "8080:8080"
        volumes:
          - .:/app
          - /app/node_modules
        environment:
          - CHOKIDAR_USEPOLLING=true
        command: au run --watch
    ```

{% hint style="warning" %}
Using polling increases CPU usage but ensures reliable file change detection in Docker environments.
{% endhint %}

## NPM Uninstalling Packages

**Issue:** Uninstalling a package without updating `aurelia.json` can lead to build errors.

**Solution:**

1.  **Uninstall the Package:**

    ```bash
    npm uninstall <package-name> --save
    ```
2. **Remove from `aurelia.json`:**
   * Delete any references in the `dependencies` array.
   * Update `copyFiles` if assets were previously copied.

{% hint style="info" %}
Always verify dependency configurations after uninstalling packages.
{% endhint %}

## NPM Version Compatibility Issues

**Issue:** NPM version 5.6.0 causes package installation problems and dependency resolution conflicts.

**Symptoms:**
- Packages fail to install properly
- Inconsistent dependency trees
- Build failures after fresh installs

**Solution:**

1.  **Upgrade NPM to 5.7.0 or Higher:**

    ```bash
    npm install -g npm@5.7.0
    ```

    or

    ```bash
    npm install -g npm@latest
    ```

2.  **Clear NPM Cache:**

    ```bash
    npm cache clean --force
    ```

3.  **Reinstall Dependencies:**

    ```bash
    rm -rf node_modules package-lock.json
    npm install
    ```

4.  **Verify Installation:**

    ```bash
    npm -v
    au -v
    ```

{% hint style="warning" %}
NPM 5.6.0 has known issues with package-lock.json handling. Always use NPM 5.7.0+ or the latest stable version.
{% endhint %}

## Building with TFS (Team Foundation Server)

**Primary Issue:** TFS sets readonly attributes on checked-out files, causing build failures when the CLI tries to overwrite destination files.

**Error Symptoms:**
- Build fails on second and subsequent attempts
- "EACCES: permission denied" errors during build
- Files cannot be overwritten in output directories

**Solution:**

1.  **Modify Environment Configuration to Remove Read-Only Files:**

    Update `aurelia_project/environments/environment.js` (or `.ts`):

    ```javascript
    import fs from 'fs';
    import path from 'path';

    // Remove destination file before build to avoid read-only conflicts
    function removeReadOnlyFile(filePath) {
      try {
        if (fs.existsSync(filePath)) {
          // Remove readonly attribute and delete file
          fs.chmodSync(filePath, 0o666);
          fs.unlinkSync(filePath);
        }
      } catch (error) {
        console.warn(`Could not remove file ${filePath}:`, error.message);
      }
    }

    // Pre-build cleanup
    export function prebuild() {
      const outputDir = 'dist';
      const commonFiles = [
        path.join(outputDir, 'app-bundle.js'),
        path.join(outputDir, 'vendor-bundle.js'),
        path.join(outputDir, 'index.html')
      ];
      
      commonFiles.forEach(removeReadOnlyFile);
    }

    export default {
      debug: false,
      testing: false,
      prebuild
    };
    ```

2.  **Update Build Task to Call Pre-build Cleanup:**

    Modify `aurelia_project/tasks/build.js`:

    ```javascript
    import environment from '../environments/environment';

    export default gulp.series(
      readProjectConfiguration,
      environment.prebuild || (() => Promise.resolve()),
      gulp.parallel(
        buildJavaScript,
        buildHTML,
        buildCSS
      ),
      writeBundles
    );
    ```

3.  **TFS Build Pipeline Configuration:**

    Add these steps to your TFS build pipeline:

    ```yaml
    # TFS Build Pipeline Steps
    - task: NodeTool@0
      displayName: 'Install Node.js'
      inputs:
        versionSpec: '14.x'

    - script: |
        npm install -g aurelia-cli
        npm install
      displayName: 'Install Dependencies'

    - script: |
        # Change .gitignore to .tfignore for TFS
        if [ -f .gitignore ]; then
          cp .gitignore .tfignore
        fi
      displayName: 'Setup TFS Ignore File'

    - script: |
        # Remove read-only attributes recursively
        find . -name "*.js" -o -name "*.html" -o -name "*.css" | xargs chmod 666
      displayName: 'Remove Read-Only Attributes'
      condition: ne(variables['Agent.OS'], 'Windows_NT')

    - script: |
        # Windows version - remove read-only attributes
        attrib -R /S *.js *.html *.css
      displayName: 'Remove Read-Only Attributes (Windows)'
      condition: eq(variables['Agent.OS'], 'Windows_NT')

    - script: |
        au build --env prod
      displayName: 'Build Application'
    ```

4.  **Alternative: Use .tfignore Instead of .gitignore:**

    Create `.tfignore` file (copy from `.gitignore`):

    ```gitignore
    # TFS Ignore File
    node_modules/
    dist/
    .vscode/
    *.log
    npm-debug.log*
    aurelia_project/environments/dev.*
    aurelia_project/environments/stage.*
    ```

5.  **Ensure Build Agent Configuration:**

    - Install Node.js (v14+ recommended)
    - Install npm (v5.7.0+ to avoid known issues)
    - Configure sufficient disk space for node_modules
    - Set appropriate file system permissions

{% hint style="warning" %}
The readonly file issue is specific to TFS and does not affect other CI systems like Azure DevOps Services or GitHub Actions.
{% endhint %}

{% hint style="info" %}
For Azure DevOps Server (formerly TFS) 2019+, consider migrating build pipelines to YAML format for better version control and portability.
{% endhint %}

{% hint style="info" %}
For more detailed configurations and advanced usage, refer to the [Aurelia CLI GitHub Repository](https://github.com/aurelia/cli).
{% endhint %}
