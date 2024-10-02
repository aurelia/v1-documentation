# Configuration with aurelia.json

The `aurelia_project/aurelia.json` file centralizes the configuration for your project, including build tasks, paths, and bundling settings. It defines how the CLI interacts with your application, and you can customize it to suit your needs.

Key sections in `aurelia.json` include:

* **`build`**: Configuration for the build process.
* **`platform`**: Settings specific to the target platform (e.g., web, Electron).
* **`transpiler`**: Options for the transpiler if you're using one (e.g., Babel, TypeScript).
* **`paths`**: Aliases for directories in your project.
* **`bundles`**: Definitions for how your application's files are bundled.

While the CLI handles most configurations automatically, you can edit `aurelia.json` for advanced customization.
