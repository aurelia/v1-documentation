# Binding engine internals

Understanding how Aurelia's binding system works under the hood can help you build better applications, debug complex binding issues, and make informed architectural decisions. This guide explores the internal mechanisms of Aurelia's binding engine.

## High-Level Architecture

Aurelia's binding system consists of several key components that work together to provide seamless data binding:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Aurelia Binding System                       │
├─────────────────────────────────────────────────────────────────┤
│  ViewCompiler                                                   │
│  ├─── Identifies binding expressions in templates               │
│  └─── Creates BindingExpression instances                       │
├─────────────────────────────────────────────────────────────────┤
│  Parser & AST                                                   │
│  ├─── Tokenizes binding expressions                             │
│  ├─── Creates Abstract Syntax Tree (AST)                       │
│  └─── Handles expression evaluation                             │
├─────────────────────────────────────────────────────────────────┤
│  ObserverLocator                                                │
│  ├─── Finds appropriate observers for properties               │
│  ├─── Manages observer lifecycle                                │
│  └─── Coordinates change detection                              │
├─────────────────────────────────────────────────────────────────┤
│  Binding Instances                                              │
│  ├─── Connect model to view                                     │
│  ├─── Handle data flow and updates                              │
│  └─── Manage binding lifecycle                                  │
└─────────────────────────────────────────────────────────────────┘
```

## View Compilation Process

### 1. Template Parsing

When Aurelia processes a template, the ViewCompiler scans for binding expressions:

```html
<!-- Original template -->
<div class="${isActive ? 'active' : ''}" 
     click.delegate="handleClick($event)">
  ${message | uppercase}
</div>
```

The ViewCompiler identifies:
- String interpolations: `${isActive ? 'active' : ''}`
- Event bindings: `click.delegate="handleClick($event)"`
- Value converter bindings: `${message | uppercase}`

### 2. AST Generation

Each binding expression is parsed into an Abstract Syntax Tree (AST):

```javascript
// Expression: ${isActive ? 'active' : ''}
// Becomes an AST structure like:
{
  type: 'Conditional',
  condition: {
    type: 'AccessScope',
    name: 'isActive'
  },
  yes: {
    type: 'LiteralString',
    value: 'active'
  },
  no: {
    type: 'LiteralString', 
    value: ''
  }
}
```

### 3. Binding Expression Creation

The ViewCompiler creates BindingExpression objects that serve as factories for actual binding instances:

```javascript
// Simplified BindingExpression
class BindingExpression {
  constructor(observerLocator, targetProperty, sourceExpression, mode) {
    this.observerLocator = observerLocator;
    this.targetProperty = targetProperty;
    this.sourceExpression = sourceExpression; // The AST
    this.mode = mode; // oneTime, oneWay, twoWay, etc.
  }

  createBinding(target) {
    return new PropertyBinding(
      this.sourceExpression,
      target,
      this.targetProperty,
      this.mode,
      this.observerLocator
    );
  }
}
```

## Abstract Syntax Tree (AST) Details

The AST represents JavaScript expressions in a structured format. Here are the main AST node types:

### Expression Types

```javascript
// AccessScope: accessing a property from scope
// Expression: someProperty
{
  type: 'AccessScope',
  name: 'someProperty'
}

// AccessMember: accessing object member
// Expression: user.name
{
  type: 'AccessMember',
  object: { type: 'AccessScope', name: 'user' },
  name: 'name'
}

// CallFunction: function calls
// Expression: calculateTotal(items, tax)
{
  type: 'CallFunction',
  func: { type: 'AccessScope', name: 'calculateTotal' },
  args: [
    { type: 'AccessScope', name: 'items' },
    { type: 'AccessScope', name: 'tax' }
  ]
}

// Binary: binary operations
// Expression: price * quantity
{
  type: 'Binary',
  operation: '*',
  left: { type: 'AccessScope', name: 'price' },
  right: { type: 'AccessScope', name: 'quantity' }
}

// Conditional: ternary operator
// Expression: isVip ? 'Gold' : 'Silver'
{
  type: 'Conditional',
  condition: { type: 'AccessScope', name: 'isVip' },
  yes: { type: 'LiteralString', value: 'Gold' },
  no: { type: 'LiteralString', value: 'Silver' }
}
```

### AST Evaluation

AST nodes can evaluate themselves given a scope:

```javascript
// Simplified AST evaluation
class AccessScope {
  constructor(name) {
    this.name = name;
  }

  evaluate(scope, lookupFunctions) {
    return scope.bindingContext[this.name];
  }

  connect(binding, scope) {
    // Set up observation for this property
    const observer = binding.observerLocator.getObserver(
      scope.bindingContext, 
      this.name
    );
    
    observer.subscribe(binding.updateTarget.bind(binding));
    return observer;
  }
}
```

## Scope and Context

### Understanding Scope

The scope object contains the data available to binding expressions:

```javascript
// Scope structure
const scope = {
  bindingContext: viewModelInstance, // The view-model
  overrideContext: {
    // Contextual properties available in templates
    $this: viewModelInstance,
    $parent: parentScope?.bindingContext,
    $root: rootScope?.bindingContext,
    // Temporary context from repeater, etc.
    $index: 0,
    $first: true,
    $last: false,
    $even: true,
    $odd: false
  }
};
```

### Scope Traversal

When resolving property names, Aurelia searches through the scope chain:

```javascript
function getProperty(scope, name) {
  // 1. Check override context first (for $parent, $index, etc.)
  if (scope.overrideContext.hasOwnProperty(name)) {
    return scope.overrideContext[name];
  }
  
  // 2. Check binding context (view-model)
  if (scope.bindingContext && scope.bindingContext.hasOwnProperty(name)) {
    return scope.bindingContext[name];
  }
  
  // 3. Traverse up the scope chain
  if (scope.overrideContext.$parent) {
    return getProperty({
      bindingContext: scope.overrideContext.$parent,
      overrideContext: scope.overrideContext.$parent.$overrideContext
    }, name);
  }
  
  return undefined;
}
```

## Observer System

### Observer Types

Aurelia uses different observer types for different scenarios:

```javascript
// Property observers for regular object properties
class SetterObserver {
  constructor(taskQueue, obj, propertyName) {
    this.taskQueue = taskQueue;
    this.obj = obj;
    this.propertyName = propertyName;
    this.callbacks = [];
    this.queued = false;
    this.observing = false;
  }

  getValue() {
    return this.obj[this.propertyName];
  }

  setValue(newValue) {
    const oldValue = this.obj[this.propertyName];
    if (oldValue !== newValue) {
      this.obj[this.propertyName] = newValue;
      this.notify(newValue, oldValue);
    }
  }

  notify(newValue, oldValue) {
    if (this.queued || this.callbacks.length === 0) {
      return;
    }

    this.queued = true;
    this.taskQueue.queueMicroTask(() => {
      this.queued = false;
      this.callbacks.forEach(callback => callback(newValue, oldValue));
    });
  }
}

// Array observers for collections
class ArrayObserver {
  constructor(taskQueue, array) {
    this.taskQueue = taskQueue;
    this.array = array;
    this.callbacks = [];
    this.reset();
  }

  reset() {
    this.oldArray = this.array.slice(0);
  }

  // Called by array mutation methods
  addChangeRecord(changeRecord) {
    if (this.callbacks.length === 0) {
      return;
    }

    this.taskQueue.queueMicroTask(() => {
      this.callbacks.forEach(callback => callback([changeRecord]));
    });
  }
}
```

### Observer Locator

The ObserverLocator determines which type of observer to use:

```javascript
class ObserverLocator {
  getObserver(obj, propertyName) {
    // Check for @observable decorator
    if (obj.$observers && obj.$observers[propertyName]) {
      return obj.$observers[propertyName];
    }

    // Check for existing property descriptor
    const descriptor = getPropertyDescriptor(obj, propertyName);
    
    if (descriptor) {
      if (descriptor.get && descriptor.set) {
        // Property already has getter/setter
        return new GetterObserver(obj, propertyName, descriptor);
      }
      
      if (descriptor.writable === false || descriptor.set === undefined) {
        // Read-only property
        return new ReadOnlyObserver(obj, propertyName);
      }
    }

    // Create setter observer for normal properties
    return new SetterObserver(this.taskQueue, obj, propertyName);
  }

  getArrayObserver(array) {
    return new ArrayObserver(this.taskQueue, array);
  }
}
```

## Binding Lifecycle

### 1. Binding Creation

```javascript
class PropertyBinding {
  constructor(sourceExpression, target, targetProperty, mode, observerLocator) {
    this.sourceExpression = sourceExpression; // AST
    this.target = target; // DOM element or object
    this.targetProperty = targetProperty; // Property name
    this.mode = mode; // Binding mode
    this.observerLocator = observerLocator;
  }

  bind(scope) {
    this.scope = scope;
    
    // Connect source (model) observation
    if (this.mode !== bindingMode.oneTime) {
      this.sourceObserver = this.sourceExpression.connect(this, scope);
    }

    // Connect target (view) observation for two-way bindings
    if (this.mode === bindingMode.twoWay) {
      this.targetObserver = this.observerLocator.getObserver(
        this.target, 
        this.targetProperty
      );
      this.targetObserver.subscribe(this.updateSource.bind(this));
    }

    // Initial update
    this.updateTarget(this.sourceExpression.evaluate(scope));
  }

  unbind() {
    if (this.sourceObserver) {
      this.sourceObserver.unsubscribe(this.updateTarget.bind(this));
    }
    
    if (this.targetObserver) {
      this.targetObserver.unsubscribe(this.updateSource.bind(this));
    }
  }
}
```

### 2. Update Process

```javascript
// Source-to-target update (model to view)
updateTarget(value) {
  // Apply value converters if present
  if (this.valueConverter) {
    value = this.valueConverter.toView(
      value, 
      ...this.valueConverterArgs.map(arg => arg.evaluate(this.scope))
    );
  }

  // Apply binding behaviors
  if (this.bindingBehavior) {
    this.bindingBehavior.bind(this, this.scope);
  }

  // Update the target
  this.targetObserver.setValue(value);
}

// Target-to-source update (view to model)
updateSource(value) {
  // Apply value converters in reverse
  if (this.valueConverter && this.valueConverter.fromView) {
    value = this.valueConverter.fromView(
      value,
      ...this.valueConverterArgs.map(arg => arg.evaluate(this.scope))
    );
  }

  // Update the source
  this.sourceExpression.assign(this.scope, value);
}
```

## Binding Modes Deep Dive

### OneTime Binding

```javascript
class OneTimeBinding extends PropertyBinding {
  bind(scope) {
    this.scope = scope;
    
    // Evaluate once and disconnect
    const value = this.sourceExpression.evaluate(scope);
    this.updateTarget(value);
    
    // No ongoing observation needed
  }

  unbind() {
    // Nothing to clean up
  }
}
```

### OneWay Binding (To-View)

```javascript
class OneWayBinding extends PropertyBinding {
  bind(scope) {
    this.scope = scope;
    
    // Set up source observation only
    this.sourceObserver = this.sourceExpression.connect(this, scope);
    
    // Initial update
    this.updateTarget(this.sourceExpression.evaluate(scope));
  }

  // Only updateTarget is needed
  // No updateSource method
}
```

### TwoWay Binding

```javascript
class TwoWayBinding extends PropertyBinding {
  bind(scope) {
    this.scope = scope;
    
    // Set up bidirectional observation
    this.sourceObserver = this.sourceExpression.connect(this, scope);
    this.targetObserver = this.observerLocator.getObserver(
      this.target, 
      this.targetProperty
    );
    
    this.targetObserver.subscribe(this.updateSource.bind(this));
    
    // Initial update
    this.updateTarget(this.sourceExpression.evaluate(scope));
  }

  // Both updateTarget and updateSource are active
}
```

## Performance Optimizations

### 1. Observer Reuse

Aurelia reuses observers for the same property:

```javascript
class ObserverCache {
  constructor() {
    this.cache = new WeakMap();
  }

  getObserver(obj, propertyName) {
    let objCache = this.cache.get(obj);
    if (!objCache) {
      objCache = new Map();
      this.cache.set(obj, objCache);
    }

    let observer = objCache.get(propertyName);
    if (!observer) {
      observer = this.createObserver(obj, propertyName);
      objCache.set(propertyName, observer);
    }

    return observer;
  }
}
```

### 2. Batch Updates

Updates are batched using the TaskQueue:

```javascript
class TaskQueue {
  constructor() {
    this.microTaskQueue = [];
    this.flushing = false;
  }

  queueMicroTask(task) {
    this.microTaskQueue.push(task);
    
    if (!this.flushing) {
      this.flushing = true;
      
      // Use browser's microtask queue
      Promise.resolve().then(() => {
        this.flushMicroTaskQueue();
      });
    }
  }

  flushMicroTaskQueue() {
    while (this.microTaskQueue.length > 0) {
      const task = this.microTaskQueue.shift();
      try {
        task();
      } catch (error) {
        this.onError(error, task);
      }
    }
    
    this.flushing = false;
  }
}
```

### 3. Dirty Checking Optimization

For computed properties without `@computedFrom`:

```javascript
class DirtyChecker {
  constructor(taskQueue) {
    this.taskQueue = taskQueue;
    this.tracked = [];
    this.checkInterval = 120; // milliseconds
  }

  addProperty(obj, propertyName, callback) {
    const tracked = {
      obj,
      propertyName,
      callback,
      oldValue: obj[propertyName]
    };
    
    this.tracked.push(tracked);
    this.scheduleCheck();
  }

  check() {
    this.tracked.forEach(tracked => {
      const newValue = tracked.obj[tracked.propertyName];
      if (newValue !== tracked.oldValue) {
        const oldValue = tracked.oldValue;
        tracked.oldValue = newValue;
        tracked.callback(newValue, oldValue);
      }
    });
  }

  scheduleCheck() {
    if (!this.scheduled) {
      this.scheduled = true;
      setTimeout(() => {
        this.scheduled = false;
        this.check();
        this.scheduleCheck(); // Continue checking
      }, this.checkInterval);
    }
  }
}
```

## Advanced Binding Scenarios

### Custom Element with Bindable Properties

```javascript
// How @bindable properties work internally
class CustomElementViewModel {
  static $bindable = {
    value: { defaultBindingMode: bindingMode.twoWay },
    config: { defaultBindingMode: bindingMode.oneWay }
  };

  // Aurelia generates something like this:
  static getBindablePropertyName() {
    return 'value';
  }

  valueChanged(newValue, oldValue) {
    // Change callback
    console.log(`Value changed from ${oldValue} to ${newValue}`);
  }
}

// During view compilation, bindable properties become regular bindings
// <my-element value.bind="parentValue"> becomes:
// new PropertyBinding(
//   new AccessScope('parentValue'), // source
//   customElementInstance, // target  
//   'value', // target property
//   bindingMode.twoWay, // mode from @bindable config
//   observerLocator
// )
```

### Value Converter Integration

```javascript
class ValueConverterBinding extends PropertyBinding {
  constructor(sourceExpression, valueConverter, args, target, targetProperty, mode, observerLocator) {
    super(sourceExpression, target, targetProperty, mode, observerLocator);
    this.valueConverter = valueConverter;
    this.valueConverterArgs = args; // AST nodes for arguments
  }

  updateTarget(value) {
    // Convert model value to view value
    const convertedValue = this.valueConverter.toView(
      value,
      ...this.valueConverterArgs.map(arg => arg.evaluate(this.scope))
    );
    
    super.updateTarget(convertedValue);
  }

  updateSource(value) {
    // Convert view value back to model value
    let convertedValue = value;
    if (this.valueConverter.fromView) {
      convertedValue = this.valueConverter.fromView(
        value,
        ...this.valueConverterArgs.map(arg => arg.evaluate(this.scope))
      );
    }
    
    super.updateSource(convertedValue);
  }
}
```

## Debugging Binding Issues

### 1. Enable Binding Logging

```javascript
// In main.js
aurelia.use.developmentLogging('debug');

// Or specifically for binding:
import { LogManager } from 'aurelia-framework';
LogManager.getLogger('aurelia-binding').setLevel(LogManager.logLevel.debug);
```

### 2. Inspect Binding Context

```html
<!-- Use $this to inspect the current binding context -->
<div>${$this | json}</div>

<!-- Check parent context -->
<div>${$parent | json}</div>

<!-- Inspect specific properties -->
<div>Current scope: ${$this.constructor.name}</div>
```

### 3. Monitor Observer Activity

```javascript
// Monkey-patch observer for debugging
const originalNotify = SetterObserver.prototype.notify;
SetterObserver.prototype.notify = function(newValue, oldValue) {
  console.log(`Property ${this.propertyName} changed:`, {
    from: oldValue,
    to: newValue,
    object: this.obj
  });
  
  return originalNotify.call(this, newValue, oldValue);
};
```

## Common Performance Patterns

### 1. Use OneTime Binding for Static Content

```html
<!-- Static content that never changes -->
<h1>${title & oneTime}</h1>
<div class="${staticClass & oneTime}">Content</div>
```

### 2. Optimize Computed Properties

```javascript
// Instead of expensive recalculation
get expensiveValue() {
  return this.items.reduce((sum, item) => sum + this.calculateComplex(item), 0);
}

// Cache and invalidate
_expensiveValue = null;
_expensiveValueDirty = true;

get expensiveValue() {
  if (this._expensiveValueDirty) {
    this._expensiveValue = this.items.reduce((sum, item) => 
      sum + this.calculateComplex(item), 0
    );
    this._expensiveValueDirty = false;
  }
  return this._expensiveValue;
}

// Invalidate cache when dependencies change
itemsChanged() {
  this._expensiveValueDirty = true;
}
```

### 3. Minimize Deep Property Paths

```html
<!-- Less efficient - multiple property traversals -->
<div>${user.profile.settings.display.theme}</div>

<!-- More efficient - extract to computed property -->
<div>${displayTheme}</div>
```

```javascript
@computedFrom('user.profile.settings.display.theme')
get displayTheme() {
  return this.user.profile.settings.display.theme;
}
```

Understanding these internals helps you write more efficient Aurelia applications and debug complex binding scenarios. The binding system's architecture provides flexibility while maintaining performance through smart optimizations like observer reuse, batched updates, and efficient change detection.