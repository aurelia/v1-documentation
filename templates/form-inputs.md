# Form inputs

Aurelia provides a powerful and intuitive approach to handling form inputs and user interactions. At the core of Aurelia's form handling is two-way binding, which creates a seamless connection between the view (HTML) and the view model (JavaScript).

### Key Characteristics of Aurelia Form Inputs

* **Automatic Two-Way Binding**: By default, form elements automatically sync changes between the view and view model
* **Reactive Data Flow**: Input changes are immediately reflected in the view model
* **Minimal Boilerplate**: Requires less code compared to traditional form handling approaches

### Data Flow Mechanism

When a user interacts with a form input, the following process occurs:

1. User enters/modifies input value
2. Native form input events are triggered
3. Aurelia's binding system detects the change
4. View model is automatically updated
5. Any dependent properties or computations are instantly refreshed

### Basic Form Input Binding

#### Text Inputs

```html
<template>
  <input type="text" value.bind="firstName">
  <p>Hello, ${firstName}</p>
</template>
```

```javascript
export class MyViewModel {
  firstName = '';
}
```

**Input Binding Variations**

| Binding Mode     | Syntax           | Description                               |
| ---------------- | ---------------- | ----------------------------------------- |
| Two-Way Binding  | `value.bind`     | Default, syncs changes in both directions |
| One-Way Binding  | `value.one-way`  | Updates view model, but not vice versa    |
| One-Time Binding | `value.one-time` | Initial value only                        |

#### Textarea

```html
<textarea value.bind="description"></textarea>
```

#### Checkbox Inputs

```html
<template>
  <input type="checkbox" checked.bind="isSubscribed">
  <label>Subscribe to newsletter</label>
</template>
```

```javascript
export class MyViewModel {
  isSubscribed = false;
}
```

#### Radio Buttons

```html
<template>
  <input type="radio" value="option1" model.bind="selectedOption">
  <input type="radio" value="option2" model.bind="selectedOption">
</template>
```

```javascript
export class MyViewModel {
  selectedOption = '';
}
```

#### Select Dropdowns

```html
<template>
  <select value.bind="selectedCountry">
    <option repeat.for="country of countries" 
            model.bind="country.code">
      ${country.name}
    </option>
  </select>
</template>
```

```javascript
export class MyViewModel {
  countries = [
    { name: 'United States', code: 'US' },
    { name: 'Canada', code: 'CA' }
  ];
  selectedCountry = '';
}
```

#### Input Binding Considerations

> **Note**: When using `model.bind`, ensure you're binding to the entire object, not just a primitive value. This allows for more complex data binding scenarios.

> **Tip**: Always initialize form-related properties in your view model to prevent undefined errors and provide default states.

### Advanced Input Techniques

#### Custom Validation

Aurelia provides flexible validation through the `aurelia-validation` plugin. Here's a comprehensive approach to form validation:

```javascript
import { inject } from 'aurelia-framework';
import { ValidationController, ValidationRules } from 'aurelia-validation';

@inject(ValidationController)
export class RegistrationViewModel {
  email = '';
  
  constructor(validationController) {
    this.validationController = validationController;

    ValidationRules
      .ensure('email')
        .required()
        .email()
      .on(this);
  }
}
```

```html
<template>
  <form>
    <input 
      type="text" 
      value.bind="email" 
      error.bind="emailErrors">
    <div if.bind="emailErrors" class="error">
      ${emailErrors}
    </div>
  </form>
</template>
```

#### Input Formatting

**Number Formatting**

```html
<template>
  <input 
    type="text" 
    value.bind="price" 
    matcher.bind="currencyFormatter">
</template>
```

```javascript
export class ViewModel {
  price = 0;
  
  currencyFormatter = {
    toView(value) {
      return value 
        ? `$${value.toFixed(2)}` 
        : '$0.00';
    },
    fromView(value) {
      return parseFloat(
        value.replace('$', '').replace(',', '')
      );
    }
  };
}
```

#### Conditional Binding

```html
<template>
  <input 
    type="text" 
    value.bind="username"
    disabled.bind="isUsernameLocked">
  
  <select 
    value.bind="accountType"
    show.bind="canSelectAccountType">
    <option>Basic</option>
    <option>Premium</option>
  </select>
</template>
```

```javascript
import { computedFrom } from 'aurelia-framework';

export class UserViewModel {
  username = '';
  isUsernameLocked = false;
  accountType = 'Basic';
  
  @computedFrom('username')
  get canSelectAccountType() {
    return this.username.length > 5;
  }
}
```

#### Computed Properties with Inputs

```javascript
import { computedFrom } from 'aurelia-framework';

export class CalculatorViewModel {
  firstNumber = 0;
  secondNumber = 0;
  
  // Computed property automatically updates
  @computedFrom('firstNumber', 'secondNumber')
  get sum() {
    return this.firstNumber + this.secondNumber;
  }
  
  @computedFrom('firstNumber', 'secondNumber')
  get average() {
    return (this.firstNumber + this.secondNumber) / 2;
  }
}
```

```html
<template>
  <input type="number" value.bind="firstNumber">
  <input type="number" value.bind="secondNumber">
  
  <p>Sum: ${sum}</p>
  <p>Average: ${average}</p>
</template>
```

#### Validation and Error Handling

| Technique         | Description               | Use Case               |
| ----------------- | ------------------------- | ---------------------- |
| Required Fields   | Ensure input is not empty | Form completeness      |
| Email Validation  | Check email format        | User registration      |
| Number Range      | Validate numeric inputs   | Age, quantity limits   |
| Custom Validators | Complex validation logic  | Specialized form rules |

{% hint style="warning" %}
Always validate both client-side and server-side to ensure data integrity.
{% endhint %}

{% hint style="info" %}
Use meaningful error messages that guide users to correct their input.
{% endhint %}

### Form Submission

Form submission in Aurelia involves managing user input, validation, and processing data with a clean, reactive approach. The framework provides multiple strategies for handling form submissions efficiently.

#### Basic Form Submission

```javascript
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@inject(HttpClient)
export class RegistrationViewModel {
  username = '';
  email = '';
  password = '';

  constructor(http) {
    this.http = http;
  }

  submitForm() {
    this.http.fetch('/api/register', {
      method: 'post',
      body: JSON.stringify({
        username: this.username,
        email: this.email,
        password: this.password
      })
    })
    .then(response => response.json())
    .then(data => {
      // Handle successful registration
    })
    .catch(error => {
      // Handle errors
    });
  }
}
```

```html
<template>
  <form submit.trigger="submitForm()">
    <input type="text" value.bind="username">
    <input type="email" value.bind="email">
    <input type="password" value.bind="password">
    <button type="submit">Register</button>
  </form>
</template>
```

#### Comprehensive Form Validation

```javascript
import {inject} from 'aurelia-framework';
import {ValidationController, ValidationRules} from 'aurelia-validation';
import {computedFrom} from 'aurelia-framework';

@inject(ValidationController)
export class AdvancedFormViewModel {
  username = '';
  email = '';
  age = null;

  constructor(validationController) {
    this.validationController = validationController;

    // Define validation rules
    ValidationRules
      .ensure('username')
        .required()
        .minLength(3)
      .ensure('email')
        .required()
        .email()
      .ensure('age')
        .required()
        .between(18, 99)
      .on(this);
  }

  @computedFrom('username', 'email', 'age')
  get isFormValid() {
    return this.username && 
           this.email && 
           this.age !== null;
  }

  submitForm() {
    this.validationController.validate()
      .then(result => {
        if (result.valid) {
          // Process form submission
          this.processRegistration();
        }
      });
  }

  processRegistration() {
    // Submission logic
  }
}
```

#### Preventing Default Form Submission

```html
<template>
  <form submit.trigger="handleSubmit($event)">
    <!-- form fields -->
  </form>
</template>
```

```javascript
export class FormViewModel {
  handleSubmit(event) {
    // Prevent default form submission
    event.preventDefault();

    // Custom submission logic
    this.processForm();
  }

  processForm() {
    // Form processing
  }
}
```

#### Handling Form Errors

```javascript
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@inject(HttpClient)
export class ErrorHandlingViewModel {
  formErrors = [];

  constructor(http) {
    this.http = http;
  }

  submitForm() {
    // Reset previous errors
    this.formErrors = [];

    this.http.fetch('/api/submit', {
      method: 'post',
      body: JSON.stringify(this.formData)
    })
    .then(response => response.json())
    .then(result => {
      // Handle successful submission
    })
    .catch(error => {
      // Populate form errors
      if (error.details) {
        this.formErrors = error.details.map(err => err.message);
      }
    });
  }
}
```

{% hint style="info" %}
Always prevent default form submission and handle it programmatically to ensure full control over the process.
{% endhint %}

### Working with Complex Forms

#### Complex Object Binding

```javascript
import {inject} from 'aurelia-framework';
import {computedFrom} from 'aurelia-framework';

export class ComplexFormViewModel {
  user = {
    personal: {
      firstName: '',
      lastName: '',
      age: null
    },
    contact: {
      email: '',
      phone: ''
    },
    preferences: {
      newsletter: false,
      interests: []
    }
  };

  interestOptions = [
    'Technology', 
    'Sports', 
    'Music', 
    'Travel'
  ];

  @computedFrom('user.personal.firstName', 'user.personal.lastName')
  get fullName() {
    return `${this.user.personal.firstName} ${this.user.personal.lastName}`;
  }

  addInterest(interest) {
    if (!this.user.preferences.interests.includes(interest)) {
      this.user.preferences.interests.push(interest);
    }
  }

  removeInterest(interest) {
    const index = this.user.preferences.interests.indexOf(interest);
    if (index > -1) {
      this.user.preferences.interests.splice(index, 1);
    }
  }
}
```

```html
<template>
  <form>
    <!-- Personal Information -->
    <section>
      <input type="text" value.bind="user.personal.firstName" placeholder="First Name">
      <input type="text" value.bind="user.personal.lastName" placeholder="Last Name">
      <input type="number" value.bind="user.personal.age" placeholder="Age">
    </section>

    <!-- Contact Information -->
    <section>
      <input type="email" value.bind="user.contact.email" placeholder="Email">
      <input type="tel" value.bind="user.contact.phone" placeholder="Phone">
    </section>

    <!-- Preferences -->
    <section>
      <label>
        <input type="checkbox" checked.bind="user.preferences.newsletter">
        Subscribe to Newsletter
      </label>

      <h3>Interests</h3>
      <div>
        <select value.bind="selectedInterest">
          <option repeat.for="interest of interestOptions" model.bind="interest">
            ${interest}
          </option>
        </select>
        <button click.delegate="addInterest(selectedInterest)">Add Interest</button>
      </div>

      <ul>
        <li repeat.for="interest of user.preferences.interests">
          ${interest}
          <button click.delegate="removeInterest(interest)">Remove</button>
        </li>
      </ul>
    </section>
  </form>
</template>
```

#### Nested Form Structures

```javascript
import {inject} from 'aurelia-framework';
import {ValidationController, ValidationRules} from 'aurelia-validation';

@inject(ValidationController)
export class NestedFormViewModel {
  addresses = [{
    type: 'home',
    street: '',
    city: '',
    country: ''
  }];

  constructor(validationController) {
    this.validationController = validationController;

    // Validation for nested structures
    ValidationRules
      .ensure('addresses[0].street').required()
      .ensure('addresses[0].city').required()
      .on(this);
  }

  addAddress() {
    this.addresses.push({
      type: 'additional',
      street: '',
      city: '',
      country: ''
    });
  }

  removeAddress(index) {
    if (this.addresses.length > 1) {
      this.addresses.splice(index, 1);
    }
  }
}
```

#### Handling Arrays of Form Data

```javascript
export class OrderFormViewModel {
  orderItems = [];

  addOrderItem() {
    this.orderItems.push({
      product: '',
      quantity: 1,
      price: 0
    });
  }

  removeOrderItem(index) {
    this.orderItems.splice(index, 1);
  }

  @computedFrom('orderItems')
  get totalOrderValue() {
    return this.orderItems.reduce((total, item) => 
      total + (item.quantity * item.price), 0);
  }
}
```

### Practical Examples

#### User Registration Form

```javascript
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';
import {ValidationController, ValidationRules} from 'aurelia-validation';

@inject(HttpClient, ValidationController)
export class RegistrationViewModel {
  registration = {
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    agreeTOS: false
  };

  constructor(http, validationController) {
    this.http = http;
    this.validationController = validationController;

    // Complex validation rules
    ValidationRules
      .ensure('username')
        .required()
        .minLength(3)
      .ensure('email')
        .required()
        .email()
      .ensure('password')
        .required()
        .minLength(8)
      .ensure('confirmPassword')
        .equals('password', 'Passwords must match')
      .ensure('agreeTOS')
        .equals(true, 'You must agree to Terms of Service')
      .on(this.registration);
  }

  register() {
    this.validationController.validate()
      .then(result => {
        if (result.valid) {
          this.http.fetch('/api/register', {
            method: 'POST',
            body: JSON.stringify(this.registration)
          });
        }
      });
  }
}
```

{% hint style="info" %}
Break complex forms into logical sections for improved maintainability.
{% endhint %}

{% hint style="warning" %}
Always sanitize and validate user input, both client-side and server-side.
{% endhint %}

