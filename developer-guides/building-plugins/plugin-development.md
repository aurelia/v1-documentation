# Plugin Development

### Plugin Entry Point

The plugin entry point is typically located at `src/index.js` (or `src/index.ts` for TypeScript). It exports a `configure` function that Aurelia calls when the plugin is loaded:

```javascript
import { PLATFORM } from 'aurelia-pal';

export function configure(config) {
  // Plugin configuration logic
  config.globalResources([
    PLATFORM.moduleName('./elements/my-custom-element'),
    PLATFORM.moduleName('./value-converters/my-value-converter')
  ]);
}
```

#### Configuration Methods

The `configure` function receives a `FrameworkConfiguration` object with several important methods:

| Method              | Description                                                     | Example                                                       |
| ------------------- | --------------------------------------------------------------- | ------------------------------------------------------------- |
| `globalResources()` | Register global resources like elements, attributes, converters | `config.globalResources(PLATFORM.moduleName('./my-element'))` |
| `plugin()`          | Load and configure other plugins                                | `config.plugin(PLATFORM.moduleName('aurelia-dialog'))`        |
| `singleton()`       | Register a singleton in the dependency injection container      | `config.singleton(MyService, MyService)`                      |
| `transient()`       | Register a transient dependency                                 | `config.transient(MyService, MyService)`                      |

### Creating Resources

#### Custom Elements

Generate a custom element using Aurelia CLI:

```bash
au generate element my-element
```

Example custom element:

```javascript
// src/elements/my-element.js
export class MyElement {
  @bindable title;
  
  constructor() {
    // Component logic
  }
}
```

#### Custom Attributes

Generate a custom attribute:

```bash
au generate attribute my-attribute
```

Example custom attribute:

```javascript
// src/attributes/my-attribute.js
export class MyAttribute {
  constructor(element) {
    this.element = element;
  }
  
  valueChanged(newValue) {
    // Attribute logic
  }
}
```

#### Value Converters

Generate a value converter:

```bash
au generate value-converter my-converter
```

Example value converter:

```javascript
// src/value-converters/my-converter.js
export class MyValueConverter {
  toView(value) {
    // Convert value for view
    return transformedValue;
  }
  
  fromView(value) {
    // Convert value from view
    return originalValue;
  }
}
```

#### Binding Behaviors

Generate a binding behavior:

```bash
au generate binding-behavior my-behavior
```

Example binding behavior:

```javascript
// src/binding-behaviors/my-behavior.js
export class MyBindingBehavior {
  bind(binding, source) {
    // Modify binding behavior
  }
  
  unbind(binding, source) {
    // Cleanup
  }
}
```

### Registering Resources

After creating resources, register them in the plugin's entry point:

```javascript
export function configure(config) {
  config.globalResources([
    PLATFORM.moduleName('./elements/my-element'),
    PLATFORM.moduleName('./attributes/my-attribute'),
    PLATFORM.moduleName('./value-converters/my-converter'),
    PLATFORM.moduleName('./binding-behaviors/my-behavior')
  ]);
}
```

### Development Workflow

#### Running the Dev App

Start the development application:

```bash
au run --open
```

This command:

* Launches the development application
* Automatically opens it in your default browser
* Provides live reloading during development

#### Testing

**Running Tests**

```bash
# Run unit tests
au test
```

{% hint style="info" %}
Recommended Testing Frameworks:

* Karma: Best for real browser testing
* Jest: Faster, but uses a simulated browser environment
{% endhint %}

**Writing Tests**

Example unit test:

```javascript
// test/unit/my-element.spec.js
import { MyElement } from '../../src/elements/my-element';

describe('MyElement', () => {
  it('should create an instance', () => {
    const element = new MyElement();
    expect(element).toBeTruthy();
  });
});
```

#### Managing Dependencies

**Internal Dependencies**

* Use `PLATFORM.moduleName()` for module references
* Avoid direct imports in the plugin's entry point

**External Dependencies**

In `package.json`:

```json
{
  "dependencies": {
    "lodash": "^4.17.21"
  },
  "peerDependencies": {
    "aurelia-pal": "^1.x.x"
  }
}
```

{% hint style="warning" %}
Recommendations:

* Add specific npm packages to `dependencies`
* Use `peerDependencies` for Aurelia core packages
* Avoid duplicating Aurelia core packages
{% endhint %}

#### Recommended Workflow

1. Generate resources
2. Implement functionality
3. Write tests
4. Run dev app to verify
5. Iterate and refine
