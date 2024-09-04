# Styling components

Aurelia provides powerful and flexible ways to manage CSS classes and inline styles for your elements. This documentation covers the various methods to manipulate classes and styles, including working with ShadowDOM.

## Class Bindings

Aurelia provides several ways to bind an element's class attribute dynamically. These methods allow you to add or remove classes based on your application's state.

### String Interpolation

You can use string interpolation to apply classes conditionally:

```html
<div class="foo ${isActive ? 'active' : ''} bar"></div>
```

This approach allows you to combine static classes with dynamic ones.

### Binding Expressions

Alternatively, you can use `.bind` or `.to-view` to set classes:

```html
<div class.bind="isActive ? 'active' : ''"></div>
```

For one-time bindings, use `.one-time`:

```html
<div class.one-time="isActive ? 'active' : ''"></div>
```

### Class Name Binding

For performance-critical situations, you can bind directly to the `className` property:

```html
<div class-name.bind="isActive ? 'active' : ''"></div>
```

This method is faster but less safe, as it may overwrite classes added by other code.

## Style Bindings

Aurelia allows you to bind styles to elements using both string and object formats.

### String Binding

You can bind a CSS string directly to the `style` attribute:

```html
<div style.bind="styleString"></div>
```

Where `styleString` in your view-model might look like:

```javascript
this.styleString = 'color: red; background-color: blue';
```

### Object Binding

You can also bind a style object:

```html
<div style.bind="styleObject"></div>
```

With a corresponding view-model:

```javascript
this.styleObject = {
  color: 'red',
  'background-color': 'blue'
};
```

## The `css` Attribute

When using string interpolation for styles, use the `css` attribute instead of `style` to ensure compatibility with Internet Explorer and Edge:

```html
<div css="width: ${width}px; height: ${height}px;"></div>
```

Note that `css` should only be used when you need interpolation. For static styles, use the standard `style` attribute:

```html
<div style="width: 100px; height: 100px;"></div>
```

## Working with Shadow DOM

When using Shadow DOM, class and style bindings work similarly but only affect the shadow root's elements. To style the host element, you'll need to use `:host` in your CSS:

```css
:host {
  display: block;
  border: 1px solid #ccc;
}
```

