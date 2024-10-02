# Observation

Aurelia’s observation system is a core feature that enables seamless data binding between your application’s model and view. This document provides a comprehensive guide to Aurelia's observation mechanisms, including the Observer Locator, the @observable decorator, and other strategies for observing changes in your application state.

Aurelia’s observation system is designed to efficiently detect changes in your application’s data models and automatically update the views. This two-way data binding ensures the user interface remains in sync with the underlying data without manual intervention.

Key features of the observation system include:

• Automatic Change Detection: Monitors changes to properties and collections.

• Efficient Updates: Updates the UI only when necessary, improving performance.

• Extensibility: Allows for custom observation strategies.

## Observer Locator

The Observer Locator is a service in Aurelia that determines the best way to observe a property or collection for changes.

The Observer Locator is responsible for:

• Identifying the appropriate observer for a given property or object.

• Providing low-level observation APIs.

• Managing the lifecycle of observers.

It acts as a central hub that other parts of the Aurelia framework use to subscribe to changes.

### Using the observer locator

While Aurelia’s binding system typically handles observation automatically, you can use the Observer Locator directly when you need fine-grained control.

### Injecting the Observer Locator

```typescript
import { inject } from 'aurelia-framework';
import { ObserverLocator } from 'aurelia-binding';

@inject(ObserverLocator)
export class MyViewModel {
  constructor(observerLocator) {
    this.observerLocator = observerLocator;
  }
}
```

### Observing a property

```typescript
let propertyObserver = this.observerLocator.getObserver(object, 'propertyName');
propertyObserver.subscribe((newValue, oldValue) => {
  console.log(`Property changed from ${oldValue} to ${newValue}`);
});
```

### Observing an Array

```typescript
let arrayObserver = this.observerLocator.getArrayObserver(array);
arrayObserver.subscribe(splices => {
  splices.forEach(splice => {
    console.log(`Array changed: ${JSON.stringify(splice)}`);
  });
});
```

{% hint style="info" %}
Use direct observation sparingly. Rely on Aurelia’s binding system to keep your code clean and maintainable for most scenarios.
{% endhint %}

## The @observable Decorator

The @observable decorator provides a convenient way to make class properties observable, enabling Aurelia to monitor changes and update the view accordingly.

### Understanding the @observable Decorator

When you apply the @observable decorator to a property, Aurelia:

• Generates a property with a getter and setter that notifies Aurelia’s binding system upon changes.

• Creates a corresponding change handler method if you follow the naming convention (propertyNameChanged).

• Allows for two-way data binding between the property and the view.

### Using @observable in your components

#### Importing the Decorator

```typescript
import { observable } from 'aurelia-framework';
```

#### Applying the Decorator

```typescript
export class MyViewModel {
  @observable myProperty;

  // Optional: Implement a change handler
  myPropertyChanged(newValue, oldValue) {
    console.log(`myProperty changed from ${oldValue} to ${newValue}`);
  }
}
```

#### Setting an Initial Value

```typescript
@observable myProperty = 'Initial Value';
```

## Example usage

{% code title="counter.ts" %}
```typescript
import { observable } from 'aurelia-framework';

export class Counter {
  @observable count = 0;

  countChanged(newValue, oldValue) {
    console.log(`Count changed from ${oldValue} to ${newValue}`);
  }

  increment() {
    this.count++;
  }

  decrement() {
    this.count--;
  }
}
```
{% endcode %}

{% code title="counter.html" %}
```html
<template>
  <div>Current Count: ${count}</div>
  <button click.delegate="increment()">Increment</button>
  <button click.delegate="decrement()">Decrement</button>
</template>
```
{% endcode %}

#### Explanation:

• The count property is decorated with @observable, making it observable by Aurelia.

• The countChanged method is automatically called whenever count changes.

• The view displays the current value of count and provides buttons to modify it.

• The UI updates automatically when count changes, thanks to Aurelia’s data binding.

## Collection Observation

For collections like arrays, maps, and sets, Aurelia observes mutations to keep the UI in sync.

### Array Observation

• Observes methods like push, pop, splice, shift, unshift, and reverse.

• Automatically updates bindings when the array changes.

```typescript
import { inject } from 'aurelia-framework';
import { ObserverLocator } from 'aurelia-binding';

@inject(ObserverLocator)
export class ItemList {
  items = [];

  constructor(observerLocator) {
    this.observerLocator = observerLocator;

    let arrayObserver = this.observerLocator.getArrayObserver(this.items);
    arrayObserver.subscribe(splices => {
      splices.forEach(splice => {
        console.log(`Array changed: ${JSON.stringify(splice)}`);
      });
    });
  }

  addItem(item) {
    this.items.push(item);
  }
}
```

```markup
<template>
  <ul>
    <li repeat.for="item of items">${item}</li>
  </ul>
  <input type="text" value.bind="newItem" />
  <button click.delegate="addItem(newItem)">Add Item</button>
</template>
```

## Computed Observation

Aurelia can observe computed properties that depend on other observable properties using the @computedFrom decorator.

```typescript
import { computedFrom } from 'aurelia-framework';

export class Person {
  firstName = 'John';
  lastName = 'Doe';

  @computedFrom('firstName', 'lastName')
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

#### Explanation

• The fullName getter is decorated with @computedFrom, specifying its dependencies.

• Aurelia only recalculates fullName when firstName or lastName changes.

• Improves performance by avoiding unnecessary recalculations and updates.

```markup
<template>
  <div>Full Name: ${fullName}</div>
  <input value.bind="firstName" placeholder="First Name" />
  <input value.bind="lastName" placeholder="Last Name" />
</template>
```

## Custom Observers

Creating custom observers allows you to define how Aurelia observes changes in non-standard objects or properties.

**Implementing a Custom Observer:**

• Create a class that implements the PropertyObserver interface.

• Implement the required methods: getValue, setValue, subscribe, and unsubscribe.

```typescript
import { subscriberCollection } from 'aurelia-binding';

@subscriberCollection()
export class CustomPropertyObserver {
  constructor(target, propertyName) {
    this.target = target;
    this.propertyName = propertyName;
  }

  getValue() {
    return this.target[this.propertyName];
  }

  setValue(newValue) {
    let oldValue = this.target[this.propertyName];
    if (newValue !== oldValue) {
      this.target[this.propertyName] = newValue;
      this.callSubscribers(newValue, oldValue);
    }
  }

  subscribe(callback) {
    this.addSubscriber(callback);
  }

  unsubscribe(callback) {
    this.removeSubscriber(callback);
  }
}
```

