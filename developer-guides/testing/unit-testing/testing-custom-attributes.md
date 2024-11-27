# Testing Custom Attributes

Custom attributes in Aurelia are powerful ways to extend HTML elements with additional behavior. Testing these attributes requires a systematic approach to verify their functionality, binding, and interactions.

### Basic Custom Attribute Test Structure

#### Simple Attribute Implementation

```javascript
export class HighlightAttribute {
  static inject = [Element];

  constructor(element) {
    this.element = element;
  }

  valueChanged(newValue) {
    this.element.style.backgroundColor = newValue;
  }
}
```

#### Comprehensive Attribute Testing

```javascript
import {StageComponent} from 'aurelia-testing';
import {bootstrap} from 'aurelia-bootstrapper';

describe('Highlight Attribute', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources(PLATFORM.moduleName('highlight'))
      .inView('<div highlight.bind="color">Test Content</div>')
      .boundTo({ color: 'yellow' });
  });

  it('should apply background color', done => {
    component.create(bootstrap).then(() => {
      const element = component.element;
      expect(element.style.backgroundColor).toBe('yellow');
      done();
    }).catch(done.fail);
  });

  afterEach(() => {
    component.dispose();
  });
});
```

### Advanced Attribute Testing Scenarios

#### Dynamic Attribute Behavior

```javascript
describe('Dynamic Attribute Behavior', () => {
  let component;

  class TooltipAttribute {
    static inject = [Element];

    constructor(element) {
      this.element = element;
      this.tooltip = null;
    }

    valueChanged(newValue) {
      if (this.tooltip) {
        this.tooltip.remove();
      }

      if (newValue) {
        this.tooltip = document.createElement('div');
        this.tooltip.className = 'tooltip';
        this.tooltip.textContent = newValue;
        this.element.appendChild(this.tooltip);
      }
    }
  }

  beforeEach(() => {
    component = StageComponent
      .withResources(PLATFORM.moduleName('tooltip'))
      .inView('<div tooltip.bind="message">Hover Me</div>')
      .boundTo({ message: 'Hello World' });
  });

  it('should create and update tooltip dynamically', done => {
    component.create(bootstrap).then(() => {
      // Check initial tooltip creation
      const tooltipElement = component.element.querySelector('.tooltip');
      expect(tooltipElement).toBeTruthy();
      expect(tooltipElement.textContent).toBe('Hello World');

      // Update tooltip
      component.viewModel.message = 'New Tooltip';

      // Wait for binding to update
      return new Promise(resolve => setTimeout(resolve, 50));
    }).then(() => {
      const tooltipElement = component.element.querySelector('.tooltip');
      expect(tooltipElement.textContent).toBe('New Tooltip');

      // Remove tooltip
      component.viewModel.message = '';

      return new Promise(resolve => setTimeout(resolve, 50));
    }).then(() => {
      const tooltipElement = component.element.querySelector('.tooltip');
      expect(tooltipElement).toBeFalsy();

      done();
    }).catch(done.fail);
  });

  afterEach(() => {
    component.dispose();
  });
});
```

#### Attribute with Multiple Parameters

```javascript
describe('Complex Attribute with Multiple Parameters', () => {
  class StyleAttribute {
    static inject = [Element];

    constructor(element) {
      this.element = element;
    }

    bind(bindingContext, overrideContext) {
      const { color, fontSize, fontWeight } = bindingContext;
      
      if (color) this.element.style.color = color;
      if (fontSize) this.element.style.fontSize = `${fontSize}px`;
      if (fontWeight) this.element.style.fontWeight = fontWeight;
    }
  }

  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources(PLATFORM.moduleName('style-attribute'))
      .inView(`
        <div 
          style.bind="{
            color: textColor, 
            fontSize: size, 
            fontWeight: weight
          }">
          Styled Text
        </div>
      `)
      .boundTo({
        textColor: 'red',
        size: 16,
        weight: 'bold'
      });
  });

  it('should apply multiple style attributes', done => {
    component.create(bootstrap).then(() => {
      const element = component.element;
      
      expect(element.style.color).toBe('red');
      expect(element.style.fontSize).toBe('16px');
      expect(element.style.fontWeight).toBe('bold');

      done();
    }).catch(done.fail);
  });

  afterEach(() => {
    component.dispose();
  });
});

```

### Lifecycle and Interaction Testing

#### Attribute Lifecycle Methods

```javascript
describe('Attribute Lifecycle', () => {
  class LoggingAttribute {
    static inject = [Element];

    constructor(element) {
      this.element = element;
      this.lifecycleEvents = [];
    }

    bind() {
      this.lifecycleEvents.push('bind');
    }

    attached() {
      this.lifecycleEvents.push('attached');
    }

    detached() {
      this.lifecycleEvents.push('detached');
    }

    unbind() {
      this.lifecycleEvents.push('unbind');
    }
  }

  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources(PLATFORM.moduleName('logging'))
      .inView('<div logging>Test</div>');
  });

  it('should trigger lifecycle methods', done => {
    component.manuallyHandleLifecycle().create(bootstrap)
      .then(() => {
        // No lifecycle methods called initially
        expect(component.viewModel.lifecycleEvents.length).toBe(0);

        // Manually trigger bind
        return component.bind();
      })
      .then(() => {
        expect(component.viewModel.lifecycleEvents).toContain('bind');

        // Trigger attached
        return component.attached();
      })
      .then(() => {
        expect(component.viewModel.lifecycleEvents).toContain('attached');

        // Trigger detached
        return component.detached();
      })
      .then(() => {
        expect(component.viewModel.lifecycleEvents).toContain('detached');

        // Trigger unbind
        return component.unbind();
      })
      .then(() => {
        expect(component.viewModel.lifecycleEvents).toContain('unbind');
        done();
      })
      .catch(done.fail);
  });
});
```

### Testing Strategies and Best Practices

* Test initial attribute application
* Verify dynamic updates
* Check lifecycle method invocations
* Test edge cases and different input types
* Ensure proper cleanup and disposal
* Mock external dependencies
* Use meaningful assertions

#### Common Pitfalls to Avoid

* Don't test implementation details
* Avoid over-complex test scenarios
* Ensure proper async handling
* Clean up DOM between tests
* Be cautious of timing-related tests
* Use meaningful and descriptive test names
