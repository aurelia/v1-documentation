# Comparison to other state management libraries

The JavaScript ecosystem offers a variety of state management libraries, each with its own strengths and weaknesses. Let's compare the Aurelia Store plugin to some of the most popular alternatives: Redux, MobX, and VueX.

### Differences to Redux

Redux is arguably the most popular state management library, particularly within the React community. While Aurelia Store shares some fundamental principles with Redux, there are key differences:

| Feature          | Redux                                                                     | Aurelia Store                                                                                  |
| ---------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Boilerplate      | Often requires significant boilerplate (actions, reducers, types)         | Less boilerplate; actions are simple functions                                                 |
| Async Operations | Typically requires middleware (e.g., Redux Thunk, Redux Saga)             | Built-in support for async actions and middleware                                              |
| Immutability     | Enforces immutability                                                     | Enforces immutability                                                                          |
| Core Concepts    | Actions, Reducers, Store                                                  | Actions, Store, Middleware                                                                     |
| Framework        | Framework-agnostic                                                        | Specifically designed for Aurelia, integrates well with Aurelia's features (DI, binding, etc.) |
| Learning Curve   | Can be steeper for beginners                                              | Generally considered easier to learn, especially for those familiar with Aurelia               |
| RxJS             | Not a core part; can be integrated with libraries like `redux-observable` | Built on top of RxJS; leverages Observables for state management                               |
| Devtools         | Excellent Redux DevTools support                                          | Excellent Redux DevTools support                                                               |

**In essence:**

* Aurelia Store aims to reduce boilerplate compared to Redux, making it easier to get started and maintain.
* Async operations are handled more seamlessly in Aurelia Store, thanks to its RxJS foundation.
* Aurelia Store is tightly integrated with Aurelia, leveraging its features for a more cohesive development experience.

### Differences to MobX

MobX is another popular state management library that takes a different approach than Redux or Aurelia Store. It relies on observables and automatic tracking of dependencies.

| Feature         | MobX                                                                       | Aurelia Store                                                    |
| --------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Core Principle  | Transparent Functional Reactive Programming (TFRP)                         | Single, immutable state tree with unidirectional data flow       |
| State Changes   | Automatic tracking of observable property changes                          | Explicit state changes through actions                           |
| Immutability    | Does not enforce immutability                                              | Enforces immutability                                            |
| Boilerplate     | Minimal boilerplate                                                        | Some boilerplate (actions), but less than Redux                  |
| Framework       | Framework-agnostic                                                         | Designed for Aurelia                                             |
| Learning Curve  | Generally considered easy to learn                                         | Easy to learn, especially for Aurelia developers                 |
| Computed Values | Built-in support for computed values (similar to Aurelia's `computedFrom`) | Can be achieved using RxJS operators or Aurelia's `computedFrom` |
| Devtools        | MobX DevTools available                                                    | Leverages Redux DevTools                                         |

**Key Differences:**

* MobX automatically tracks changes to observable properties, while Aurelia Store requires explicit state updates through actions.
* MobX does not enforce immutability, which can lead to potential issues if not handled carefully.
* MobX's core principles (TFRP) differ significantly from Aurelia Store's approach.

**Why Choose Aurelia Store over MobX?**

* **Explicit State Changes:** Aurelia Store's explicit state changes through actions make it easier to reason about how and when the state is updated.
* **Immutability:** Aurelia Store's enforcement of immutability helps prevent accidental side-effects and makes debugging easier.
* **Redux DevTools:** Aurelia Store's integration with Redux DevTools provides a powerful debugging experience.
* **Aurelia Integration:** Aurelia Store is designed specifically for Aurelia, providing a more seamless integration with the framework.

### Differences to VueX

VueX is the official state management library for Vue.js, and it shares some similarities with Aurelia Store.

| Feature        | VueX                                                  | Aurelia Store                                                      |
| -------------- | ----------------------------------------------------- | ------------------------------------------------------------------ |
| Core Concepts  | Mutations, Actions, Getters, Modules                  | Actions, Store, Middleware                                         |
| State Changes  | Mutations (synchronous) and Actions (can be async)    | Actions (can be async)                                             |
| Immutability   | Enforces immutability within mutations                | Enforces immutability                                              |
| Framework      | Specifically designed for Vue.js                      | Designed for Aurelia                                               |
| Learning Curve | Easy to learn, especially for Vue.js developers       | Easy to learn, especially for Aurelia developers                   |
| Devtools       | Excellent Vue DevTools support                        | Leverages Redux DevTools                                           |
| Modules        | Built-in support for modules to organize large stores | Can achieve modularity using multiple stores or custom conventions |
| RxJS           | Not a core part                                       | Built on top of RxJS; leverages Observables for state management   |

**Key Differences:**

* **Mutations vs. Actions:** VueX uses mutations for synchronous state changes and actions for asynchronous operations. Aurelia Store uses actions for both.
* **Modules:** VueX has built-in support for modules, while Aurelia Store relies on other mechanisms for organizing large stores.
* **RxJS:** VueX does not use RxJS, while Aurelia Store is built on top of it.

**Why Choose Aurelia Store over VueX?**

* **Aurelia Integration:** Aurelia Store is designed specifically for Aurelia, providing a more seamless integration with the framework's features.
* **RxJS Foundation:** Aurelia Store's use of RxJS provides a powerful and flexible way to manage state and handle asynchronous operations.
* **Redux DevTools:** While Vue DevTools is excellent, some developers might prefer the Redux DevTools ecosystem.
