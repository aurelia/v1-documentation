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
```markup
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

```markup
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

Views in Aurelia use standard HTML templates, wrapped in Web Components' HTMLTemplateElement. The binding syntax is straightforward - just append .bind to any HTML attribute, and Aurelia will link it to the corresponding view-model property.

By default, .bind uses one-way binding for most attributes, updating the view when the view-model changes. For form controls, it defaults to two-way binding, allowing data to flow both ways. You can be explicit about binding direction using .one-way, .two-way, or .one-time (useful for static data, improving performance).

Event binding is also supported. Use .trigger on any event, native or custom, to invoke an expression when the event fires.

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
```markup
<!-- hello.html -->
<template>
  <h1>Hello, World!</h1>
</template>
```
{% endcode %}

To use this component:

```markup
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

```markup
<template>
  <svg>
    <rect width="10" height="10" fill="red" x="50" y="50"/>
  </svg>
</template>
```

To use it:

```markup
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
```markup
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

```markup
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

```markup
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

```markup
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

```markup
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

## Advanced component example

In modern web applications, it is common to interact with external data sources. Aurelia makes it easy to integrate API calls within your components. This example demonstrates how to fetch data from an external API using Aurelia's `HttpClient`. We'll create a component that retrieves and displays a list of users, showcasing Aurelia's ability to handle asynchronous operations and update the UI accordingly.

### Step 1: Define the View

Create an HTML file named `data-fetcher.html`.

```markup
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

```markup
<template>
  <require from="./data-fetcher"></require>

  <data-fetcher></data-fetcher>
</template>
```

The `DataFetcher` component exemplifies how Aurelia can seamlessly handle asynchronous data fetching within a user interface. By utilizing Aurelia's `HttpClient`, this component initiates an API call to retrieve a list of users from a remote server. The fetched data is then bound to the component's view, ensuring the user interface is dynamically updated to display the retrieved information. Through declarative data binding and event handling, the component stays responsive, allowing users to trigger data fetching via a button click, while Aurelia efficiently manages the UI updates in response to the asynchronous operation.
