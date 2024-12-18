# Store

### Introduction

The Aurelia Store plugin provides a robust and flexible solution for managing state in your Aurelia applications. It is built upon the solid foundation of RxJS, leveraging the power of `Observables` and `BehaviorSubject` to create a reactive and predictable state management system.&#x20;

While you don't need to be an RxJS expert to use the Aurelia Store, having a basic understanding of these concepts can be beneficial. In essence, `Observables` represent a stream of data that can be observed over time, while a `BehaviorSubject` is a special type of `Observable` that holds a current value and emits it to new subscribers.

### Getting started

To begin using the Aurelia Store plugin, you first need to install it via npm:

```bash
npm install aurelia-store
```

**Note:** If you are not already using RxJS in your project, it's recommended to install it as a project dependency:

```bash
npm install rxjs
```

This ensures that you have the latest version of RxJS and that it's properly integrated into your project.
