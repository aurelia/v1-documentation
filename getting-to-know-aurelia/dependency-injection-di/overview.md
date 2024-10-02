---
description: >-
  Aurelia's Dependency Injection (DI) system is a powerful feature that promotes
  modularity, testability, and maintainability in your applications. This
  documentation provides a comprehensive guide to u
---

# Overview

Dependency Injection is a design pattern that allows a class to receive its dependencies from an external source rather than creating them internally. This approach fosters loose coupling between classes, making your codebase more modular and easier to test.

## What is Dependency Injection?

In an object-oriented world, complex systems are often broken down into smaller, manageable objects, each handling a specific concern. Dependency Injection simplifies assembling these objects by managing their creation and providing them with the necessary dependencies at runtime.

### Aurelia's DI Container

Aurelia's DI container is at the heart of its DI system. It manages the registration, resolution, and lifetimes of dependencies, ensuring that each component receives the correct instances it requires to function.

### Object Lifetime and Container Hierarchies

Understanding object lifetimes and container hierarchies is crucial for effective dependency management in Aurelia.

**Lifetime Behaviors**

Aurelia supports three primary lifetime behaviors:

* **Container Singleton**
  * **Description:** A singleton instance is created the first time it's requested within a specific container and reused for all subsequent requests in that container.
  * **Behavior:** The DI container maintains a reference to the singleton, preventing it from being garbage collected until the container itself is disposed.
  *   **Example:**

      ```javascript
      container.registerSingleton(HttpClient);
      ```
* **Application Singleton**
  * **Description:** This is similar to a container singleton but scoped to the root container, making it accessible across all child containers.
  * **Behavior:** Unless explicitly overridden, all child containers inherit the singleton instance from the root.
  *   **Example:**

      ```javascript
      container.registerSingleton(AppService);
      ```
* **Transient**
  * **Description:** A new instance is created every time the dependency is requested.
  * **Behavior:** No references are held by the container, allowing instances to be garbage collected when no longer in use.
  *   **Example:**

      ```javascript
      container.registerTransient(Logger);
      ```

**Container Hierarchies**

Aurelia allows the creation of child containers from parent containers. Child containers inherit registrations from their parents but can also override them. This hierarchy enables scoped dependency management, especially useful in scenarios like routed components and custom elements.

**Example:**

```javascript
// Creating a child container from the root
const childContainer = rootContainer.createChild();

// Overriding a registration in the child container
childContainer.registerSingleton(ServiceA, CustomServiceA);

// Resolving ServiceA from the child container returns CustomServiceA
const serviceA = childContainer.get(ServiceA);
```

### How Aurelia Utilizes DI Containers

Aurelia leverages DI containers extensively to manage the lifecycle and dependencies of various components within an application.

**Custom Elements and Custom Attributes**

* **Child Container Creation:** For each custom element or attribute, Aurelia creates a child container scoped to that element or attribute.
* **Singleton Registration:** These are registered as singletons within their respective child containers.
* **Dependency Scope:** Dependencies are isolated and scoped appropriately within the DOM hierarchy.

{% hint style="info" %}
Aurelia only creates child containers for custom elements or elements with custom attributes. Plain HTML elements or elements with only binding expressions do not trigger child container creation.&#x20;
{% endhint %}

**Routed Components**

* **Child Container Creation:** Each navigation event via Aurelia's Router results in a child container.
* **View-Model Registration:** The view-model associated with the route is auto-registered within this child container.
* **Singleton Behavior:** By default, view-models are singletons within their container unless explicitly configured otherwise.

**Dynamic Components**

* **Child Container Creation:** Dynamic composition using the `<compose>` element or the `CompositionEngine` creates child containers.
* **Auto-Registration:** Components are auto-registered within their respective child containers, following the singleton behavior.
