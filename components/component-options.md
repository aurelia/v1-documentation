# Component options

Aurelia provides a set of powerful decorators that allow you to customize the behavior and rendering of custom elements. These decorators offer fine-grained control over how elements interact with their content, how they're rendered, and how they integrate with the DOM.

### @children(selector)

The `@children` decorator creates an array property on your class that is automatically synchronized with immediate child elements matching the specified selector.

#### Usage

```javascript
import { customElement, children } from 'aurelia-framework';

@customElement('parent-element')
export class ParentElement {
  @children('child-element')
  childElements;

  attached() {
    console.log(this.childElements.length); // Logs the number of child-element instances
  }
}
```

#### Behavior

* The decorated property becomes an array containing references to child elements matching the selector.
* The array is automatically updated when child elements are added or removed.
* Only immediate children are considered; descendants at deeper levels are not included.

#### Limitations

* Does not work with `@containerless()` elements.
* The selector must be a valid CSS selector string.

### @child(selector)

The `@child` decorator creates a property reference to a single immediate child element matching the specified selector.

#### Usage

```javascript
import { customElement, child } from 'aurelia-framework';

@customElement('parent-element')
export class ParentElement {
  @child('header')
  headerElement;

  attached() {
    if (this.headerElement) {
      console.log('Header element found:', this.headerElement);
    }
  }
}
```

#### Behavior

* The decorated property becomes a reference to the first child element matching the selector.
* If multiple children match the selector, only the first one is referenced.
* The reference is updated automatically if the matching child element changes.

#### Limitations

* Does not work with `@containerless()` elements.
* Only references a single element, even if multiple matches exist.

### @useView(path)

The `@useView` decorator specifies an alternative view template file for the custom element.

#### Usage

```javascript
import { useView, PLATFORM } from 'aurelia-framework';

@useView(PLATFORM.moduleName('./alternative-view.html'))
export class MyCustomElement {
  // Element logic here
}
```

#### Behavior

* Overrides the default view naming convention.
* The specified path is relative to the component's JavaScript file.
* Allows for more flexible view organization and reuse.

#### Considerations

* Ensure the path is correct to avoid runtime errors.
* Can be useful for creating multiple elements with similar logic but different views.

### @noView()

The `@noView` decorator indicates that the custom element does not have a view, allowing the element to handle its own rendering internally.

#### Usage

```javascript
import { customElement, noView } from 'aurelia-framework';

@customElement('my-element')
@noView()
export class MyElement {
  constructor() {
    this.element = null;
  }

  bind(bindingContext) {
    this.element.textContent = 'This element manages its own content';
  }
}
```

#### Behavior

* Prevents Aurelia from attempting to load or compile a view for this element.
* The element is responsible for managing its own content and rendering.

#### Use Cases

* Creating elements that render themselves dynamically.
* Wrapping third-party libraries that manage their own DOM.

### @inlineView(markup, dependencies?)

The `@inlineView` decorator allows the developer to provide a string that will be compiled into the view.

#### Usage

```javascript
import { customElement, inlineView } from 'aurelia-framework';

@customElement('greeting-element')
@inlineView('<template>Hello, ${name}!</template>')
export class GreetingElement {
  name = 'World';
}
```

#### Parameters

* `markup`: A string containing the HTML markup for the view.
* `dependencies` (optional): An array of view resource dependencies.

#### Behavior

* The provided markup string is compiled as the element's view.
* Useful for simple components or rapid prototyping.

#### Advanced Usage with Dependencies

```javascript
import { customElement, inlineView } from 'aurelia-framework';
import { MyCustomAttribute } from './my-custom-attribute';

@customElement('advanced-greeting')
@inlineView(
  '<template><div my-custom-attribute>Hello, ${name}!</div></template>',
  [MyCustomAttribute]
)
export class AdvancedGreeting {
  name = 'World';
}
```

### @useShadowDOM(options?)

The `@useShadowDOM` decorator causes the view to be rendered in the Shadow DOM, providing encapsulation for the element's styles and markup.

#### Usage

```javascript
import { customElement, useShadowDOM } from 'aurelia-framework';

@customElement('encapsulated-element')
@useShadowDOM({ mode: 'open' })
export class EncapsulatedElement {
  // Element logic here
}
```

#### Options

* `mode`: 'open' | 'closed'
  * 'open': Allows external JavaScript to access the shadow DOM.
  * 'closed': Denies external access to the shadow DOM.

#### Behavior

* Creates a shadow root for the custom element.
* Renders the element's view inside the shadow DOM.
* Provides style encapsulation.

#### Special Considerations

* When an element is rendered to Shadow DOM, a special `DOMBoundary` instance can be injected into the constructor, representing the shadow root.
* Not compatible with `@containerless()`.

### @containerless()

The `@containerless` decorator causes the element's view to be rendered without the custom element container wrapping it.

#### Usage

```javascript
import { customElement, containerless } from 'aurelia-framework';

@customElement('my-containerless-element')
@containerless()
export class MyContainerlessElement {
  // Element logic here
}
```

#### Behavior

* The element's view is inserted directly into the DOM, replacing the custom element tag.
* Useful for creating wrapper-less UI components.

#### Limitations

* Incompatible with `@child`, `@children`, and `@useShadowDOM` decorators.
* Cannot be used with surrogate behaviors.
* Use sparingly, as it can make debugging more challenging.

{% hint style="info" %}
#### Performance Consideration

While `@containerless()` can slightly improve rendering performance by reducing DOM depth; it should be used carefully as it can complicate the component structure and affect how the element interacts with the surrounding DOM.
{% endhint %}
