# Reasons for state management

In modern web development, managing application state effectively is crucial for building complex and maintainable applications. While traditional service-oriented approaches have their place, they can become increasingly complex as your application grows. Let's explore why adopting a state management solution like the Aurelia Store plugin can be beneficial.

### The Problem with Traditional Approaches

In a classic service-oriented architecture, data is often scattered across multiple services, each responsible for a specific part of the application's logic. While this can be simpler initially, especially when combined with a powerful IoC container like Aurelia's, it can lead to several challenges as the application scales:

* **Increased Complexity:** Services can become tightly coupled, with intricate dependencies that are hard to understand and manage.
* **Difficult Change Tracking:** Keeping track of which service modified what data and how to notify all affected components can become a daunting task.
* **Unpredictable State Changes:** It can be challenging to reason about the application's state when multiple services can modify it independently.

### The Single Store Solution

In contrast, the Aurelia Store plugin promotes a single store approach, where all application data is centralized in a single source of truth – the store. The store's content represents the application's state at any given moment, providing a clear snapshot of the entire application's data.

#### Benefits of a Single Store

* **One Source of Truth:** With all data in one place, it's easier to understand the application's current state and how it changes over time.
* **Predictable State Changes:** Modifications to the state are made exclusively through actions, which are the only way to create a new application state. This ensures that state changes are predictable and traceable.
* **Reduced Side-Effects:** By controlling state changes through actions, you can minimize unintended side-effects and make your application more robust.
* **Simplified Communication:** Components don't need to communicate directly with each other to share data. They simply subscribe to the store and react to changes in the relevant parts of the state.

In essence, a single store approach simplifies state management, makes your application more predictable, and reduces the risk of unintended side-effects.

### Why RxJS?

The Aurelia Store plugin is built on top of RxJS, a powerful library for reactive programming. Let's explore why RxJS is a good fit for state management and how it's used within the plugin.

#### `BehaviorSubject` and `Observable`: The Core

At the heart of the Aurelia Store plugin are two key RxJS concepts:

* **`BehaviorSubject`:** The store's internal state is managed by a private `BehaviorSubject` called `_state`. A `BehaviorSubject` is a type of `Observable` that holds a current value and emits it to any new subscribers. It starts with an initial state and emits new states as they are created by actions.
* **`Observable`:** The `BehaviorSubject` is not directly accessible from outside the store. Instead, a public `Observable` called `state` is exposed. This `Observable` provides read-only access to the stream of states, ensuring that consumers can only observe state changes and not directly modify the `BehaviorSubject`'s internal queue.

#### RxJS: "Lodash/Underscore for Events"

RxJS can be thought of as "Lodash/Underscore for events." It provides a rich set of operators and utilities for working with streams of data, allowing you to transform, filter, and combine them in various ways. In the context of the Aurelia Store plugin, these operators can be used to manipulate the state stream, select specific parts of the state, or even test the stream over time.

#### Reactive vs. Imperative

RxJS promotes a reactive programming paradigm. Instead of imperatively telling components to update when something changes (e.g., "a click on button A should trigger a re-rendering on component B"), components observe the state and react to changes automatically. This is achieved through the observer pattern, where components subscribe to the state `Observable` and update themselves when the relevant parts of the state change.

#### Benefits of the Reactive Approach

* **Decoupling:** Components are decoupled from the actions that trigger state changes. They only care about the state itself.
* **Automatic Updates:** Components automatically update when the state changes, eliminating the need for manual updates.
* **Simplified Logic:** The application's logic becomes more declarative and easier to reason about.

#### Asynchronous Nature

Another key benefit of using RxJS is its asynchronous nature. State changes, whether triggered by synchronous actions, asynchronous operations like AJAX requests, or long-running processes like WebSockets, are all handled seamlessly through the `Observable` stream. Components simply subscribe to the stream and react to changes whenever they occur, regardless of the underlying mechanism.
