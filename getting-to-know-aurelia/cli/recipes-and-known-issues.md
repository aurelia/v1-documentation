# Recipes & known issues

## Aurelia CLI Bundler + Docker

**Recipe:**

1.  **Dockerfile Configuration:**

    ```dockerfile
    FROM node:14
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    RUN au build --env prod
    CMD ["npm", "start"]
    ```
2.  **Build and Run:**

    ```bash
    docker build -t your-app-name .
    docker run -p 8080:8080 your-app-name
    ```

{% hint style="info" %}
Ensure the Dockerfile copies all necessary files and sets the correct working directory.
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

## Building with TFS

**Issue:** Team Foundation Server (TFS) builds may fail due to environment discrepancies.

**Solution:**

1. **Ensure Node.js and NPM are Installed:**
   * Configure build agents with necessary tools.
2. **Install Dependencies During Build:**
   * Add `npm install` as a build step before running build commands.
3.  **Run Build Commands:**

    ```bash
    au build --env prod
    ```

{% hint style="info" %}
Use TFS build pipelines to automate these steps for consistency.
{% endhint %}

{% hint style="info" %}
For more detailed configurations and advanced usage, refer to the [Aurelia CLI GitHub Repository](https://github.com/aurelia/cli).
{% endhint %}
