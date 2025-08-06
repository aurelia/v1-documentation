# HTML behaviors

HTML behaviors in Aurelia allow you to extend HTML with reusable, encapsulated functionality. They come in two primary forms: Custom Elements and Custom Attributes. Understanding these behaviors is fundamental to creating modular, maintainable Aurelia applications.

## Understanding HTML Behaviors

HTML behaviors enable you to:
- Create reusable UI components
- Encapsulate complex functionality
- Extend existing HTML elements with new capabilities
- Build a library of domain-specific components

### The Two Types of HTML Behaviors

#### Custom Elements
Custom elements create entirely new HTML elements with their own templates, view-models, and behavior.

```html
<!-- Custom element usage -->
<user-profile user.bind="currentUser"></user-profile>
<data-grid items.bind="products" columns.bind="gridColumns"></data-grid>
```

#### Custom Attributes  
Custom attributes add behavior to existing HTML elements without changing their fundamental nature.

```html
<!-- Custom attribute usage -->
<div tooltip="Click me for more info">Hover over me</div>
<input type="text" auto-focus validate="required|email">
```

## Creating HTML Behaviors

### Naming Conventions

Aurelia follows consistent naming conventions for HTML behaviors:

**JavaScript/TypeScript Class Names:**
- Use PascalCase with descriptive suffixes
- Custom Elements: `UserProfileCustomElement` or just `UserProfile`
- Custom Attributes: `TooltipCustomAttribute` or just `Tooltip`

**HTML Usage:**
- Automatically converted to kebab-case
- `UserProfile` becomes `<user-profile>`
- `TooltipCustomAttribute` becomes `tooltip="..."`

### Resource Registration

There are multiple ways to make HTML behaviors available in your application:

#### Local Registration (Per-Template)
```html
<template>
  <require from="./user-profile"></require>
  <require from="./tooltip"></require>
  
  <user-profile user.bind="currentUser"></user-profile>
  <div tooltip="Information">Hover me</div>
</template>
```

#### Global Registration
```javascript
// main.js
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .globalResources([
      PLATFORM.moduleName('resources/elements/user-profile'),
      PLATFORM.moduleName('resources/attributes/tooltip')
    ]);

  aurelia.start().then(() => aurelia.setRoot());
}
```

#### Feature-Based Registration
```javascript
// resources/index.js
import { PLATFORM } from 'aurelia-pal';

export function configure(config) {
  config.globalResources([
    PLATFORM.moduleName('./elements/user-profile'),
    PLATFORM.moduleName('./attributes/tooltip'),
    PLATFORM.moduleName('./value-converters/date-format')
  ]);
}

// main.js
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .feature(PLATFORM.moduleName('resources'));
}
```

## Custom Element Fundamentals

### Basic Custom Element Structure

**user-card.js**
```javascript
export class UserCard {
  // Bindable properties
  @bindable user;
  @bindable showDetails = false;

  // Lifecycle methods
  bind() {
    console.log('UserCard binding with user:', this.user);
  }

  // Component methods
  toggleDetails() {
    this.showDetails = !this.showDetails;
  }
}
```

**user-card.html**
```html
<template>
  <div class="user-card">
    <img src.bind="user.avatar" alt="User avatar">
    <h3>${user.name}</h3>
    <p>${user.email}</p>
    
    <button click.delegate="toggleDetails()">
      ${showDetails ? 'Hide' : 'Show'} Details
    </button>
    
    <div if.bind="showDetails" class="details">
      <p>Department: ${user.department}</p>
      <p>Role: ${user.role}</p>
    </div>
  </div>
</template>
```

### Custom Element with Content Projection

**modal-dialog.html**
```html
<template>
  <div class="modal-overlay" if.bind="isOpen">
    <div class="modal-content">
      <header class="modal-header">
        <slot name="header">
          <h2>Modal Title</h2>
        </slot>
        <button click.delegate="close()" class="close-btn">&times;</button>
      </header>
      
      <main class="modal-body">
        <slot>
          <p>Default modal content</p>
        </slot>
      </main>
      
      <footer class="modal-footer">
        <slot name="footer">
          <button click.delegate="close()">Close</button>
        </slot>
      </footer>
    </div>
  </div>
</template>
```

Usage:
```html
<modal-dialog is-open.bind="showModal">
  <h3 slot="header">Confirm Action</h3>
  <p>Are you sure you want to delete this item?</p>
  <div slot="footer">
    <button click.delegate="confirmDelete()">Delete</button>
    <button click.delegate="showModal = false">Cancel</button>
  </div>
</modal-dialog>
```

## Custom Attribute Fundamentals

### Simple Custom Attribute

**highlight.js**
```javascript
import { customAttribute, inject } from 'aurelia-framework';

@customAttribute('highlight')
@inject(Element)
export class Highlight {
  constructor(element) {
    this.element = element;
  }

  valueChanged(newValue, oldValue) {
    this.element.style.backgroundColor = newValue || 'yellow';
  }

  attached() {
    this.element.style.transition = 'background-color 0.3s ease';
  }
}
```

Usage:
```html
<p highlight.bind="highlightColor">This text can be highlighted</p>
<div highlight="red">This div has a red background</div>
```

### Multi-Property Custom Attribute

**border-style.js**
```javascript
import { customAttribute, bindable } from 'aurelia-framework';

@customAttribute('border-style')
export class BorderStyle {
  @bindable width = '1px';
  @bindable color = 'black';
  @bindable style = 'solid';
  @bindable radius = '0';

  bind() {
    this.updateBorder();
  }

  widthChanged() { this.updateBorder(); }
  colorChanged() { this.updateBorder(); }
  styleChanged() { this.updateBorder(); }
  radiusChanged() { this.updateBorder(); }

  updateBorder() {
    if (this.element) {
      this.element.style.border = `${this.width} ${this.style} ${this.color}`;
      this.element.style.borderRadius = this.radius;
    }
  }
}
```

Usage:
```html
<div border-style="width: 2px; color: blue; style: dashed; radius: 5px">
  Custom bordered content
</div>
```

## Lifecycle Integration

### Custom Element Lifecycle

```javascript
export class LifecycleExample {
  constructor() {
    console.log('1. Constructor called');
  }

  created(owningView, myView) {
    console.log('2. Created - view created');
  }

  bind(bindingContext, overrideContext) {
    console.log('3. Bind - data binding established');
  }

  attached() {
    console.log('4. Attached - added to DOM');
  }

  detached() {
    console.log('5. Detached - removed from DOM');
  }

  unbind() {
    console.log('6. Unbind - data binding removed');
  }
}
```

### Custom Attribute Lifecycle

```javascript
@customAttribute('lifecycle-demo')
export class LifecycleDemo {
  constructor() {
    console.log('Attribute constructor');
  }

  created(owningView, myView) {
    console.log('Attribute created');
  }

  bind(bindingContext, overrideContext) {
    console.log('Attribute bind');
  }

  attached() {
    console.log('Attribute attached');
    // Safe to manipulate DOM here
  }

  detached() {
    console.log('Attribute detached');
    // Cleanup DOM manipulations
  }

  unbind() {
    console.log('Attribute unbind');
  }
}
```

## Advanced HTML Behavior Patterns

### Behavior Composition

Combine multiple behaviors for powerful functionality:

```html
<div class="interactive-element"
     tooltip="Click me!"
     highlight.bind="isActive ? 'yellow' : 'transparent'"
     auto-focus.bind="shouldFocus"
     validate="required|minLength:3">
  Interactive content
</div>
```

### Behavior Inheritance

```javascript
// Base behavior
export class BaseElement {
  @bindable isVisible = true;
  
  attached() {
    this.setupCommonBehavior();
  }
  
  setupCommonBehavior() {
    // Common functionality
  }
}

// Inherited behavior
export class SpecializedElement extends BaseElement {
  @bindable specialProperty;
  
  attached() {
    super.attached(); // Call parent lifecycle
    this.setupSpecializedBehavior();
  }
  
  setupSpecializedBehavior() {
    // Specialized functionality
  }
}
```

### Dynamic Behavior Loading

```javascript
export class DynamicBehaviorHost {
  @bindable behaviorName;

  behaviorNameChanged(newBehavior) {
    if (newBehavior) {
      this.loadBehavior(newBehavior);
    }
  }

  loadBehavior(behaviorName) {
    // Dynamically load and apply behavior
    import(`./behaviors/${behaviorName}`).then(module => {
      // Apply the behavior
    });
  }
}
```

## Integration with Dependency Injection

### Injecting Services into Behaviors

```javascript
import { inject } from 'aurelia-framework';
import { ApiService } from './api-service';
import { EventAggregator } from 'aurelia-event-aggregator';

@inject(ApiService, EventAggregator)
export class DataBoundElement {
  constructor(apiService, eventAggregator) {
    this.apiService = apiService;
    this.eventAggregator = eventAggregator;
  }

  async attached() {
    this.data = await this.apiService.fetchData();
    this.eventAggregator.publish('data-loaded', this.data);
  }
}
```

### Behavior-Specific Services

```javascript
// behavior-helper.js
export class BehaviorHelper {
  formatData(data) {
    // Utility methods for behaviors
    return data.map(item => ({ ...item, formatted: true }));
  }
}

// custom-behavior.js
@inject(BehaviorHelper)
export class CustomBehavior {
  constructor(behaviorHelper) {
    this.helper = behaviorHelper;
  }

  processData(rawData) {
    return this.helper.formatData(rawData);
  }
}
```

## Testing HTML Behaviors

### Unit Testing Custom Elements

```javascript
import { StageComponent } from 'aurelia-testing';
import { bootstrap } from 'aurelia-bootstrapper';

describe('UserCard', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources('user-card')
      .inView('<user-card user.bind="user"></user-card>')
      .boundTo({ 
        user: { name: 'John Doe', email: 'john@example.com' } 
      });
  });

  afterEach(() => {
    component.dispose();
  });

  it('should display user name', async () => {
    await component.create(bootstrap);
    expect(component.element.textContent).toContain('John Doe');
  });
});
```

### Testing Custom Attributes

```javascript
describe('Highlight Attribute', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources('highlight')
      .inView('<div highlight.bind="color">Test</div>')
      .boundTo({ color: 'red' });
  });

  it('should apply background color', async () => {
    await component.create(bootstrap);
    const element = component.element.querySelector('div');
    expect(element.style.backgroundColor).toBe('red');
  });
});
```

## Performance Considerations

### Efficient Behavior Creation

```javascript
// Good: Minimal constructor logic
export class EfficientBehavior {
  constructor() {
    // Only essential initialization
  }
  
  attached() {
    // DOM manipulation and setup here
    this.setupExpensiveOperations();
  }
  
  detached() {
    // Always cleanup
    this.cleanupOperations();
  }
}
```

### Memory Management

```javascript
export class MemoryAwareBehavior {
  attached() {
    this.listener = this.handleEvent.bind(this);
    document.addEventListener('click', this.listener);
    
    this.timer = setInterval(() => {
      this.updateData();
    }, 1000);
  }

  detached() {
    // Critical: Remove event listeners
    document.removeEventListener('click', this.listener);
    
    // Clear timers
    if (this.timer) {
      clearInterval(this.timer);
      this.timer = null;
    }
  }
}
```

## Best Practices

### 1. Single Responsibility
Each behavior should have one clear purpose:

```javascript
// Good: Single purpose
export class ValidationHighlight {
  // Only handles validation highlighting
}

// Avoid: Multiple responsibilities
export class ValidationHighlightDataFetcherNotifier {
  // Too many responsibilities
}
```

### 2. Consistent Naming
Follow consistent naming patterns:

```javascript
// Elements: Domain + Purpose
export class UserProfile { }
export class ProductCard { }
export class OrderSummary { }

// Attributes: Action + Context  
export class ValidateInput { }
export class HighlightError { }
export class AutoFocus { }
```

### 3. Proper Lifecycle Usage
Use appropriate lifecycle hooks:

```javascript
export class ProperLifecycleBehavior {
  bind() {
    // Data binding setup
  }
  
  attached() {
    // DOM manipulation
    // Event listener setup
    // Third-party library initialization
  }
  
  detached() {
    // Cleanup DOM manipulations
    // Remove event listeners
    // Destroy third-party libraries
  }
}
```

### 4. Bindable Property Design
Design bindable properties thoughtfully:

```javascript
export class WellDesignedElement {
  @bindable({ defaultBindingMode: bindingMode.twoWay }) value;
  @bindable({ defaultBindingMode: bindingMode.oneTime }) config;
  @bindable({ defaultBindingMode: bindingMode.oneWay }) data;
  
  // Provide meaningful defaults
  @bindable size = 'medium';
  @bindable theme = 'default';
}
```

HTML behaviors are the foundation of component-based architecture in Aurelia. By understanding both custom elements and custom attributes, you can create powerful, reusable functionality that makes your applications more maintainable and scalable.