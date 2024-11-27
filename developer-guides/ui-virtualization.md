# UI Virtualization

### Overview

UI Virtualization is a powerful technique for efficiently rendering large data collections in web applications. Traditional rendering methods can lead to significant performance issues and browser memory constraints when dealing with extensive lists containing thousands or tens of thousands of items.

#### What is UI Virtualization?

UI Virtualization is a rendering strategy that only creates DOM elements for the items currently visible in the viewport. Instead of rendering the entire list at once, it dynamically renders and removes items as the user scrolls, creating a seamless and performant user experience.

#### Use Cases

* Large data grids
* Infinite scroll lists
* Performance-critical applications with extensive datasets
* Complex lists with thousands of items

#### Performance Benefits

* Reduced memory consumption
* Faster initial rendering
* Smooth scrolling experience
* Minimal DOM manipulation overhead

### Installation

#### NPM Installation

Install the UI Virtualization plugin using npm:

```shell
npm install aurelia-ui-virtualization
```

#### Plugin Configuration

In your `main.js`, configure the plugin during application startup:

```javascript
import { PLATFORM } from 'aurelia-pal';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .plugin(PLATFORM.moduleName('aurelia-ui-virtualization'));

  aurelia.start().then(a => a.setRoot());
}
```

{% hint style="info" %}
**Note:** Use `PLATFORM.moduleName()` when using Webpack to ensure proper module resolution.
{% endhint %}



### Basic Usage

The UI Virtualization plugin introduces the `virtual-repeat.for` attribute, which works similarly to Aurelia's standard `repeat.for` but with virtualization capabilities.

#### Syntax and Requirements

* Use `virtual-repeat.for` instead of `repeat.for`
* Rows must have a fixed height
* Only one item per row is supported

#### Examples

**Basic List Rendering**

```html
<template>
  <div virtual-repeat.for="item of items">
    ${$index} - ${item}
  </div>
</template>
```

**Unordered List**

```html
<template>
  <ul>
    <li virtual-repeat.for="item of items">
      ${$index} - ${item}
    </li>
  </ul>
</template>
```

**Table Row Virtualization**

```html
<template>
  <table>
    <tr virtual-repeat.for="item of items">
      <td>${$index}</td>
      <td>${item}</td>
    </tr>
  </table>
</template>
```

### Infinite Scrolling

The UI Virtualization plugin provides an `infinite-scroll-next` attribute for dynamically loading more items during scrolling.

#### Scroll Context Parameters

When triggered, the callback receives three arguments:

1. `topIndex`: Current top item index
2. `isAtBottom`: Boolean indicating bottom of list
3. `isAtTop`: Boolean indicating top of list

**Method 1: Direct Function**

```html
<template>
  <div 
    virtual-repeat.for="item of items" 
    infinite-scroll-next="loadMoreItems">
    ${item}
  </div>
</template>
```

```javascript
export class App {
  items = ['Initial', 'Data'];

  loadMoreItems(topIndex, isAtBottom, isAtTop) {
    if (isAtBottom) {
      // Append more items
      this.items.push(...newItems);
    }
  }
}
```

**Method 2: Call Binding**

```html
<template>
  <div 
    virtual-repeat.for="item of items" 
    infinite-scroll-next.call="loadMoreItems($scrollContext)">
    ${item}
  </div>
</template>
```

```javascript
export class App {
  loadMoreItems(scrollContext) {
    // More flexible scroll context handling
  }
}
```

### Advanced Techniques

#### Nested Elements

```html
<template>
  <div virtual-repeat.for="person of persons">
    <template with.bind="person">
      ${name}
    </template>
  </div>
</template>
```

#### CSS and Styling

Use contextual properties for styling:

```html
<template>
  <div 
    virtual-repeat.for="person of persons" 
    class="${$odd ? 'odd' : 'even'}-row">
    ${person.name}
  </div>
</template>
```

### Caveats and Limitations

#### Template Restrictions

* `<template>` cannot be the root element
* Other template controllers (with, if) cannot be used directly

#### CSS Selector Limitations

* `:nth-child` and similar selectors may behave unexpectedly
* Use contextual properties for conditional styling

#### Lifecycle Considerations

* Item views trigger bind/unbind and attach/detach during rendering
* Be mindful of performance in complex scenarios

By understanding these principles, you can effectively implement UI Virtualization in your Aurelia applications, ensuring smooth performance with large datasets.
