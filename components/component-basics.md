---
description: >-
  At its core, every Aurelia app is built from components. This guide will walk
  you through creating simple components, leveraging dependency injection, and
  mastering the component lifecycle.
---

# Component basics

Aurelia's UI components are built using view and view-model pairs. Views are HTML-based and render to the DOM, while view-models are JavaScript files that supply data and behavior. The framework's robust data-binding system connects these elements, ensuring that data changes are reflected in the view and vice versa.

A basic example of data-binding using `.bind` and `${}` interpolation looks like this:

{% code title="person.js" %}
```javascript
export class Person {
  name = 'Donald Draper';
}
```
{% endcode %}

{% code title="person.html" %}
```html
<template>
  <label for="name">Enter Name:</label>
  <input id="name" type="text" value.bind="name">
  <p>Name is ${name}</p>
</template>
```
{% endcode %}

{% hint style="info" %}
You can see a working example of the above [here](https://gist.dumber.app/?gist=ef456dac744e9d1af4c5b2f46d5ccf19).
{% endhint %}

Your view-model's raw data may not always be display-ready. This is especially common with dates and numbers. Fortunately, Aurelia provides elegant solutions for formatting these values directly in your templates, ensuring a polished UI without cluttering your view-model logic.

```javascript
export class NetWorth {
  constructor() {
    this.update();
    setInterval(() => this.update(), 1000);
  }

  update() {
    this.currentDate = new Date();
    this.netWorth = Math.random() * 1000000000;
  }
}
```

```html
<template>
  ${currentDate} <br>
  ${netWorth}
</template>
```

{% hint style="info" %}
You can see a working example of the above [here](https://gist.dumber.app/?gist=d9b4210940f52e9d70d8adf7db8fae40).
{% endhint %}

While displaying raw date and currency values works, it's not ideal for user readability. You could create formatted properties in your view-model, but this approach can become unwieldy, especially when keeping these properties in sync with changing values. Luckily, Aurelia offers a cleaner solution to this common problem called [Value Converters](../templates/value-converters-pipes.md).

## Creating a component

Aurelia's UI components consist of view and view-model pairs. The view, written in HTML, renders to the DOM, while the view-model, written in ES Next, provides data and behavior. The Templating Engine and Dependency Injection create these pairs and manage the component lifecycle.

Aurelia's databinding system links the view and view-model, ensuring changes in one are reflected in the other. This separation of concerns enhances collaboration, maintainability, and flexibility.

To create a UI component, simply make two files: one for the view-model and one for the view. For example, to create a "Hello" component, you'd need:

1. hello.ts/js (view-model)
2. hello.html (view)

{% tabs %}
{% tab title="Javascript" %}
```javascript
export class Hello {
  constructor() {
    this.firstName = 'John';
    this.lastName = 'Doe';
  }

  sayHello() {
    alert(`Hello ${this.firstName} ${this.lastName}. Nice to meet you.`);
  }
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
export class Hello {
  firstName: string = 'John';
  lastName: string = 'Doe';

  sayHello() {
    alert(`Hello ${this.firstName} ${this.lastName}. Nice to meet you.`);
  }
}
```
{% endtab %}
{% endtabs %}

Aurelia's view-models are simple classes, showcasing one of its core strengths: the ability to use vanilla JavaScript for much of your application.

Views in Aurelia use standard HTML templates, wrapped in Web Components' HTMLTemplateElement. The binding syntax is straightforward - just append `.bind` to any HTML attribute, and Aurelia will link it to the corresponding view-model property.

By default, `.bind` uses `one-way` binding for most attributes, updating the view when the view-model changes. For form controls, it defaults to `two-way` binding, allowing data to flow both ways. You can be explicit about binding direction using `.one-way`, `.two-way`, or `.one-time` (useful for static data, improving performance).

Event binding is also supported. Use `.trigger` on any event, native or custom, to invoke an expression when the event fires.

This flexible, intuitive approach simplifies the development process while maintaining full control over your application's behavior.

## Explicit component creation using @customElement

The `@customElement` decorator provides a way to define components, bypassing conventions explicitly. In the example below, we specify the component and its name.

```javascript
import { customElement } from 'aurelia-framework';

@customElement('hello')
export class Hello {
  constructor() {
    this.firstName = 'John';
    this.lastName = 'Doe';
  }

  sayHello() {
    alert(`Hello ${this.firstName} ${this.lastName}. Nice to meet you.`);
  }
}
```

## Components Without a View-Model

Sometimes, you may only need a template without any associated logic. In such cases, you can create a component with just a view:

{% code title="hello.html" %}
```html
<!-- hello.html -->
<template>
  <h1>Hello, World!</h1>
</template>
```
{% endcode %}

To use this component:

```html
<hello></hello>
```

## Components Without a View

Conversely, you can create a component with only a view-model:

{% code title="no-view.js" %}
```javascript
// no-view.js
export class NoView {
  message = 'I have no view!';
}
```
{% endcode %}

Use the `@noView()` decorator to indicate that this component has no view:

```javascript
import { noView } from 'aurelia-framework';

@noView()
export class NoView {
  message = 'I have no view!';
}
```

## SVG Elements

SVG (scalable vector graphic) tags can support Aurelia's custom element `<template>` tags by nesting the templated code inside a second `<svg>` tag. For example, if you had a base `<svg>` element and wanted to add a templated `<rect>` inside it, you would first put your custom tag inside the main `<svg>` tag. Also, make sure the custom element class uses the `@containerless()` decorator.

```javascript
import {containerless} from 'aurelia-framework';

@containerless()
export class MyCustomRect {
  ...
}
```

```html
<template>
  <svg>
    <rect width="10" height="10" fill="red" x="50" y="50"/>
  </svg>
</template>
```

To use it:

```html
<template>
  <require from="my-custom-rect"></require>

  <svg width="100" height="100" >
    <my-custom-rect></my-custom-rect>
  </svg>
</template>
```

## Custom Element Options

* `@children(selector)` - Decorates a property to create an array on your class that automatically synchronizes items based on a query selector against the element's immediate child content. Does not work with `@containerless()`, see below.
* `@child(selector)` - Decorates a property to create a reference to a single immediate child content element. Does not work with `@containerless()`, see below.
* `@processContent(false|Function)` - Tells the compiler that the element's content requires special processing. If you provide `false` to the decorator, the compiler will not process the content of your custom element. You are expected to do custom processing yourself. But you can also supply a custom function to process the content during the view's compilation. That function can then return true/false to indicate whether or not the compiler should also process the content. The function takes the following form `function(compiler, resources, node, instruction):boolean`
* `@useView(path)` - Specifies a different view to use.
* `@noView()` - Indicates that this custom element does not have a view and that the author intends for the element to handle its own rendering internally.
* `@inlineView(markup, dependencies?)` - Allows the developer to provide a string that will be compiled into the view.
* `@useShadowDOM(options?: { mode: 'open' | 'closed' })` - Causes the view to be rendered in the ShadowDOM. When an element is rendered to ShadowDOM, a special `DOMBoundary` instance can optionally be injected into the constructor. This represents the shadow root. Does not work with `@containerless()`, see below.
* `@containerless()` - Causes the element's view to be rendered without wrapping the custom element container. This cannot be used in conjunction with `@child`, `@children` or `@useShadowDOM` decorators. It also cannot be uses with surrogate behaviors. Use sparingly.

## Custom Element Variable Binding

It's worth noting that when binding variables to custom elements, **use camelCase inside the custom element's View-Model and dash-case on the html element**. See the following example:

{% code title="view-model.js" %}
```javascript
import {bindable} from 'aurelia-framework';

export class SayHello {
  @bindable to;
  @bindable greetingCallback;

  speak(){
    this.greetingCallback(`Hello ${this.to}!`);
  }
}
```
{% endcode %}

{% code title="view-model.html" %}
```html
<template>
  <require from="./say-hello"></require>

  <input type="text" ref="name">
  <say-hello to.bind="name.value" greeting-callback.call="doSomething($event)"></say-hello>
</template>
```
{% endcode %}

## Template Parts

Template parts in Aurelia 1 are a powerful feature that allows you to define reusable sections within your templates. They provide a way to organize and structure your view, making it more modular and easier to maintain.

### Basic Usage

To define a template part, use the `<template part="name">` syntax:

```html
<template>
  <template part="header">
    <h1>Welcome to My App</h1>
  </template>
  
  <main>
    <!-- Main content here -->
  </main>
  
  <template part="footer">
    <p>&copy; 2024 My Company</p>
  </template>
</template>
```

### Referencing Parts

You can reference these parts in other templates using the `part` attribute:

```html
<template>
  <compose view-model="./page-layout">
    <header replace-part="header">
      <h2>Custom Header for This Page</h2>
    </header>
  </compose>
</template>
```

### Dynamic Parts

Template parts can also be dynamic, allowing for conditional rendering:

```html
<template part="user-info">
  <div if.bind="isLoggedIn">
    Welcome, ${username}!
  </div>
  <div else>
    Please log in.
  </div>
</template>
```

### Nested Parts

Parts can be nested within other parts for more complex structures:

```html
<template part="sidebar">
  <nav>
    <template part="menu-items">
      <ul>
        <li><a href="#home">Home</a></li>
        <li><a href="#about">About</a></li>
      </ul>
    </template>
  </nav>
</template>
```

## Basic content projection

So far, we've only talked about custom elements that look like `<custom-element attr.bind="vmProp"></custom-element>`. Now it's time to create custom elements with content inside them. Let's create a name tag custom element. When the `name-tag` element is used, it will take the name and display it as content in the element.

```html
<name-tag>
  Ralphie
</name-tag>
```

Aurelia custom elements utilize the "slot based" content projection standard from the Web Component specifications. Let's look at how this will work with our `name-tag` element. This custom element utilizes a single slot, so we simply need to add a `<slot></slot>` element in our template where we would like content to be projected.

{% code title="name-tag.html" %}
```html
<template>
  <div class="header">
    Hello, my name is
  </div>
  <div class="name">
    <slot></slot>
  </div>
</template>
```
{% endcode %}

Aurelia will project the element's content in to the template where the `<slot></slot>` element is located.

## Declarative computed values

Aurelia provides a powerful and flexible way to handle computed values in your applications. As your custom elements grow more complex, you'll often need to work with values that are derived from other properties. This document explores the various methods to handle computed values in Aurelia, with a focus on the declarative `let` element.

### Traditional Methods

Before diving into the `let` element, let's review Aurelia's conventional methods for handling computed values.

#### Simple Interpolation

The most straightforward method is to use interpolation directly in the view:

```html
<div>
  First name: <input value.bind="firstName">
  Last name: <input value.bind="lastName">
</div>
Full name is: "${firstName} ${lastName}"
```

This approach works for simple cases but becomes unwieldy when you reuse the computed value or when the computation is more complex.

#### View Model Computed Properties

You can create computed properties in your view model for more control and reusability. Aurelia supports two main approaches:

**1. Manual Computation**

```javascript
export class App {
  @bindable firstName;
  @bindable lastName;

  firstNameChanged(newFirstName) {
    this.fullName = `${newFirstName} ${this.lastName}`;
  }

  lastNameChanged(newLastName) {
    this.fullName = `${this.firstName} ${newLastName}`;
  }
}
```

This method uses Aurelia's change callbacks to manually update the `fullName` property whenever `firstName` or `lastName` changes.

**2. Getter with `@computedFrom` Decorator**

```javascript
export class App {
  @bindable firstName;
  @bindable lastName;

  @computedFrom('firstName', 'lastName')
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

The `@computedFrom` decorator tells Aurelia which properties to observe for changes, allowing for efficient updates of the computed property.

### The `let` Element

Aurelia introduces the `let` element as a more declarative and view-centric approach to computed values. This element simplifies creating and using computed values directly in your templates.

#### Basic Usage

The `let` element can be used with either interpolation or binding expressions:

**Interpolation**

```html
<let full-name="${firstName} ${lastName}"></let>
<div>
  First name: <input value.bind="firstName">
  Last name: <input value.bind="lastName">
</div>
Full name is: "${fullName}"
```

**Binding Expression**

```html
<let full-name.bind="firstName + ' ' + lastName"></let>
<div>
  First name: <input value.bind="firstName">
  Last name: <input value.bind="lastName">
</div>
Full name is: "${fullName}"
```

In both cases, `fullName` is automatically recomputed whenever `firstName` or `lastName` changes without the need for explicit change handlers in the view model.

#### Binding to View Model

By default, the `let` element creates properties in the view's override context. If you need to access the computed value in your view model, use the `to-binding-context` attribute:

```html
<let to-binding-context full-name="${firstName} ${lastName}"></let>
<div>
  First name: <input value.bind="firstName">
  Last name: <input value.bind="lastName">
</div>
Full name is: "${fullName}"
```

This approach allows you to react to changes in the computed value within your view model:

```javascript
export class App {
  @bindable firstName;
  @bindable lastName;

  @observable fullName;

  fullNameChanged(fullName) {
    // React to changes in fullName
    console.log('Full name updated:', fullName);
  }
}
```

### Advantages of the `let` Element

1. **Declarative Syntax**: Keeps your templates clean and easy to understand.
2. **Reduced View Model Complexity**: Minimizes the need for manual change handlers or computed properties in the view model.
3. **Automatic Dependency Tracking**: Aurelia automatically tracks dependencies and updates the computed value when needed.
4. **Flexibility**: Can be used for simple string concatenations or more complex computations.
5. **Reusability**: Computed values can be easily reused throughout the template.

### Best Practices

1. Use `let` for view-specific computations that don't require complex logic.
2. Prefer `to-binding-context` when you need to react to changes in the computed value from the view model.
3. Consider using view model methods instead for complex computations or those used across multiple views.
4. Be mindful of performance: while convenient, overuse of `let` elements could impact rendering performance in large templates.

## Surrogate behaviors

Surrogate behaviors allow you to add attributes, event handlers, and bindings on the template element for a custom element. This can be extremely useful in many cases, but one particular area that it is helpful is with dealing with `aria` attributes to help add accessibility to your custom elements. When using surrogate behaviors, you add attributes to the template element for your custom element. These attributes will be placed on the custom element itself at runtime. For example, consider the view for a `my-button` custom element:

{% code title="my-button.html" %}
```html
<template role="button">
  <div>My Button</div>
</template>
```
{% endcode %}

```html
<template>
  <require from="my-button"></require>

  <my-button></my-button>
</template>
```

The `role="button"` attribute will automatically be set on the `my-button` element whenever it used in an Aurelia application. If you were to check your browser's Dev Tools while running a template that used the `my-buttom` custom element, you will see something that looks like the below

```html
<my-button class="au-target" au-target-id="1" role="button">
  <div>My Button</div>
</my-button>
```

It is important to note that Surrogate Behaviors cannot be used with a custom element that is using the `@containerless` decorator discussed below as this decorator removes the wrapping custom element from the DOM, and thus, there is nowhere for the Surrogate Behaviors to be placed.

## Advanced component example

In modern web applications, it is common to interact with external data sources. Aurelia makes it easy to integrate API calls within your components. This example demonstrates how to fetch data from an external API using Aurelia's `HttpClient`. We'll create a component that retrieves and displays a list of users, showcasing Aurelia's ability to handle asynchronous operations and update the UI accordingly.

### Step 1: Define the View

Create an HTML file named `data-fetcher.html`.

{% code title="data-fetcher.html" %}
```html
<template>
  <h3>Data from API</h3>
  <button click.delegate="fetchData()">Fetch Data</button>
  <ul>
    <li repeat.for="item of dataList">${item.name}</li>
  </ul>
</template>
```

### Step 2: Define the View-Model

Create a JavaScript file named `data-fetcher.js`.

{% code title="data-fetcher.js" %}
```javascript
import { HttpClient } from 'aurelia-fetch-client';

export class DataFetcher {
  static inject = [HttpClient];

  constructor(http) {
    this.http = http;
    this.dataList = [];
  }

  fetchData() {
    this.http.fetch('https://jsonplaceholder.typicode.com/users')
      .then(response => response.json())
      .then(data => {
        this.dataList = data;
      });
  }
}
```

#### Step 3: Using the Data Fetcher Component

Include the component in your main HTML file (such as `app.html`).

```html
<template>
  <require from="./data-fetcher"></require>

  <data-fetcher></data-fetcher>
</template>
```

The `DataFetcher` component exemplifies how Aurelia can seamlessly handle asynchronous data fetching within a user interface. By utilizing Aurelia's `HttpClient`, this component initiates an API call to retrieve a list of users from a remote server. The fetched data is then bound to the component's view, ensuring the user interface is dynamically updated to display the retrieved information. Through declarative data binding and event handling, the component stays responsive, allowing users to trigger data fetching via a button click, while Aurelia efficiently manages the UI updates in response to the asynchronous operation.
