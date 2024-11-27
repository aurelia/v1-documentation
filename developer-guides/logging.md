# Logging

### Overview

Aurelia provides a flexible logging system that allows developers to implement structured and configurable logging across their applications. The logging mechanism is designed to be lightweight, extensible, and easy to use, supporting various log levels and custom output methods.

### Getting Started

#### Importing Logging Utilities

To begin using logging in Aurelia, you'll need to import the necessary modules:

```javascript
import {LogManager} from 'aurelia-framework';
import {ConsoleAppender} from 'aurelia-logging-console';
```

#### Basic Configuration

To set up logging, you'll typically configure it in your application's main configuration:

```javascript
export function configure(aurelia) {
  // Add a console appender
  LogManager.addAppender(new ConsoleAppender());

  // Optionally set a default log level
  LogManager.setLevel(LogManager.logLevel.debug);

  aurelia.use
    .standardConfiguration()
    .globalResources();

  aurelia.start().then(() => aurelia.setRoot());
}
```

#### Creating a Logger

To create a logger in your application or component:

```javascript
import {LogManager} from 'aurelia-framework';

export class MyComponent {
  constructor() {
    this.logger = LogManager.getLogger(this.constructor.name);
  }

  someMethod() {
    this.logger.debug('Debug message');
    this.logger.info('Information message');
    this.logger.warn('Warning message');
    this.logger.error('Error message');
  }
}
```

### Log Levels

Aurelia supports multiple log levels to control the verbosity of logging:

| Log Level | Description                                                   |
| --------- | ------------------------------------------------------------- |
| `none`    | No logging                                                    |
| `error`   | Critical errors only                                          |
| `warn`    | Warnings and errors                                           |
| `info`    | Informational messages, warnings, and errors                  |
| `debug`   | Detailed debugging information, including all previous levels |

#### Setting Log Levels

You can set the global log level using:

```javascript
// Set to debug level
LogManager.setLevel(LogManager.logLevel.debug);
```

> **Note**: Log levels are hierarchical. Setting a level includes all levels above it.

### Appenders

#### What are Appenders?

Appenders are responsible for outputting log messages. Aurelia provides a default `ConsoleAppender`, but you can create custom appenders.

#### Default ConsoleAppender

The `ConsoleAppender` writes log messages to the browser console:

```javascript
import {ConsoleAppender} from 'aurelia-logging-console';

// Add console appender
LogManager.addAppender(new ConsoleAppender());
```

#### Creating Custom Appenders

You can create a custom appender by implementing the Appender interface:

```javascript
class CustomAppender {
  debug(logger, message, ...rest) {
    // Custom debug logging logic
  }

  info(logger, message, ...rest) {
    // Custom info logging logic
  }

  warn(logger, message, ...rest) {
    // Custom warning logging logic
  }

  error(logger, message, ...rest) {
    // Custom error logging logic
  }
}

// Add the custom appender
LogManager.addAppender(new CustomAppender());
```

### Advanced Usage

#### Multiple Appenders

You can add multiple appenders to route logs to different destinations:

```javascript
LogManager.addAppender(new ConsoleAppender());
LogManager.addAppender(new CustomRemoteLoggerAppender());
```
