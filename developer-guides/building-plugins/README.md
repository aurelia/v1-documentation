# Building plugins

### Introduction

#### What is an Aurelia Plugin?

An Aurelia plugin is a reusable package that extends the functionality of Aurelia applications. Plugins can introduce:

* Custom elements
* Custom attributes
* Value converters
* Binding behaviors
* Global configuration options
* Shared services and utilities

#### Benefits of Creating Plugins

Plugins offer several advantages to the Aurelia ecosystem:

* **Modularity**: Encapsulate and share specific functionality
* **Reusability**: Use the same plugin across multiple projects
* **Abstraction**: Simplify complex implementations
* **Community Contribution**: Share innovative solutions with other developers

### Getting Started

#### Prerequisites

Before creating an Aurelia plugin, ensure you have:

* Node.js (version 8.0 or higher)
* npm (version 5.0 or higher)
* Aurelia CLI installed globally

```bash
# Install Aurelia CLI globally
npm install -g aurelia-cli
```

#### Creating a New Plugin Project

You can create a new Aurelia plugin project using the Aurelia CLI:

```bash
# Create a new plugin project
au new --plugin

# Or specify a project name
au new my-awesome-plugin --plugin
```

When prompted, you'll have several configuration options:

* Language (ESNext or TypeScript)
* Module loader
* Unit testing framework
* Project structure

{% hint style="info" %}
Pro Tip: Choose the default ESNext setup if you're unsure. You can always customize later.
{% endhint %}

#### Plugin Project Structure

A typical Aurelia plugin project created by the CLI will have this structure:

```
my-awesome-plugin/
│
├── src/                  # Plugin source code
│   └── index.js          # Plugin entry point
│   └── elements/         # Custom elements
│   └── attributes/       # Custom attributes
│   └── value-converters/ # Value converters
│
├── dev-app/              # Development application
│   └── src/              # Dev app source
│   └── index.html        # Dev app entry
│
├── test/                 # Unit tests
│   └── unit/             # Unit test files
│
├── package.json          # Project metadata and dependencies
└── aurelia.json          # Aurelia-specific configuration
```

**Key Folders Explained**

* `src/`: Contains the actual plugin source code
  * `index.js`: The main entry point for configuring the plugin
* `dev-app/`: A development application for testing the plugin
* `test/`: Contains unit tests for the plugin
* `dist/`: (Generated) Compiled plugin files for distribution

{% hint style="warning" %}
Important: Always use `PLATFORM.moduleName()` when referencing resources in the `src/` directory to ensure proper module loading.
{% endhint %}

### Best Practices

#### Dependency Management

* Use `peerDependencies` for Aurelia core packages
* Specify exact versions for critical dependencies
* Avoid unnecessary dependencies

#### Compatibility Considerations

* Support multiple Aurelia versions when possible
* Use feature detection over version checking
* Provide clear documentation for version compatibility

#### Performance Tips

* Lazy load complex resources
* Minimize global resource registration
* Use efficient dependency injection patterns

{% hint style="info" %}
Pro Tips:

* Write comprehensive tests
* Document plugin usage clearly
* Provide migration guides for breaking changes
* Maintain consistent versioning
{% endhint %}

#### Next Steps

Now that you've created your plugin project, you're ready to:

* Define your plugin's entry point
* Create custom resources
* Configure plugin behavior
* Develop and test your plugin

In the next sections, we'll explore plugin development, resource creation, and advanced configuration techniques in more detail.
