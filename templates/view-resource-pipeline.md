# View Resource Pipeline

The View Resource Pipeline is the mechanism Aurelia uses to discover, load, and process all the resources your templates depend on. This includes custom elements, custom attributes, value converters, binding behaviors, and even CSS files.

## The `<require>` Element

The primary way to import resources into a template is with the `<require>` element:

{% code title="hello.html" %}
```markup
<template>
  <require from="bootstrap/css/bootstrap.min.css"></require>
  <p class="lead">Hello, World!</p>
  <p>Lorem Ipsum...</p>
</template>
```
{% endcode %}

The `<require>` tag isn't limited to HTML or view models — it can also handle CSS files. Aurelia's View Resource Pipeline intelligently recognizes and processes these different resource types based on their file extension.

### Importing Custom Elements

```markup
<template>
  <require from="./my-component"></require>
  <my-component></my-component>
</template>
```

### Importing Custom Attributes

```markup
<template>
  <require from="./tooltip"></require>
  <p tooltip="Hello!">Hover me</p>
</template>
```

### Importing Value Converters

```markup
<template>
  <require from="./date-format"></require>
  <p>${createdDate | dateFormat}</p>
</template>
```

### Importing CSS

CSS imported via `<require>` is scoped and processed by the pipeline:

```markup
<template>
  <require from="./styles.css"></require>
  <div class="styled-content">Content here</div>
</template>
```

## Aliasing with `as`

You can alias an imported resource using the `as` attribute. This is useful when a resource name conflicts with another or when you want a more descriptive name:

```markup
<template>
  <require from="./shared/nav-bar" as="app-nav"></require>
  <app-nav></app-nav>
</template>
```

## Global Resources

Instead of importing resources in every template, you can register them globally in your application's startup configuration:

```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .globalResources([
      './resources/nav-bar',
      './resources/date-format',
      './resources/tooltip'
    ]);

  aurelia.start().then(() => aurelia.setRoot());
}
```

Global resources are available in all templates without a `<require>` import.

## Resource Conventions

Aurelia uses naming conventions to determine what type of resource a file contains:

- **Custom elements** — A class with a corresponding `.html` template file (e.g., `nav-bar.js` + `nav-bar.html`)
- **Custom attributes** — A class decorated with `@customAttribute` or following the `*CustomAttribute` naming convention
- **Value converters** — A class following the `*ValueConverter` naming convention (e.g., `DateFormatValueConverter`)
- **Binding behaviors** — A class following the `*BindingBehavior` naming convention (e.g., `ThrottleBindingBehavior`)

## Extending the Pipeline

You can create custom view resource types by implementing a `ViewEngineHooks` class. This allows you to intercept and transform resources as they flow through the pipeline:

```javascript
import { ViewEngineHooks } from 'aurelia-framework';

export class MyViewEngineHooks extends ViewEngineHooks {
  beforeCompile(content, resources, instruction) {
    // Modify template content before compilation
  }

  afterCompile(viewFactory) {
    // Modify the compiled view factory
  }

  beforeCreate(viewFactory) {
    // Intercept before a view instance is created
  }

  afterCreate(view) {
    // Modify a view after creation
  }
}
```

Register your hooks as a global resource:

```javascript
aurelia.use.globalResources('./my-view-engine-hooks');
```

{% hint style="info" %}
The View Resource Pipeline is one of Aurelia's most extensible features. By creating custom hooks and resource types, you can integrate virtually any asset type into your templates.
{% endhint %}
