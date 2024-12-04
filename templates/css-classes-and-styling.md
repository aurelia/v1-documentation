# CSS classes and styling

### Class Binding in Aurelia

Aurelia provides flexible and powerful ways to dynamically manage CSS classes on HTML elements. You can bind classes conditionally, dynamically, and with different binding modes.

#### Basic Class Binding Syntax

Aurelia offers multiple ways to bind classes to an element:

1. String Interpolation

```html
<div class="foo ${isActive ? 'active' : ''} bar"></div>
```

2. Binding Directive

```html
<div class.bind="isActive ? 'active' : ''"></div>
```

3. One-Time Binding

```html
<div class.one-time="isActive ? 'active' : ''"></div>
```

#### Conditional Class Binding

You can apply classes based on component state or conditions:

```html
<template>
  <div class.bind="isError ? 'error-message' : 'normal-message'">
    ${message}
  </div>

  <button 
    class.bind="isDisabled ? 'disabled' : ''" 
    disabled.bind="isDisabled">
    Submit
  </button>
</template>
```

The binding system will only add or remove classes specified in the binding expression to ensure maximum interoperability with other JavaScript libraries. This ensures classes added by other code (eg via `classList.add(...)`) are preserved. This "safe by default" behavior comes at a small cost but can be noticeable in benchmarks or other performance-critical situations like repeats with lots of elements. You can opt out of the default behavior by binding directly to the element's [className](https://developer.mozilla.org/en-US/docs/Web/API/Element/className) property using `class-name.bind="...."` or `class-name.one-time="..."`. This will be marginally faster but can add up over a lot of bindings.

#### Binding Modes

Aurelia supports different binding modes for classes:

| Mode     | Syntax      | Behavior                                             |
| -------- | ----------- | ---------------------------------------------------- |
| Default  | `.bind`     | Updates dynamically, two-way binding on form inputs. |
| One-Time | `.one-time` | Sets class once, no future updates                   |
| One-Way  | `.to-view`  | Updates from view-model to view                      |

### Style Binding in Aurelia

Aurelia provides robust methods for dynamically binding styles to HTML elements, supporting string and object-based style assignments.

#### String-based Style Binding

You can bind styles using a string representation of CSS:

```javascript
export class StyleExample {
  constructor() {
    this.styleString = 'color: red; background-color: blue';
  }
}
```

```html
<template>
  <div style.bind="styleString"></div>
</template>
```

#### Object-based Style Binding

Bind styles using a JavaScript object for more structured style management:

```javascript
export class StyleObjectExample {
  constructor() {
    this.styleObject = {
      color: 'red',
      'background-color': 'blue',
      'font-weight': 'bold'
    };
  }
}
```

```html
<template>
  <div style.bind="styleObject"></div>
</template>
```

#### Style Interpolation

**Incorrect Interpolation**

```html
<!-- This will NOT work -->
<div style="width: ${width}px; height: ${height}px;"></div>
```

**Correct Interpolation**

```html
<!-- Use css custom attribute -->
<div css="width: ${width}px; height: ${height}px;"></div>
```

#### CSS Custom Attribute Usage

The `css` attribute is crucial for style interpolation, especially for compatibility with Internet Explorer and Edge:

```html
<template>
  <!-- Interpolation requires css attribute -->
  <div css="width: ${dynamicWidth}px"></div>

  <!-- Static styles work normally -->
  <div style="color: red;"></div>
</template>
```

#### Important Notes

* Use `css` for string interpolation
* Static styles can use standard `style` attribute
* Interpolated styles require the `css` attribute
* Objects and strings can both be bound to styles
