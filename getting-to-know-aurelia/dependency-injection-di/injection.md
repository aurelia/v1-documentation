# Injection

Injecting dependencies into your classes is straightforward in Aurelia. Aurelia provides multiple mechanisms to declare and manage dependencies, ensuring that each class receives the necessary instances it requires.

## Declaring Dependencies with `@inject`

The `@inject` decorator is the primary method for declaring a class's dependencies. It specifies which dependencies should be injected into the class upon instantiation.

**Example:**

```javascript
import { CustomerService } from 'backend/customer-service';
import { inject } from 'aurelia-framework';

@inject(CustomerService)
export class CustomerEditScreen {
  constructor(customerService) {
    this.customerService = customerService;
    this.customer = null;
  }

  activate(params) {
    return this.customerService.getCustomerById(params.customerId)
      .then(customer => this.customer = customer);
  }
}
```

**Key Points:**

* **Decorator Usage:** The `@inject` decorator lists the dependencies required by the class.
* **Constructor Matching:** The constructor parameters should match the dependencies declared in the `@inject` decorator, both in type and order.
* **Multiple Dependencies:** You can inject multiple dependencies by listing them within the `@inject` decorator.

**Example with Multiple Dependencies:**

```javascript
import { CustomerService } from 'backend/customer-service';
import { CommonDialogs } from 'resources/dialogs/common-dialogs';
import { EventAggregator } from 'aurelia-event-aggregator';
import { inject } from 'aurelia-framework';

@inject(CustomerService, CommonDialogs, EventAggregator)
export class CustomerEditScreen {
  constructor(customerService, dialogs, ea) {
    this.customerService = customerService;
    this.dialogs = dialogs;
    this.ea = ea;
    this.customer = null;
  }

  activate(params) {
    return this.customerService.getCustomerById(params.customerId)
      .then(customer => this.customer = customer)
      .then(customer => this.ea.publish('edit', customer));
  }
}
```

## Using `@autoinject` with TypeScript

TypeScript offers an experimental feature to provide type metadata to Aurelia's DI container automatically. By enabling `"emitDecoratorMetadata": true` in your `tsconfig.json`, you can use the `@autoinject` decorator to eliminate the need to specify dependencies manually.

**Example:**

```typescript
import { CustomerService } from 'backend/customer-service';
import { CommonDialogs } from 'resources/dialogs/common-dialogs';
import { EventAggregator } from 'aurelia-event-aggregator';
import { autoinject } from 'aurelia-framework';

@autoinject
export class CustomerEditScreen {
  constructor(
    private customerService: CustomerService,
    private dialogs: CommonDialogs,
    private ea: EventAggregator
  ) {
    this.customer = null;
  }

  activate(params) {
    return this.customerService.getCustomerById(params.customerId)
      .then(customer => this.customer = customer)
      .then(customer => this.ea.publish('edit', customer));
  }
}
```

{% hint style="info" %}
Interestingly, you don't need to use Aurelia's `@autoinject` decorator at all to achieve this. Adding any decorator will trigger TypeScript to emit type metadata. However, using `@autoinject` enhances code clarity and consistency.
{% endhint %}

## Providing Inject Metadata without Decorators

If you're not using Babel's or TypeScript's decorator support or prefer not to, you can provide inject metadata using a static `inject` property or method on your class.

**Example:**

```javascript
import { CustomerService } from 'backend/customer-service';
import { CommonDialogs } from 'resources/dialogs/common-dialogs';
import { EventAggregator } from 'aurelia-event-aggregator';

export class CustomerEditScreen {
  static inject = [CustomerService, CommonDialogs, EventAggregator];

  constructor(customerService, dialogs, ea) {
    this.customerService = customerService;
    this.dialogs = dialogs;
    this.ea = ea;
    this.customer = null;
  }

  activate(params) {
    return this.customerService.getCustomerById(params.customerId)
      .then(customer => this.customer = customer)
      .then(customer => this.ea.publish('edit:begin', customer));
  }
}
```

**Key Points:**

* **Static `inject` Property:** Assign an array of dependencies to the static `inject` property of the class.
* **Order Matters:** Ensure that the order of dependencies in the `inject` array matches the constructor parameters.
* **No Decorators Needed:** This method provides flexibility for environments where decorators are not preferred or supported.

## Recursive Dependency Resolution

Aurelia's DI container resolves dependencies recursively. If class A depends on class B, and class B depends on classes C and D, the container will resolve all dependencies in the hierarchy automatically.

**Example:**

```javascript
@inject(ServiceB)
export class ServiceA {
  constructor(serviceB) {
    this.serviceB = serviceB;
  }
}

@inject(ServiceC, ServiceD)
export class ServiceB {
  constructor(serviceC, serviceD) {
    this.serviceC = serviceC;
    this.serviceD = serviceD;
  }
}

@inject()
export class ServiceC {
  // ...
}

@inject()
export class ServiceD {
  // ...
}
```

When `ServiceA` is instantiated, Aurelia automatically resolves `ServiceB`, which in turn resolves `ServiceC` and `ServiceD`.
