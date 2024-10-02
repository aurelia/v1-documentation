# Resolvers

Resolvers in Aurelia's DI system provide advanced control over instantiating and injecting dependencies. They allow you to modify the resolution behavior for specific dependencies, enabling patterns like lazy loading, optional dependencies, and factory-based instantiation.

## Built-in Resolvers

Aurelia offers several built-in resolvers that can be used to customize dependency injection:

* **`Lazy`**
  * **Purpose:** Injects a function that lazily retrieves the dependency when invoked.
  *   **Usage:**

      ```javascript
      import { inject, Lazy } from 'aurelia-framework';
      import { HttpClient } from 'aurelia-fetch-client';

      @inject(Lazy.of(HttpClient))
      export class CustomerDetail {
        constructor(getHttpClient) {
          this.getHttpClient = getHttpClient;
        }

        fetchData() {
          const httpClient = this.getHttpClient();
          // Use httpClient...
        }
      }
      ```
  * **Explanation:** Instead of injecting an instance of `HttpClient` directly, a function `getHttpClient` is injected. This function can be called to retrieve the `HttpClient` instance when needed.
* **`All`**
  * **Purpose:** Injects an array of all registered instances matching the provided key.
  *   **Usage:**

      ```javascript
      import { inject, All } from 'aurelia-framework';
      import { Plugin } from './plugin';

      @inject(All.of(Plugin))
      export class PluginManager {
        constructor(plugins) {
          this.plugins = plugins;
        }

        initialize() {
          this.plugins.forEach(plugin => plugin.init());
        }
      }
      ```
  * **Explanation:** Injects all instances registered under the `Plugin` key as an array, allowing you to manage multiple plugins collectively.
* **`Optional`**
  * **Purpose:** Injects an instance if it exists; otherwise, injects `null`.
  *   **Usage:**

      ```javascript
      import { inject, Optional } from 'aurelia-framework';
      import { LoggedInUser } from './user-service';

      @inject(Optional.of(LoggedInUser))
      export class UserProfile {
        constructor(user) {
          this.user = user;
        }

        display() {
          if (this.user) {
            // Display user information
          } else {
            // Prompt for login
          }
        }
      }
      ```
  * **Explanation:** The `LoggedInUser` dependency is injected only if it has been previously registered. If not, `null` is injected, allowing for optional usage.
* **`Parent`**
  * **Purpose:** Starts dependency resolution from the parent container instead of the current one.
  *   **Usage:**

      ```javascript
      import { inject, Parent } from 'aurelia-framework';
      import { MyCustomElement } from './my-custom-element';

      @inject(Parent.of(MyCustomElement))
      export class ChildComponent {
        constructor(parentElement) {
          this.parentElement = parentElement;
        }
      }
      ```
  * **Explanation:** Instead of resolving from the current container, the resolver starts from the parent container, allowing access to dependencies defined higher up in the container hierarchy.
* **`Factory`**
  * **Purpose:** Injects a factory function that can create instances of the dependency, allowing for dynamic data passing.
  *   **Usage:**

      ```javascript
      import { inject, Factory } from 'aurelia-framework';
      import { CustomClass } from './custom-class';

      @inject(Factory.of(CustomClass))
      export class Component {
        constructor(createCustomClass) {
          this.createCustomClass = createCustomClass;
        }

        createInstance(data) {
          const instance = this.createCustomClass(data);
          // Use the instance...
        }
      }
      ```
  * **Explanation:** A factory function `createCustomClass` is injected, which can be called with specific data to create instances of `CustomClass` on demand.
* **`NewInstance`**
  * **Purpose:** Injects a new dependency instance each time, ignoring any existing instances in the container.
  *   **Usage:**

      ```javascript
      import { inject, NewInstance } from 'aurelia-framework';
      import { CustomClass } from './custom-class';

      @inject(NewInstance.of(CustomClass))
      export class AnotherComponent {
        constructor(customClassInstance) {
          this.customClassInstance = customClassInstance;
        }
      }
      ```
  * **Explanation:** Each time `AnotherComponent` is instantiated, a new instance of `CustomClass` is created, regardless of any existing instances in the container.

## Using Resolvers with TypeScript

When using TypeScript, the `@autoinject` decorator does not support resolvers directly. Instead, you should use argument decorators to apply resolvers without duplicating the constructor's parameter order.

**Example:**

```typescript
import { inject, NewInstance } from 'aurelia-framework';
import { HttpClient } from 'aurelia-fetch-client';

export class CustomerDetail {
  constructor(@NewInstance.of(HttpClient) private httpClient: HttpClient) {}

  fetchData() {
    // Always uses a new instance of HttpClient
    const client = this.httpClient;
    // Use the client...
  }
}
```

## Custom Resolvers

In addition to built-in resolvers, Aurelia allows you to create custom resolvers to tailor the dependency resolution process to your needs.

**Example:**

```javascript
import { Container, Resolver } from 'aurelia-framework';

class CustomResolver extends Resolver {
  get(container, key) {
    // Custom resolution logic
    return new CustomClass();
  }
}

container.registerResolver('CustomKey', new CustomResolver());
```

**Usage in Injection:**

```javascript
import { inject } from 'aurelia-framework';

@inject('CustomKey')
export class CustomComponent {
  constructor(customInstance) {
    this.customInstance = customInstance;
  }
}
```

## Best Practices

* **Ensure Consistency Between `@inject` and Constructor Parameters:** The dependencies listed in the `@inject` decorator must match the constructor parameters in both type and order.
* **Leverage `@autoinject` When Possible:** Using `@autoinject` with TypeScript can reduce boilerplate and improve clarity by automatically inferring dependencies from constructor parameter types.
* **Use Resolvers Appropriately:** Utilize built-in resolvers like `Lazy`, `All`, and `Optional` to handle specific dependency injection scenarios effectively.
* **Avoid Overusing Singleton Lifetimes:** While singletons are convenient, overusing them can lead to tightly coupled code. Prefer transient lifetimes for dependencies that should have a limited scope.
* **Explicit Configuration for Critical Services:** To prevent unintended behaviors, consider explicitly registering essential services with specific lifetimes.
* **Utilize Child Containers for Scoped Dependencies:** When dealing with components like custom elements or routed views, appropriately leverage child containers to scope dependencies.
* **Maintain Loose Coupling Between Components:** Strive for loose coupling by minimizing direct dependencies between unrelated components, enhancing modularity and testability.
