# Building your app

To build your application without starting the development server, run:

```bash
au build
```

You can also specify an environment:

```bash
au build --env stage
```

This command creates optimized bundles of your app, ready for deployment.

## Build Environments

The `--env` flag determines which environment configuration is used and what optimizations are applied:

```bash
au build              # Development build (default)
au build --env dev    # Explicit development build
au build --env stage  # Staging build with minification
au build --env prod   # Production build with full optimization
```

In `stage` and `prod` environments, the build process automatically minifies your code and generates source maps (based on your `aurelia.json` configuration).

## Build Output

By default, built files are placed in the `scripts/` directory (or the output path configured in `aurelia.json`). The output typically includes:

- **`app-bundle.js`** — Your application code
- **`vendor-bundle.js`** — Third-party dependencies

{% hint style="info" %}
If your project uses Webpack instead of the built-in bundler, the build output location and file names are determined by your `webpack.config.js`.
{% endhint %}

## Watch Mode

To rebuild automatically when files change without starting a server:

```bash
au build --watch
```

This is useful when you're serving your app through an external server but still want the CLI to handle bundling.

## Troubleshooting

If builds fail, common causes include:

- **Missing dependencies** — Run `npm install` to ensure all packages are installed
- **TypeScript errors** — Check for type errors in your source files
- **Bundle configuration** — Verify that third-party dependencies are correctly listed in `aurelia.json`

See [Recipes & known issues](recipes-and-known-issues.md) for solutions to common build problems.
