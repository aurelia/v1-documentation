# Dynamic composition

Embracing the DRY (Don't Repeat Yourself) Principle doesn't mean we should tightly couple our view and view-model pairs. Thankfully, Aurelia offers a solution: a custom element that dynamically combines an HTML template, a view-model, and optional initialization data. This flexible approach allows for more modular and reusable code, making your development process smoother and more efficient.

## The `compose` Element

The `compose` element is a custom element provided by Aurelia that enables dynamic rendering of components based on runtime data or logic. It's highly versatile and can be used in various scenarios.

#### Basic Usage

At its simplest, you can use the `compose` element like this:

```html
<compose view-model="./components/user-dashboard"></compose>
```

This will load and render the `UserDashboard` component from the specified path.

## Composition Scenarios

### Composing Without a View Model

Sometimes, you might want to compose a view without a corresponding view-model. This is useful for simple templates that don't require any backing logic:

```html
<compose view="./templates/static-content.html"></compose>
```

In this case, Aurelia will render the `static-content.html` template without any associated view-model.

### Composing with a View Model

More commonly, you'll want to compose both a view and a view-model:

```html
<compose view-model="./components/product-list"></compose>
```

Here, Aurelia will look for both `product-list.js` (or `product-list.ts` for TypeScript) and `product-list.html` files in the `components` directory.

### Passing Data to Composed Components

One of the most powerful features of dynamic composition is the ability to pass data to composed components. You can do this using the `model` attribute:

```html
<compose view-model="./components/order-details" model.bind="currentOrder"></compose>
```

In the `OrderDetails` component, you can access this data through the `activate` lifecycle method:

```javascript
export class OrderDetails {
  activate(model) {
    this.order = model;
    this.orderItems = model.items;
    this.totalAmount = model.totalAmount;
  }
}
```

This allows you to pass complex objects or primitives to your dynamically composed components, making them highly reusable.

### Dynamic View-Model Selection

You can also dynamically select which view-model to compose based on runtime conditions:

```html
<compose view-model.bind="selectedComponent"></compose>
```

In your parent view-model:

```javascript
export class ProductPage {
  constructor() {
    this.selectedComponent = './components/product-grid';
  }

  switchToListView() {
    this.selectedComponent = './components/product-list';
  }

  switchToGridView() {
    this.selectedComponent = './components/product-grid';
  }
}
```

This allows you to switch between different views of your data based on user preference or other conditions.

## Composing with Custom View and View-Model

For maximum flexibility, you can specify both view and view-model separately:

```html
<compose
  view="./templates/custom-product-view.html"
  view-model="./view-models/product-view-model"
></compose>
```

This is particularly useful when you want to reuse a view-model with different views, or vice versa.

{% hint style="info" %}
While the full view lifecycle (created, bind, attached, detached, unbind) is supported during dynamic composition, the full navigation lifecycle is not. Only the `activate` hook is enabled. It receives a single parameter which is the `model` and can optionally return a promise if executing an asynchronous task.
{% endhint %}
