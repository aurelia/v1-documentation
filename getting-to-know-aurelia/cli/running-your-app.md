# Running your app

Navigate to your project directory and start the development server with:

```bash
au run
```

This command builds your application and starts a minimal web server that serves your app. The development server automatically refreshes your browser when source code changes are detected.

## Common Options

### Specifying an Environment

Run with a specific environment configuration:

```bash
au run --env prod
```

See [Environments](environments.md) for details on configuring environment-specific settings.

### Changing the Port

Override the default port (configured in `aurelia.json`):

```bash
au run --port 8080
```

### Opening the Browser

Automatically open your default browser when the server starts:

```bash
au run --open
```

## Hot Module Reload (HMR)

If your project uses Webpack, you can enable Hot Module Reload for faster development:

```bash
au run --hmr
```

HMR updates modules in the browser without a full page reload, preserving application state during development.

## ASP.NET Core Projects

If you chose to use ASP.NET Core and Webpack during project setup, start your application using:

```bash
dotnet run
```

This starts both the ASP.NET Core server and the Webpack development middleware.

## What Happens on `au run`

1. The CLI builds your application using the current environment settings
2. A local web server starts (BrowserSync by default)
3. The CLI watches your source files for changes
4. When changes are detected, affected bundles are rebuilt
5. The browser automatically refreshes to show your changes

{% hint style="info" %}
The `au run` command combines building and serving. If you only need to build without a server, use `au build` instead. If you need the server without file watching, you can serve the output directory with any static file server.
{% endhint %}
