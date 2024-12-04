# Validation

### Introduction

Aurelia's validation plugin provides a robust and flexible form and data validation system. It offers a fluent, declarative API that makes it easy to define validation rules for your application's models and forms.

The validation system supports:

* Declarative rule definition
* Customizable validation triggers
* Flexible error rendering
* Extensive rule types
* Internationalization support

#### Key Features

* Fluent rule configuration
* Multiple validation triggers
* Custom validation rules
* Easy error display
* Seamless integration with Aurelia's binding system

### Installation and Setup

#### Installation

Install the validation plugin using npm or your chosen package installer:

```bash
npm install aurelia-validation
```

#### Plugin Configuration

In your application's main configuration file (typically `main.js` or `main.ts`), register the validation plugin:

```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin(PLATFORM.moduleName('aurelia-validation'));
}
```

#### Optional Configuration

You can customize the default validation trigger during plugin registration:

```javascript
export function configure(aurelia) {
  aurelia.use.plugin('aurelia-validation', config => {
    config.defaultValidationTrigger(validateTrigger.manual);
  });
}
```

### Validation Basics

#### Defining Rules

Aurelia's validation uses a fluent API to define validation rules. The primary class for defining rules is `ValidationRules`.

**Basic Rule Definition**

```javascript
import { ValidationRules } from 'aurelia-validation';

ValidationRules
  .ensure('firstName')
    .required()
  .ensure('email')
    .required()
    .email()
  .on(MyClass);
```

#### Rule Types

Aurelia provides several built-in validation rules:

| Rule                | Description                                                | Example             |
| ------------------- | ---------------------------------------------------------- | ------------------- |
| `required()`        | Ensures the property is not null, undefined, or whitespace | `.required()`       |
| `email()`           | Validates email format                                     | `.email()`          |
| `minLength(length)` | Validates minimum string length                            | `.minLength(3)`     |
| `maxLength(length)` | Validates maximum string length                            | `.maxLength(50)`    |
| `matches(regex)`    | Validates against a regular expression                     | `.matches(/^\d+$/)` |
| `min(value)`        | Validates minimum numeric value                            | `.min(0)`           |
| `max(value)`        | Validates maximum numeric value                            | `.max(100)`         |

**Custom Display Names**

You can set custom display names for more meaningful error messages:

```javascript
ValidationRules
  .ensure('ssn').displayName('Social Security Number')
    .required()
    .matches(/\d{3}-\d{2}-\d{4}/)
  .on(Person);
```

#### Rule Sequencing

Rules are evaluated in parallel by default. Use `.then()` to create sequential rule evaluation:

```javascript
ValidationRules
  .ensure('email')
    .email()
    .required()
    .then()
    .satisfiesRule('emailNotInUse')
  .on(User);
```

**Customizing Error Messages**

Override default messages using `.withMessage()`:

```javascript
ValidationRules
  .ensure('password')
    .required().withMessage('Please enter a password')
    .minLength(8).withMessage('Password must be at least 8 characters long')
  .on(User);
```

#### Example: Complete Validation Setup

```javascript
export class RegistrationForm {
  firstName = '';
  email = '';
  password = '';

  constructor() {
    ValidationRules
      .ensure('firstName')
        .required().withMessage('First name is required')
        .minLength(2).withMessage('First name must be at least 2 characters')
      .ensure('email')
        .required().withMessage('Email is required')
        .email().withMessage('Please enter a valid email address')
      .ensure('password')
        .required().withMessage('Password is required')
        .minLength(8).withMessage('Password must be at least 8 characters')
      .on(this);
  }
}
```

### Validation Controllers

The ValidationController orchestrates validation execution and error rendering in your application. Controllers manage the validation lifecycle and maintain the current validation state.

#### Creating a Controller

You can create a controller using dependency injection:

```javascript
import { inject, NewInstance } from 'aurelia-dependency-injection';
import { ValidationController } from 'aurelia-validation';

@inject(NewInstance.of(ValidationController))
export class RegistrationForm {
  controller = null;

  constructor(controller) {
    this.controller = controller;
  }
}
```

Alternatively, use the ValidationControllerFactory:

```javascript
import { ValidationControllerFactory } from 'aurelia-validation';

@inject(ValidationControllerFactory)
export class RegistrationForm {
  controller = null;

  constructor(controllerFactory) {
    this.controller = controllerFactory.createForCurrentScope();
  }
}
```

#### Validation Triggers

Controllers support different validation triggers that determine when validation occurs:

* `blur`: Validates when an input loses focus (default)
* `focusout`: Validates when an element or its children lose focus
* `change`: Validates when the model value changes
* `changeOrBlur`: Validates on both change and blur events
* `changeOrFocusout`: Validates on both change and focusout events
* `manual`: Validation only occurs when explicitly called

Set the trigger in your view-model:

```javascript
import { validateTrigger } from 'aurelia-validation';

export class RegistrationForm {
  constructor(controller) {
    this.controller = controller;
    this.controller.validateTrigger = validateTrigger.changeOrBlur;
  }
}
```

#### Validate and Reset Methods

Trigger validation programmatically using the controller's methods:

```javascript
class RegistrationForm {
  async submit() {
    const result = await this.controller.validate();
    if (result.valid) {
      // proceed with form submission
    }
  }

  reset() {
    this.controller.reset();
  }
}
```

You can validate specific objects or properties:

```javascript
// Validate specific object
controller.validate({ object: person });

// Validate specific property
controller.validate({ object: person, propertyName: 'email' });
```

### Displaying Validation Errors

#### Built-in Error Display

The simplest way to display validation errors is using the `validation-errors` attribute and controller's errors property:

```html
<form>
  <div validation-errors.bind="errors">
    <input value.bind="email & validate">
    
    <ul if.bind="errors.length">
      <li repeat.for="error of errors">
        ${error.error.message}
      </li>
    </ul>
  </div>
</form>
```

#### Bootstrap Form Integration

Here's an example using Bootstrap form styling:

```html
<form>
  <div class="form-group" validation-errors.bind="firstNameErrors"
       class.bind="firstNameErrors.length ? 'has-error' : ''">
    <label for="firstName">First Name</label>
    <input type="text" class="form-control" id="firstName"
           value.bind="firstName & validate">
    <span class="help-block" repeat.for="error of firstNameErrors">
      ${error.error.message}
    </span>
  </div>
</form>
```

#### Custom Renderer Implementation

Create custom renderers for more control over error display:

```javascript
import { ValidationRenderer, RenderInstruction } from 'aurelia-validation';

export class CustomFormRenderer {
  render(instruction: RenderInstruction) {
    for (let { result, elements } of instruction.unrender) {
      for (let element of elements) {
        this.removeError(element, result);
      }
    }

    for (let { result, elements } of instruction.render) {
      for (let element of elements) {
        this.addError(element, result);
      }
    }
  }

  addError(element, result) {
    if (result.valid) return;

    const formGroup = element.closest('.form-group');
    if (!formGroup) return;

    // Add error class
    formGroup.classList.add('has-error');

    // Add error message
    const message = document.createElement('div');
    message.className = 'validation-error';
    message.textContent = result.message;
    message.id = `validation-message-${result.id}`;
    formGroup.appendChild(message);
  }

  removeError(element, result) {
    if (result.valid) return;

    const formGroup = element.closest('.form-group');
    if (!formGroup) return;

    // Remove error class
    formGroup.classList.remove('has-error');

    // Remove error message
    const message = formGroup.querySelector(`#validation-message-${result.id}`);
    if (message) {
      formGroup.removeChild(message);
    }
  }
}
```

Register your custom renderer with the controller:

```javascript
export class RegistrationForm {
  constructor(controller) {
    this.controller = controller;
    this.renderer = new CustomFormRenderer();
    this.controller.addRenderer(this.renderer);
  }

  detached() {
    this.controller.removeRenderer(this.renderer);
  }
}
```

#### Renderer Binding Context

When using validation-errors, you can specify both the errors and the controller:

```html
<div validation-errors="errors.bind: myErrors; controller.bind: myController">
  <input value.bind="email & validate:myController">
  <span repeat.for="error of myErrors">
    ${error.error.message}
  </span>
</div>
```

{% hint style="info" %}
Always remove custom renderers in the detached lifecycle method to prevent memory leaks and ensure proper cleanup.
{% endhint %}

### Advanced Validation Techniques

#### Conditional Validation

Implement conditional validation using the `.when()` method:

```javascript
ValidationRules
  .ensure('alternateEmail')
    .email()
    .required()
      .when(person => person.needsAlternateContact)
    .withMessage('Alternate email is required when alternate contact is enabled')
  .on(Person);
```

#### Custom Rules

Create reusable custom validation rules using `ValidationRules.customRule()`:

```javascript
// Define a custom date validation rule
ValidationRules.customRule(
  'date',
  (value, obj) => value === null || value === undefined || value instanceof Date,
  '${$displayName} must be a valid date.'
);

// Define a custom rule with parameters
ValidationRules.customRule(
  'numericRange',
  (value, obj, min, max) => {
    if (value === null || value === undefined) return true;
    return Number.isInteger(value) && value >= min && value <= max;
  },
  '${$displayName} must be between ${$config.min} and ${$config.max}.',
  (min, max) => ({ min, max })
);

// Using custom rules
ValidationRules
  .ensure('birthDate')
    .required()
    .satisfiesRule('date')
  .ensure('age')
    .satisfiesRule('numericRange', 18, 100)
  .on(Person);
```

#### Entity Validation

Validate entire objects using controller's `addObject()` method:

```javascript
export class OrderForm {
  constructor(controller) {
    this.controller = controller;
    this.order = new Order();
    
    // Add object for validation
    this.controller.addObject(this.order);
  }

  // Validate entire object
  async submit() {
    const result = await this.controller.validate();
    if (result.valid) {
      await this.submitOrder();
    }
  }

  detached() {
    // Clean up
    this.controller.removeObject(this.order);
  }
}
```

Define object-level rules using `ensureObject()`:

```javascript
ValidationRules
  .ensureObject()
    .satisfies(obj => obj.endDate > obj.startDate)
    .withMessage('End date must be after start date')
  .on(DateRange);
```

### Integration

#### Working with I18N

Integrate validation messages with aurelia-i18n:

```javascript
import { I18N } from 'aurelia-i18n';
import { ValidationMessageProvider } from 'aurelia-validation';

// Configure during app startup
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin('aurelia-i18n')
    .plugin('aurelia-validation');

  // Override message provider
  ValidationMessageProvider.prototype.getMessage = function(key) {
    const i18n = aurelia.container.get(I18N);
    const translation = i18n.tr(`validationMessages.${key}`);
    return this.parser.parse(translation);
  };

  ValidationMessageProvider.prototype.getDisplayName = function(propertyName, displayName) {
    if (displayName) return displayName;
    const i18n = aurelia.container.get(I18N);
    return i18n.tr(propertyName);
  };
}
```

Translation files example:

```json
{
  "validationMessages": {
    "required": "${$displayName} is required",
    "email": "${$displayName} must be a valid email",
    "minLength": "${$displayName} must be at least ${$config.length} characters"
  },
  "propertyNames": {
    "firstName": "First Name",
    "email": "Email Address"
  }
}
```

### API Reference

#### ValidationRules

Core methods for defining validation rules:

| Method                 | Description                              |
| ---------------------- | ---------------------------------------- |
| `ensure(property)`     | Targets a property for validation        |
| `ensureObject()`       | Targets the entire object for validation |
| `required()`           | Validates presence                       |
| `email()`              | Validates email format                   |
| `minLength(length)`    | Validates minimum length                 |
| `maxLength(length)`    | Validates maximum length                 |
| `matches(regex)`       | Validates against regular expression     |
| `satisfies(condition)` | Custom validation function               |
| `satisfiesRule(name)`  | Uses a custom rule                       |
| `when(condition)`      | Conditional validation                   |
| `withMessage(message)` | Custom error message                     |
| `on(target)`           | Applies rules to class/object            |

#### ValidationController

Core controller methods:

| Method                                     | Description                               |
| ------------------------------------------ | ----------------------------------------- |
| `validate()`                               | Validates all registered objects/bindings |
| `reset()`                                  | Clears all validation results             |
| `addObject(object, rules?)`                | Registers object for validation           |
| `removeObject(object)`                     | Removes object from validation            |
| `addRenderer(renderer)`                    | Adds custom error renderer                |
| `removeRenderer(renderer)`                 | Removes error renderer                    |
| `addError(message, object, propertyName?)` | Adds manual error                         |
| `removeError(result)`                      | Removes specific error                    |

#### ValidateResult

Properties available in validation results:

| Property       | Type    | Description                    |
| -------------- | ------- | ------------------------------ |
| `valid`        | boolean | Validation result              |
| `message`      | string  | Error message                  |
| `object`       | any     | Validated object               |
| `propertyName` | string  | Validated property             |
| `rule`         | any     | Rule that generated the result |

{% hint style="info" %}
When implementing custom validation rules, always handle null/undefined values appropriately to maintain consistent validation behavior.
{% endhint %}

{% hint style="warning" %}
Be careful when using async custom rules, which may impact form submission timing and user experience.
{% endhint %}

