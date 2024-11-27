# Advanced Plugin Techniques

### Wrapping Other Plugins

You can create a plugin that combines multiple plugins:

```javascript
export function configure(config) {
  // Load and configure multiple plugins
  config.plugin(PLATFORM.moduleName('aurelia-animator-css'));
  
  config.plugin(PLATFORM.moduleName('aurelia-dialog'), dialogConfig => {
    dialogConfig.useDefaults();
    dialogConfig.settings.lock = true;
    dialogConfig.settings.ignoreTransitions = true;
  });
}
```

### Modifying Dependency Injection Behavior

Override default DI behavior:

```javascript
import { MyService } from './my-service';

export function configure(config) {
  // Create a new instance for each injection
  config.transient(MyService, MyService);

  // Singleton with a custom implementation
  config.singleton(CustomLogger, class {
    constructor() {
      // Custom singleton logic
    }
  });
}
```

### Configuration Options

Create a configurable plugin:

```javascript
export function configure(config, callback) {
  const pluginConfig = {
    // Default configuration
    option1: false,
    option2: 'default value'
  };

  // Allow external configuration
  if (typeof callback === 'function') {
    callback(pluginConfig);
  }

  // Apply configuration
  config.container.registerInstance('MyPluginConfig', pluginConfig);
}
```

Usage in an app:

```javascript
aurelia.use.plugin('my-plugin', config => {
  config.option1 = true;
  config.option2 = 'custom value';
});
```

### Dynamic Resource Registration

Dynamically register resources based on configuration:

```javascript
export function configure(config, callback) {
  const resources = [];

  // Configurable resource registration
  function registerResource(resourcePath) {
    resources.push(PLATFORM.moduleName(resourcePath));
  }

  // Allow external configuration
  if (typeof callback === 'function') {
    callback(registerResource);
  }

  // Register all collected resources
  config.globalResources(resources);
}
```

### Plugin Composition

Create a plugin that composes multiple sub-plugins:

```javascript
import { UIPlugin } from './ui-plugin';
import { DataPlugin } from './data-plugin';

export function configure(config) {
  // Compose multiple plugin configurations
  config.plugin(PLATFORM.moduleName('./ui-plugin'));
  config.plugin(PLATFORM.moduleName('./data-plugin'));
}
```
