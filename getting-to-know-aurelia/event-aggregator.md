# Event aggregator

The Event Aggregator pattern in Aurelia allows disparate components and services to communicate through events without requiring direct references to each other. Reducing tight coupling promotes a cleaner and more maintainable codebase.

## Configuration

### Including Event Aggregator in Your Project

To use the Event Aggregator in your Aurelia application, include the `aurelia-event-aggregator` plugin during the configuration phase. You can do this using either `basicConfiguration` or `standardConfiguration`.

* **Basic Configuration**: Provides the essential Event Aggregator functionality.
* **Standard Configuration**: Includes the Event Aggregator along with other commonly used utilities.

{% code title="main.js" %}
```javascript
import { Aurelia } from 'aurelia-framework';
import { standardConfiguration } from 'aurelia-event-aggregator';

export function configure(aurelia: Aurelia) {
  aurelia.use.standardConfiguration();
  aurelia.start().then(() => aurelia.setRoot());
}
```
{% endcode %}

## Accessing the Global Event Aggregator

Once configured, the singleton `EventAggregator` is accessible via the Aurelia global object. This allows you to publish and subscribe to events throughout your application without manually injecting the Event Aggregator each time.

```javascript
// Accessing the global EventAggregator
const eventAggregator = aurelia.container.get(EventAggregator);
```

## Creating Additional Event Aggregator Instances

In addition to the global Event Aggregator, you can create multiple instances as needed. This is useful for scoping events to specific parts of your application.

### Using `includeEventsIn`

To add Event Aggregator capabilities to any object, use the `includeEventsIn` function. This will extend the target object with `publish`and `subscribe` methods.

```javascript
import { includeEventsIn } from 'aurelia-event-aggregator';

const myObject = {};
includeEventsIn(myObject);

// Now `myObject` can publish and subscribe to events
myObject.publish('customChannel', { data: 'payload' });
myObject.subscribe('customChannel', payload => {
  console.log(payload.data);
});
```

## Publishing Events

Publishing events allow components to broadcast messages that other application parts can listen to.

### Publishing to a Channel

You can publish events to named channels. Subscribers listening to these channels will receive the published payload.

```javascript
import { autoinject } from 'aurelia-framework';
import { EventAggregator } from 'aurelia-event-aggregator';

@autoinject
export class APublisher {
  constructor(private eventAggregator: EventAggregator) { }

  publish(): void {
    const payload = { key: 'value' };
    this.eventAggregator.publish('channelName', payload);
  }
}
```

### Publishing a Message Object

Instead of using string channels, you can publish instances of message classes. This approach provides type safety and clarity.

```javascript
// some-message.ts
export class SomeMessage { 
  constructor(public content: string) { }
}

// a-publisher.ts
import { autoinject } from 'aurelia-framework';
import { EventAggregator } from 'aurelia-event-aggregator';
import { SomeMessage } from './some-message';

@autoinject
export class APublisher {
  constructor(private eventAggregator: EventAggregator) { }

  publish(): void {
    const message = new SomeMessage('Hello, World!');
    this.eventAggregator.publish(message);
  }
}
```

## Subscribing to Events

Subscribing to events allows components to react to messages published on specific channels or message types.

### Subscribing to a Channel

Listen for events published to a specific channel by providing the channel name and a callback function.

```javascript
import { autoinject } from 'aurelia-framework';
import { EventAggregator } from 'aurelia-event-aggregator';

@autoinject
export class ASubscriber {
  constructor(private eventAggregator: EventAggregator) { }

  subscribe(): void {
    this.eventAggregator.subscribe('channelName', payload => {
      console.log('Received payload:', payload);
      // Handle the payload as needed
    });
  }
}
```

### Subscribing to a Message Type

Subscribe to specific message types by providing the message class and a callback function. This method leverages TypeScript's type system for enhanced reliability.

```javascript
import { autoinject } from 'aurelia-framework';
import { EventAggregator } from 'aurelia-event-aggregator';
import { SomeMessage } from './some-message';

@autoinject
export class ASubscriber {
  constructor(private eventAggregator: EventAggregator) { }

  subscribe(): void {
    this.eventAggregator.subscribe(SomeMessage, (message: SomeMessage) => {
      console.log('Received message content:', message.content);
      // Handle the message as needed
    });
  }
}
```

## Merging Event Aggregator into Objects

To integrate Event Aggregator functionality into any custom object, use the `includeEventsIn` method. This approach enables objects to publish and subscribe to events without directly interfacing with the Event Aggregator service.

```javascript
import { includeEventsIn } from 'aurelia-event-aggregator';

class MyCustomObject {
  constructor() {
    includeEventsIn(this);
  }

  triggerEvent() {
    this.publish('myCustomChannel', { info: 'Custom event triggered' });
  }

  listenToEvent() {
    this.subscribe('myCustomChannel', payload => {
      console.log('Custom event received:', payload.info);
    });
  }
}

const myObject = new MyCustomObject();
myObject.listenToEvent();
myObject.triggerEvent();
```

## Best Practices

**Cleanup after yourself**: Always unsubscribe from events when they are no longer needed to prevent memory leaks. This is usually done inside of the `detached` lifecycle callback.

```javascript
let subscription = this.eventAggregator.subscribe('channelName', callback);

// To unsubscribe
subscription.dispose();
```

**Use Message Classes for Clarity**: Utilizing message objects instead of string channels can improve code maintainability and type safety.

**Limit Global Event Usage**: While the global Event Aggregator is convenient, consider creating scoped instances for more controlled event handling.
