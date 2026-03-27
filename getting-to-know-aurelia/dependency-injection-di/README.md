# Dependency injection (DI)

Dependency Injection (DI) is one of Aurelia's core features. It manages the creation and delivery of object dependencies, promoting loose coupling and making your application more modular and testable.

Instead of components creating their own dependencies, Aurelia's DI container provides them automatically. This means your classes declare what they need, and the framework handles the rest.

## How It Works

At its simplest, you use the `@autoinject` decorator (or the static `inject` property) to tell Aurelia what dependencies a class needs:

```javascript
import { autoinject } from 'aurelia-framework';
import { HttpClient } from 'aurelia-fetch-client';
import { EventAggregator } from 'aurelia-event-aggregator';

@autoinject
export class UserService {
  constructor(private http: HttpClient, private ea: EventAggregator) {
  }
}
```

Aurelia automatically creates and injects instances of `HttpClient` and `EventAggregator` when `UserService` is requested.

## Key Concepts

- **Container** — The DI container manages object creation and lifetime. Aurelia creates a root container at startup, and child containers are created for each component.
- **Registration** — How the container knows what to create. By default, classes are registered as transients (new instance each time). You can also register singletons and custom factories.
- **Resolution** — How dependencies are located and provided. The container walks up the container hierarchy until it finds a registration.
- **Resolvers** — Special helpers like `Lazy`, `All`, `Optional`, and `Factory` that give you fine-grained control over how dependencies are resolved.

## In This Section

- [Overview](overview.md) — Deep dive into how the DI container works, object lifetimes, and container hierarchies
- [Injection](injection.md) — All the ways to declare and receive dependencies
- [Resolvers](resolvers.md) — Advanced resolution strategies for special scenarios

{% hint style="info" %}
If you're using TypeScript, `@autoinject` reads type metadata to determine dependencies automatically. For JavaScript, use the static `inject` property or the `@inject()` decorator instead.
{% endhint %}
