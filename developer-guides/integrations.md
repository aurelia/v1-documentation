# Integrations

## Polymer Integration

Polymer is a lightweight library developed by Google that helps you create custom web components using a simple and declarative syntax. It provides features to make web component development more straightforward and efficient.

### Integration Approach

Aurelia offers robust support for integrating Web Components, including those created with Polymer. The integration allows you to:

* Use Polymer components directly in Aurelia applications
* Leverage two-way data binding
* Seamlessly interact with Polymer-created custom elements

### Compatibility Considerations

* Supported Polymer Versions: Polymer 1.x and 2.x
* Aurelia Version: Aurelia 1.x
* Recommended Polyfills: Web Components polyfills for broader browser support

## Installation

### Prerequisites

Before integrating Polymer with Aurelia, ensure you have the following:

* Node.js (latest LTS)
* Aurelia CLI or an existing Aurelia 1.x project
* Basic understanding of Web Components

### Required Dependencies

Install the following dependencies:

```bash
# Polymer Web Components Base Library
npm install @polymer/polymer

# Aurelia Web Component Support
npm install aurelia-web-components

# Web Components Polyfills
npm install @webcomponents/webcomponentsjs

```

### Installation Steps

**1. Add Polyfills**

In your project's entry point (typically `main.js`), include Web Components polyfills:

```bash
import '@webcomponents/webcomponentsjs/webcomponents-loader.js';
```

**2. Configure Aurelia Plugin**

Update your `main.js` to register the Web Components plugin:

```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin('aurelia-web-components');  // Add Web Component support

  aurelia.start().then(() => {
    aurelia.setRoot();
  });
}
```

**3. Import Polymer Components**

You can now import and use Polymer components in your Aurelia views:

```javascript
// In your component
import '@polymer/paper-button/paper-button.js';
```

### Usage Patterns

#### Importing Polymer Components

**Direct Import**

Import Polymer components in your Aurelia component:

```javascript
// In your component class
import '@polymer/paper-button/paper-button.js';
import '@polymer/paper-input/paper-input.js';
```

**View-Based Import**

You can also import components directly in your view:

```markup
<require from="@polymer/paper-button/paper-button.js"></require>
```

#### Binding to Polymer Components

**One-Way Binding**

Bind Aurelia view-model properties to Polymer component attributes:

```html
<template>
  <paper-input 
    value.bind="userName" 
    label="Username">
  </paper-input>
</template>
```

```javascript
export class MyComponent {
  userName = '';
}
```

**Two-Way Binding**

Use `.two-way` binding for two-way data synchronization:

```markup
<template>
  <paper-input 
    value.two-way="userName" 
    label="Username">
  </paper-input>
</template>
```

#### Event Handling

**Standard Event Binding**

Bind Polymer component events using Aurelia's event binding:

```html
<template>
  <paper-button 
    on-click.trigger="handleButtonClick($event)">
    Click Me
  </paper-button>
</template>
```

```javascript
export class MyComponent {
  handleButtonClick(event) {
    console.log('Button clicked', event);
  }
}
```

#### **Nested Polymer Components**

Combine multiple Polymer components:

```markup
<template>
  <paper-card>
    <paper-input 
      label="Name" 
      value.bind="name">
    </paper-input>
    <paper-button 
      raised 
      on-click.delegate="submitForm()">
      Submit
    </paper-button>
  </paper-card>
</template>
```

### Examples

#### Basic Component Integration

**Simple Form Example**

```javascript
// my-form.js
import '@polymer/paper-input/paper-input.js';
import '@polymer/paper-button/paper-button.js';

export class MyForm {
  name = '';
  email = '';

  submitForm() {
    console.log(`Submitting: ${this.name}, ${this.email}`);
  }
}
```

```markup
<!-- my-form.html -->
<template>
  <div class="form-container">
    <paper-input 
      label="Name" 
      value.two-way="name">
    </paper-input>
    
    <paper-input 
      label="Email" 
      type="email" 
      value.two-way="email">
    </paper-input>
    
    <paper-button 
      raised 
      on-click.delegate="submitForm()">
      Submit
    </paper-button>
  </div>
</template>
```

#### Advanced Example: Custom Element Wrapper

```javascript
// polymer-chart-wrapper.js
import '@polymer/paper-chart/paper-chart.js';

export class PolymerChartWrapper {
  @bindable chartData;
  @bindable chartType = 'bar';

  attached() {
    // Additional setup if needed
  }

  chartDataChanged(newValue) {
    // React to data changes
    this.updateChart(newValue);
  }
}
```
