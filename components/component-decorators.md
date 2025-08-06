# Component decorators

Aurelia provides a comprehensive set of decorators that allow you to customize the behavior of custom elements and components. These decorators give you fine-grained control over how components are created, rendered, and interact with their environment.

## Core Component Decorators

### @customElement

The `@customElement` decorator allows you to explicitly define a custom element and override naming conventions.

```javascript
import { customElement } from 'aurelia-framework';

@customElement('my-widget')
export class SpecialWidget {
  // Component logic
}
```

**Usage:**
```html
<my-widget></my-widget>
```

**Benefits:**
- Override default naming conventions
- Create more semantic element names
- Avoid naming conflicts

### @bindable

The `@bindable` decorator makes properties available for data binding from parent components.

```javascript
import { bindable, bindingMode } from 'aurelia-framework';

export class UserCard {
  @bindable user;
  @bindable({ defaultBindingMode: bindingMode.twoWay }) isSelected;
  @bindable({ primaryProperty: true }) title;
  @bindable({ changeHandler: 'userDataChanged' }) userData;
  
  userDataChanged(newValue, oldValue) {
    console.log('User data changed:', { newValue, oldValue });
  }
}
```

**Configuration Options:**
- `defaultBindingMode`: Controls data flow direction
- `primaryProperty`: Makes this the default property for the element
- `changeHandler`: Specifies a custom change callback method name

## View and Template Decorators

### @useView

Specifies an alternative view file for the component.

```javascript
import { useView } from 'aurelia-framework';

@useView('./alternative-template.html')
export class MyComponent {
  message = 'Hello from alternative view!';
}
```

**Use Cases:**
- A/B testing different layouts
- Conditional template selection
- Shared view-models with different presentations

### @inlineView

Defines the component's template inline within the JavaScript/TypeScript file.

```javascript
import { inlineView } from 'aurelia-framework';

@inlineView(`
  <template>
    <h1>Inline Template</h1>
    <p>\${message}</p>
    <button click.delegate="sayHello()">Click me</button>
  </template>
`)
export class InlineComponent {
  message = 'This template is defined inline!';
  
  sayHello() {
    alert(`Hello! ${this.message}`);
  }
}
```

**Benefits:**
- Single-file components
- Better for simple components
- Eliminates need for separate .html files

**Advanced Usage with Dependencies:**
```javascript
@inlineView(`
  <template>
    <require from="./value-converter"></require>
    <div>\${data | myConverter}</div>
  </template>
`, [
  './value-converter'
])
export class InlineWithDeps {
  data = 'Transform me';
}
```

### @noView

Indicates that the component handles its own rendering and doesn't need a view file.

```javascript
import { noView, inject } from 'aurelia-framework';

@noView()
@inject(Element)
export class CanvasChart {
  constructor(element) {
    this.element = element;
  }

  attached() {
    // Create canvas and draw chart
    const canvas = document.createElement('canvas');
    this.element.appendChild(canvas);
    this.drawChart(canvas);
  }

  drawChart(canvas) {
    const ctx = canvas.getContext('2d');
    // Chart drawing logic
  }
}
```

## DOM and Rendering Decorators

### @containerless

Removes the custom element wrapper from the rendered output.

```javascript
import { containerless } from 'aurelia-framework';

@containerless()
export class InlineText {
  @bindable text;
}
```

**Template:**
```html
<template>${text}</template>
```

**Usage and Output:**
```html
<!-- Usage -->
<div>
  Before
  <inline-text text="middle"></inline-text>
  After
</div>

<!-- Rendered output (without @containerless) -->
<div>
  Before
  <inline-text>middle</inline-text>
  After
</div>

<!-- Rendered output (with @containerless) -->
<div>
  Before
  middle
  After
</div>
```

**Important Limitations:**
- Cannot use with `@child` or `@children`
- Cannot use with `@useShadowDOM`
- Cannot use with surrogate behaviors
- Use sparingly as it can affect styling and event handling

### @useShadowDOM

Renders the component using Shadow DOM for complete style encapsulation.

```javascript
import { useShadowDOM, inject } from 'aurelia-framework';

@useShadowDOM({ mode: 'open' })  // or 'closed'
@inject(Element)
export class IsolatedComponent {
  constructor(element) {
    this.element = element;
  }

  attached() {
    // Can inject DOMBoundary to access shadow root
    console.log('Shadow root:', this.element.shadowRoot);
  }
}
```

**Template:**
```html
<template>
  <style>
    :host {
      display: block;
      padding: 20px;
      border: 2px solid blue;
    }
    
    p {
      color: red;
      font-weight: bold;
    }
  </style>
  
  <p>This paragraph is styled only within this component</p>
  <slot></slot>
</template>
```

**Advanced Shadow DOM with DOMBoundary:**
```javascript
import { useShadowDOM, inject } from 'aurelia-framework';
import { DOMBoundary } from 'aurelia-templating';

@useShadowDOM()
@inject(Element, DOMBoundary)
export class AdvancedShadowComponent {
  constructor(element, domBoundary) {
    this.element = element;
    this.shadowRoot = domBoundary;
  }

  attached() {
    // Direct access to shadow root
    this.shadowRoot.addEventListener('custom-event', this.handleEvent.bind(this));
  }
}
```

## Content and Children Decorators

### @child

Creates a reference to a single child element matching the selector.

```javascript
import { child, inject } from 'aurelia-framework';

@inject(Element)
export class ParentComponent {
  @child('.primary-button') primaryButton;
  @child('input[type="text"]') textInput;

  attached() {
    console.log('Primary button element:', this.primaryButton);
    
    if (this.textInput) {
      this.textInput.focus();
    }
  }
}
```

**Template:**
```html
<template>
  <div>
    <input type="text" placeholder="Enter text">
    <button class="primary-button">Primary Action</button>
    <button class="secondary-button">Secondary Action</button>
  </div>
</template>
```

**Limitations:**
- Cannot use with `@containerless`
- Only selects immediate children
- Returns the first matching element

### @children

Creates an array of all child elements matching the selector.

```javascript
import { children } from 'aurelia-framework';

export class TabContainer {
  @children('.tab-pane') tabPanes;

  attached() {
    console.log(`Found ${this.tabPanes.length} tab panes`);
    this.tabPanes.forEach((pane, index) => {
      pane.setAttribute('data-tab-index', index);
    });
  }

  switchToTab(index) {
    this.tabPanes.forEach((pane, i) => {
      pane.style.display = i === index ? 'block' : 'none';
    });
  }
}
```

**Template:**
```html
<template>
  <div class="tab-headers">
    <button repeat.for="pane of tabPanes" 
            click.delegate="switchToTab($index)">
      Tab ${$index + 1}
    </button>
  </div>
  
  <div class="tab-content">
    <div class="tab-pane">Content 1</div>
    <div class="tab-pane">Content 2</div>
    <div class="tab-pane">Content 3</div>
  </div>
</template>
```

**Advanced Usage with Dynamic Children:**
```javascript
export class DynamicList {
  @children('.list-item') listItems;
  items = ['Item 1', 'Item 2', 'Item 3'];

  childrenChanged(newItems, oldItems) {
    console.log(`List items changed: ${oldItems?.length || 0} -> ${newItems.length}`);
    this.updateItemIndices();
  }

  updateItemIndices() {
    this.listItems.forEach((item, index) => {
      item.setAttribute('data-index', index);
    });
  }

  addItem() {
    this.items.push(`Item ${this.items.length + 1}`);
  }
}
```

## Content Processing Decorators

### @processContent

Controls how the component's content is processed during compilation.

```javascript
import { processContent } from 'aurelia-framework';

// Disable content processing entirely
@processContent(false)
export class RawContentComponent {
  // Component handles its own content processing
}

// Custom content processing
@processContent((compiler, resources, node, instruction) => {
  // Custom processing logic
  const children = node.childNodes;
  
  for (let i = 0; i < children.length; i++) {
    const child = children[i];
    if (child.nodeType === Node.TEXT_NODE) {
      // Transform text content
      child.textContent = child.textContent.toUpperCase();
    }
  }
  
  // Return true to continue with normal processing
  // Return false to skip normal processing
  return true;
})
export class TextTransformComponent {
  // Component with custom content transformation
}
```

**Advanced Content Processing Example:**
```javascript
@processContent((compiler, resources, node, instruction) => {
  // Find all 'data-template' attributes and convert to repeaters
  const dataTemplateElements = node.querySelectorAll('[data-template]');
  
  dataTemplateElements.forEach(element => {
    const templateType = element.getAttribute('data-template');
    element.removeAttribute('data-template');
    
    switch (templateType) {
      case 'repeater':
        element.setAttribute('repeat.for', 'item of items');
        break;
      case 'conditional':
        element.setAttribute('if.bind', 'isVisible');
        break;
    }
  });
  
  return true; // Continue normal processing
})
export class TemplateProcessor {
  items = [1, 2, 3];
  isVisible = true;
}
```

## Practical Examples

### Media Player Component

```javascript
import { 
  customElement, 
  bindable, 
  child, 
  children, 
  inlineView,
  inject
} from 'aurelia-framework';

@customElement('media-player')
@inlineView(`
  <template>
    <div class="media-container">
      <video ref="videoElement" 
             src.bind="source"
             poster.bind="poster">
      </video>
      
      <div class="controls">
        <button class="play-pause" click.delegate="togglePlayback()">
          \${isPlaying ? 'Pause' : 'Play'}
        </button>
        <input type="range" 
               class="progress-bar"
               min="0" 
               max.bind="duration"
               value.bind="currentTime">
        <button class="control-btn volume" click.delegate="toggleMute()">
          \${isMuted ? '🔇' : '🔊'}
        </button>
      </div>
    </div>
  </template>
`)
export class MediaPlayer {
  @bindable source;
  @bindable poster;
  @bindable({ defaultBindingMode: bindingMode.twoWay }) isPlaying = false;

  @child('video') videoElement;
  @children('.control-btn') controlButtons;

  duration = 0;
  currentTime = 0;
  isMuted = false;

  attached() {
    this.videoElement.addEventListener('loadedmetadata', () => {
      this.duration = this.videoElement.duration;
    });

    this.videoElement.addEventListener('timeupdate', () => {
      this.currentTime = this.videoElement.currentTime;
    });

    // Style control buttons
    this.controlButtons.forEach(btn => {
      btn.style.margin = '0 5px';
    });
  }

  togglePlayback() {
    if (this.isPlaying) {
      this.videoElement.pause();
    } else {
      this.videoElement.play();
    }
    this.isPlaying = !this.isPlaying;
  }

  toggleMute() {
    this.videoElement.muted = !this.videoElement.muted;
    this.isMuted = this.videoElement.muted;
  }
}
```

### Data Visualization Component

```javascript
import { 
  customElement, 
  noView, 
  bindable, 
  inject,
  processContent
} from 'aurelia-framework';

@customElement('data-chart')
@noView()
@inject(Element)
@processContent((compiler, resources, node, instruction) => {
  // Process data attributes and convert to configuration
  const dataElements = node.querySelectorAll('[data-series]');
  const config = { series: [] };
  
  dataElements.forEach(element => {
    const seriesName = element.getAttribute('data-series');
    const values = element.textContent.split(',').map(v => parseFloat(v.trim()));
    
    config.series.push({
      name: seriesName,
      data: values
    });
    
    element.remove(); // Remove from DOM
  });
  
  // Store config on the instruction for the component
  instruction.chartConfig = config;
  
  return false; // Skip normal content processing
})
export class DataChart {
  @bindable type = 'line';
  @bindable title;
  @bindable({ defaultBindingMode: bindingMode.oneTime }) data;

  constructor(element) {
    this.element = element;
  }

  bind(bindingContext, overrideContext, instruction) {
    // Access processed config
    if (instruction && instruction.chartConfig) {
      this.processedData = instruction.chartConfig;
    }
  }

  attached() {
    this.renderChart();
  }

  renderChart() {
    // Create canvas for chart
    const canvas = document.createElement('canvas');
    canvas.width = 400;
    canvas.height = 300;
    this.element.appendChild(canvas);

    const ctx = canvas.getContext('2d');
    
    // Simple chart rendering (in practice, you'd use a library like Chart.js)
    this.drawChart(ctx, this.processedData || this.data);
  }

  drawChart(ctx, data) {
    // Chart rendering logic
    ctx.fillStyle = '#333';
    ctx.font = '16px Arial';
    ctx.fillText(this.title || 'Chart', 10, 30);
    
    // Draw series data
    if (data && data.series) {
      data.series.forEach((series, index) => {
        this.drawSeries(ctx, series, index);
      });
    }
  }

  drawSeries(ctx, series, index) {
    const colors = ['red', 'blue', 'green', 'orange'];
    ctx.strokeStyle = colors[index % colors.length];
    ctx.beginPath();
    
    series.data.forEach((value, i) => {
      const x = 50 + (i * 20);
      const y = 250 - (value * 5);
      
      if (i === 0) {
        ctx.moveTo(x, y);
      } else {
        ctx.lineTo(x, y);
      }
    });
    
    ctx.stroke();
  }
}
```

**Usage:**
```html
<data-chart type="line" title="Sales Data">
  <div data-series="Q1">10, 20, 30, 25</div>
  <div data-series="Q2">15, 25, 35, 30</div>
</data-chart>
```

## Best Practices

### 1. Choose the Right Decorator Combination

```javascript
// Good: Logical decorator combination
@customElement('tooltip')
@containerless()
@inlineView('<template><span class="tooltip">${content}</span></template>')
export class SimpleTooltip {
  @bindable content;
}

// Avoid: Conflicting decorators
@containerless()
@useShadowDOM()  // These conflict!
export class ConflictingComponent { }
```

### 2. Use @child and @children Appropriately

```javascript
// Good: Specific, meaningful selectors
export class FormValidator {
  @children('input[required], select[required]') requiredFields;
  @child('.submit-button') submitButton;
}

// Avoid: Overly broad selectors
export class BadValidator {
  @children('*') allChildren; // Too broad, performance impact
}
```

### 3. Manage Memory in Custom Decorators

```javascript
@useShadowDOM()
export class MemoryAwareComponent {
  @children('.dynamic-item') dynamicItems;

  attached() {
    // Set up event listeners
    this.boundHandler = this.handleEvent.bind(this);
    document.addEventListener('global-event', this.boundHandler);
  }

  detached() {
    // Always clean up
    document.removeEventListener('global-event', this.boundHandler);
    this.dynamicItems = null;
  }
}
```

### 4. Use @processContent Judiciously

```javascript
// Good: Simple, focused content processing
@processContent((compiler, resources, node) => {
  // Simple transformation
  const textNodes = node.querySelectorAll('.uppercase');
  textNodes.forEach(n => n.textContent = n.textContent.toUpperCase());
  return true;
})
export class TextProcessor { }

// Avoid: Complex processing that should be in the view-model
@processContent((compiler, resources, node) => {
  // This complex logic should be elsewhere
  // ... 100 lines of complex DOM manipulation
})
export class OverComplexProcessor { }
```

Component decorators in Aurelia provide powerful ways to customize component behavior, but they should be used thoughtfully. Understanding when and how to use each decorator will help you create more maintainable and efficient components.