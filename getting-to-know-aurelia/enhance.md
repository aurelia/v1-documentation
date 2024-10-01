# Enhance

Aurelia provides the `enhance` functionality to progressively enhance existing HTML pages or parts of them. This allows you to add Aurelia's powerful data-binding and templating features to static content without managing routing or overhauling your entire application into a single-page app.

## What Is Enhance in Aurelia?

The `enhance` method enables you to apply Aurelia's binding and templating capabilities to existing DOM elements. Instead of bootstrapping a full Aurelia application that replaces the content of your `<body>` or a root element, `enhance` You can target specific elements, enhancing them with Aurelia's features while keeping the rest of the page intact.

## When to Use Enhance

Use the `enhance` functionality when you:

* Have server-side rendered HTML that you want to augment with client-side interactions.
* Need to enhance a part of the page without controlling the entire application's bootstrapping process.
* Are integrating Aurelia into an existing application incrementally.
* Want to add dynamic behavior to static HTML without restructuring your application into a full SPA.

## How to Use Enhance

### Enhancing the Entire Page

To enhance the entire page, you can call the `enhance` method on the `Aurelia` instance after configuring it:

```javascript
aurelia.use.standardConfiguration();

aurelia.start().then(() => {
  aurelia.enhance();
});
```

This will compile and bind the entire DOM using an empty binding context.

### Enhancing Part of a Page

To enhance a specific element or portion of the page, you can pass the element and an optional binding context to the `enhance` method:

```javascript
let element = document.querySelector('#partial-area');
let viewModel = { message: 'Hello from Aurelia!' };

aurelia.start().then(() => {
  aurelia.enhance(viewModel, element);
});
```

Alternatively, using the `TemplatingEngine`:

```javascript
import { TemplatingEngine } from 'aurelia-templating';

export class App {
  static inject = [TemplatingEngine];

  constructor(templatingEngine) {
    this.templatingEngine = templatingEngine;
  }

  attached() {
    let element = document.querySelector('#partial-area');
    let viewModel = { title: 'Hello from Aurelia!', description: 'This is a description from enhance.' };

    this.templatingEngine.enhance({
      element: element,
      bindingContext: viewModel
    });
  }
}
```

Then inside your HTML, you might have something like this:

```markup
<div>
  <p>This content is not enhanced by Aurelia.</p>
  <div id="partial-area">
    <h2>${title}</h2>
    <p>${description}</p>
  </div>
</div>
```

## Registering Resources for Enhanced Content

When enhancing content that uses custom elements, attributes, value converters, or binding behaviors, you need to ensure that these resources are registered with Aurelia. The enhanced content will not recognize them if they are not registered globally, leading to binding errors or unexpected behavior.

### Global Resource Registration

You can register resources globally using the `globalResources` method during Aurelia's configuration phase. This makes the resources available throughout your application, including in enhanced content.

Suppose you have a custom element `my-element` defined in `my-element.ts`:

{% code title="my-element.ts" %}
```typescript
export class MyElement {
  // Custom element logic here
}
```
{% endcode %}

Inside of your `main.ts/main.js` file:

```typescript
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from 'aurelia-pal';

let aurelia = new Aurelia();
aurelia.use
  .standardConfiguration()
  .globalResources(PLATFORM.moduleName('my-element'));

aurelia.start().then(() => {
  aurelia.enhance();
});
```

* **Bootstrap Timing:** Ensure that Aurelia is started (`aurelia.start()`) before calling `enhance`. The `enhance` method should be called within the promise returned by `start()`. You can also await the `start` method.
* **Binding Context:** Providing a binding context is optional. If omitted, the view will use an empty binding context, and bindings may not render as expected.
* **Resources Registration:** If your enhanced content uses custom elements or attributes, ensure they are globally registered or specify the necessary resources during Aurelia's configuration.
* **Multiple Enhancements:** You can call `enhance` multiple times on different elements with different binding contexts if needed.
