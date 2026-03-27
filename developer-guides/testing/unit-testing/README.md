# Unit Testing

Unit testing in Aurelia allows you to verify individual components, services, and behaviors in isolation. Aurelia's architecture is designed with testability in mind — its dependency injection system makes it straightforward to mock dependencies and test components independently.

## Getting Started

Aurelia provides the `aurelia-testing` library, which gives you tools to stage and test custom elements and attributes in a mini Aurelia application environment.

### Installation

```bash
npm install aurelia-testing --save-dev
```

### Basic Test Setup

A typical unit test uses `StageComponent` from `aurelia-testing` along with a test runner like Jest or Jasmine:

```javascript
import { StageComponent } from 'aurelia-testing';
import { bootstrap } from 'aurelia-bootstrapper';

describe('MyComponent', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources('my-component')
      .inView('<my-component></my-component>');
  });

  afterEach(() => {
    component.dispose();
  });

  it('should render', done => {
    component.create(bootstrap).then(() => {
      expect(document.querySelector('my-component')).not.toBeNull();
      done();
    }).catch(done.fail);
  });
});
```

### Testing Without the DOM

For view-models that don't require DOM interaction, you can test them directly:

```javascript
import { MyService } from '../src/my-service';

describe('MyService', () => {
  let service;

  beforeEach(() => {
    service = new MyService();
  });

  it('should process data correctly', () => {
    const result = service.processData('input');
    expect(result).toBe('expected output');
  });
});
```

## What's Covered

The following sections provide detailed guidance on testing specific aspects of Aurelia applications:

- [Testing Custom Elements](testing-custom-elements.md) — Test your custom components with data binding and lifecycle methods
- [Testing Custom Attributes](testing-custom-attributes.md) — Verify custom attribute behavior
- [Testing Routing and Navigation](testing-routing-and-navigation.md) — Test router configuration and navigation
- [Advanced Testing](advanced-testing.md) — Complex testing scenarios and patterns
- [Helpful Properties, Functions, and Advanced Querying](helpful-properties-functions-and-advanced-querying.md) — Utility methods for testing
- [Dependency Injection Testing](dependency-injection-testing.md) — Mock and test DI-managed services
- [Testing Value Converters and Binding Behaviors](testing-custom-value-converters-and-binding-behaviors.md) — Test custom pipes and behaviors
- [Internationalization Testing](internationalization-i18n-testing.md) — Test i18n integrations

{% hint style="info" %}
Always call `component.dispose()` in your `afterEach` block to clean up the DOM and prevent test pollution between test cases.
{% endhint %}
