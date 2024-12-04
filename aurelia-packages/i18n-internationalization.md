# i18n Internationalization

### Overview

Aurelia's internationalization (i18n) features are provided through the `aurelia-i18n` plugin, which is built on top of the widely-used `i18next` library. This combination offers a powerful and flexible solution for adding multilingual support to your Aurelia applications, including translations, number formatting, date formatting, and relative time displays.

### Features

* Text translations with variable substitution
* Number and date formatting based on locale
* Relative time formatting
* Multiple translation namespaces
* HTML content support in translations
* Flexible backend options for loading translations
* TypeScript support
* Value converters and binding behaviors for declarative usage

### Requirements

* Aurelia application
* Node.js and NPM
* Basic understanding of JSON formatting

## Installation

### Using Aurelia CLI

Install the required packages using npm:

```bash
npm install aurelia-i18n i18next
```

For XHR backend support (optional):

```bash
npm install i18next-xhr-backend
```

### Using Webpack

Install the same packages as above:

```bash
npm install aurelia-i18n i18next
```

And optionally:

```bash
npm install i18next-xhr-backend
```

### TypeScript Support

If you're using TypeScript, install the necessary type definitions:

```bash
npm install -D @types/i18next @types/i18next-xhr-backend
```

## Setup and Configuration

### Basic Setup

#### 1. Configure Manual Bootstrapping

Update your `index.html` to use manual bootstrapping:

```html
<body aurelia-app="main">
  <!-- your content -->
</body>
```

#### 2. Project Structure

Create the following folder structure in your project:

```
src/
  ├── locales/
  │   ├── en/
  │   │   └── translation.json
  │   └── de/
  │       └── translation.json
```

#### 3. Translation File Format

Create your translation files using this structure:

```json
{
  "greeting": "Hello!",
  "welcome": {
    "title": "Welcome to our app",
    "message": "Hello {{name}}, how are you?"
  },
  "items": {
    "one": "{{count}} item",
    "other": "{{count}} items"
  }
}
```

### Configuration Options

#### Basic Configuration

Configure the plugin in your `main.js/ts`:

```javascript
import {I18N, TCustomAttribute} from 'aurelia-i18n';
import Backend from 'i18next-xhr-backend';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin('aurelia-i18n', instance => {
      let aliases = ['t', 'i18n'];
      TCustomAttribute.configureAliases(aliases);

      return instance.setup({
        backend: {
          loadPath: './locales/{{lng}}/{{ns}}.json',
        },
        attributes: aliases,
        lng: 'en',
        fallbackLng: 'en',
        debug: false
      });
    });

  aurelia.start().then(a => a.setRoot());
}
```

#### Using Built-in Aurelia Loader

Alternative configuration using Aurelia's built-in loader:

```javascript
import {I18N, Backend, TCustomAttribute} from 'aurelia-i18n';

export function configure(aurelia) {
  aurelia.use
    .plugin('aurelia-i18n', instance => {
      let aliases = ['t', 'i18n'];
      TCustomAttribute.configureAliases(aliases);

      instance.i18next.use(Backend.with(aurelia.loader));

      return instance.setup({
        backend: {
          loadPath: './locales/{{lng}}/{{ns}}.json',
        },
        attributes: aliases,
        lng: 'en',
        fallbackLng: 'en',
        debug: false
      });
    });
}
```

### Multiple Namespaces

You can organize translations into multiple namespaces for better organization:

```javascript
instance.setup({
  backend: {
    loadPath: './locales/{{lng}}/{{ns}}.json',
  },
  ns: ['translation', 'navigation', 'errors'],
  defaultNS: 'translation',
  fallbackLng: 'en',
  debug: false
});
```

This allows you to structure your translations like this:

```
locales/
  ├── en/
  │   ├── translation.json
  │   ├── navigation.json
  │   └── errors.json
  └── de/
      ├── translation.json
      ├── navigation.json
      └── errors.json
```

{% hint style="info" %}
When using namespaces, reference strings outside the default namespace by prefixing them with the namespace, e.g., `navigation:home.title`.
{% endhint %}

{% hint style="warning" %}
Ensure all your translation files are properly formatted JSON and that all namespaces are present in each language folder to avoid runtime errors.
{% endhint %}

## Using Translations

### Setting and Getting Locales

#### Setting the Active Locale

Change the application's language using the `setLocale` method:

```javascript
import {I18N} from 'aurelia-i18n';

export class MyViewModel {
  static inject = [I18N];
  
  constructor(i18n) {
    this.i18n = i18n;
    
    this.i18n.setLocale('de-DE').then(() => {
      // Locale has been loaded and set
    });
  }
}
```

#### Getting the Active Locale

Retrieve the current locale using `getLocale`:

```javascript
constructor(i18n) {
  this.i18n = i18n;
  const currentLocale = this.i18n.getLocale();
}
```

### Translation Methods

#### Translating via Code

Use the `tr` method for programmatic translations:

```javascript
import {I18N} from 'aurelia-i18n';

export class MyViewModel {
  static inject = [I18N];
  
  constructor(i18n) {
    this.i18n = i18n;
    
    // Simple translation
    let translated = this.i18n.tr('myKey');
    
    // With parameters
    let withParams = this.i18n.tr('welcome.message', { name: 'John' });
  }
}
```

#### Translating via HTML Attributes

Use the `t` attribute in your templates:

```html
<!-- Simple translation -->
<span t="title">Title</span>

<!-- With HTML content -->
<div t="[html]content">Content</div>

<!-- Multiple attributes -->
<input t="[placeholder,title]input.placeholder">

<!-- Different keys for different attributes -->
<span t="[text]title;[class]titleClass">Title</span>
```

Available special attributes:

* `[text]`: Sets textContent (default)
* `[html]`: Sets innerHTML
* `[append]`: Appends translation to existing content
* `[prepend]`: Prepends translation to existing content

#### Using Parameters in Templates

Pass parameters using the `t-params` attribute:

```html
<span t="welcome.message" t-params.bind="{ name: userName }"></span>
```

#### Using Value Converters

The `t` value converter provides a declarative way to translate content:

```html
<div class="greeting">
  <!-- Simple translation -->
  ${ 'greeting' | t }
  
  <!-- With parameters -->
  ${ 'welcome.message' | t: { name: userName } }
  
  <!-- With plural forms -->
  ${ 'items' | t: { count: itemCount } }
</div>
```

#### Using Binding Behaviors

For automatic updates when locale changes, use the `t` binding behavior:

```html
<div class="greeting">
  ${ 'welcome.message' & t: { name: userName } }
</div>
```

#### Working with Images

Images can be translated when different images need to be displayed for different languages:

```html
<!-- Basic image translation -->
<img t="header.logo">

<!-- With default fallback -->
<img data-src="images/default-logo.png" t="header.logo">
```

Translation file:

```json
{
  "header": {
    "logo": "images/logo-en.png"
  }
}
```

#### Complex Parameter Handling

**Object Parameters**

```javascript
// Translation file
{
  "validation": {
    "range": "Value must be between {{range.min}} and {{range.max}}"
  }
}

// Usage in code
this.i18n.tr('validation.range', {
  range: {
    min: 1,
    max: 100
  }
});
```

**Context-Based Translations**

```javascript
// Translation file
{
  "greeting": "Hello!",
  "greeting_male": "Hello sir!",
  "greeting_female": "Hello madam!"
}

// Usage
this.i18n.tr('greeting', { context: 'male' });
this.i18n.tr('greeting', { context: 'female' });
```

**Plural Forms**

```javascript
// Translation file
{
  "items": {
    "zero": "No items",
    "one": "One item",
    "few": "A few items",
    "many": "{{count}} items",
    "other": "{{count}} items"
  }
}

// Usage
this.i18n.tr('items', { count: 0 });  // "No items"
this.i18n.tr('items', { count: 1 });  // "One item"
this.i18n.tr('items', { count: 100 }); // "100 items"
```

## Formatting

### Number Formatting

#### Via Code

Use the `nf` method for programmatic number formatting:

```javascript
import {I18N} from 'aurelia-i18n';

export class MyViewModel {
  static inject = [I18N];
  
  constructor(i18n) {
    this.i18n = i18n;
    
    // Basic formatting
    let nf = this.i18n.nf();
    let formatted = nf.format(123456.789);
    
    // Currency formatting
    let currencyFormat = this.i18n.nf({ 
      style: 'currency', 
      currency: 'EUR' 
    }, 'de');
    let price = currencyFormat.format(123456.789);
  }
}
```

#### Using the Number Value Converter

Format numbers declaratively in templates:

```html
<!-- Default formatting -->
${ 1234567.89 | nf }

<!-- Currency formatting -->
${ 1234567.89 | nf: { style: 'currency', currency: 'EUR' } : 'de' }

<!-- With specific locale -->
${ 1234567.89 | nf: undefined : 'de' }
```

### Date Formatting

#### Via Code

Use the `df` method for date formatting:

```javascript
import {I18N} from 'aurelia-i18n';

export class MyViewModel {
  static inject = [I18N];
  
  constructor(i18n) {
    this.i18n = i18n;
    
    // Basic date formatting
    let df = this.i18n.df();
    let formatted = df.format(new Date());
    
    // Custom formatting
    let options = {
      year: 'numeric',
      month: 'long',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit'
    };
    
    let customDf = this.i18n.df(options, 'de');
    let customFormatted = customDf.format(new Date());
  }
}
```

#### Using the Date Value Converter

Format dates in templates:

```html
<!-- Default formatting -->
${ myDate | df }

<!-- Custom formatting -->
${ myDate | df: { 
    year: 'numeric', 
    month: 'long', 
    day: '2-digit' 
  } : 'de' }
```

### Relative Time Formatting

#### Via Code

Use the `RelativeTime` service:

```javascript
import {RelativeTime} from 'aurelia-i18n';

export class MyViewModel {
  static inject = [RelativeTime];
  
  constructor(relativeTime) {
    this.rt = relativeTime;
    
    let pastDate = new Date();
    pastDate.setHours(pastDate.getHours() - 2);
    
    let relative = this.rt.getRelativeTime(pastDate);
    // Output: "2 hours ago"
  }
}
```

#### Using the Relative Time Value Converter

Format relative times in templates:

```html
<!-- Simple relative time -->
${ myDate | rt }
```

{% hint style="info" %}
Relative time formatting automatically updates when the locale changes.
{% endhint %}

{% hint style="warning" %}
When using number or date formatting with value converters, ensure your locale binding is properly set up to trigger updates when the locale changes.
{% endhint %}

## Error Handling and Debugging

### **Missing Translations**

```javascript
// Configuration to handle missing translations
instance.setup({
  saveMissing: true,
  missingKeyHandler: (lng, ns, key, fallbackValue) => {
    console.warn(`Missing translation: ${key} for language: ${lng}`);
    // Optional: Report to error tracking service
  },
  fallbackLng: 'en',
  skipTranslationOnMissingKey: true
});
```

### **Common Issues and Solutions**

> warning: Common Pitfalls

* **Issue**: Translations not updating when locale changes
  * Solution: Ensure you're using the binding behavior (`&t`) instead of value converter (`|t`)
* **Issue**: HTML content displayed as escaped text
  * Solution: Use the `[html]` attribute: `t="[html]myKey"`
* **Issue**: Parameters not being replaced
  * Solution: Check parameter naming matches exactly in translation files

### **Debugging Tools**

```javascript
// Enable debug mode
instance.setup({
  debug: true,
  debug: {
    skipOnMissingKey: false,
    sendMissing: true,
    keySeparator: '.',
    load: 'all'
  }
});
```

## Build and Bundle Configuration

### Aurelia CLI Configuration

The latest versions of Aurelia CLI include built-in support for loading locale files. No additional configuration is required if you're using the standard CLI setup.

### Webpack Configuration

#### Using i18next-resource-store-loader

For bundling all translations with your main application:

1. Install the loader:

```bash
npm install i18next-resource-store-loader
```

2. Configure your project structure:

```
src/
  ├── assets/
  │   └── i18n/
  │       ├── index.js
  │       ├── de/
  │       │   └── translation.json
  │       └── en/
  │           └── translation.json
```

3. Configure your main.js/ts:

```javascript
import {PLATFORM} from 'aurelia-pal';
import resBundle from 'i18next-resource-store-loader!./assets/i18n/index.js';

export function configure(aurelia) {
  aurelia.use
    .plugin(PLATFORM.moduleName('aurelia-i18n'), instance => {
      return instance.setup({
        resources: resBundle,
        lng: 'en',
        fallbackLng: 'en',
        debug: false
      });
    });
}
```

#### Copying Translation Files

Add to your webpack.config.js to copy translation files:

```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  plugins: [
    new CopyWebpackPlugin([
      { from: 'src/locales/', to: 'locales/' }
    ])
  ]
};
```

## API Reference

### Core Classes

#### I18N Class

**Properties**

* `i18next`: The underlying i18next instance
* `ea`: EventAggregator instance
* `Intl`: Reference to the Intl API

**Methods**

* `setup(options: I18NConfigOptions): Promise<i18next>`
  * Initializes the plugin with the provided options
  * Returns a promise that resolves when initialization is complete
* `setLocale(locale: string): Promise<any>`
  * Changes the active locale
  * Returns a promise that resolves when locale is changed
* `getLocale(): string`
  * Returns the current active locale
* `tr(key: string, options?: any): string`
  * Translates the given key with optional parameters
  * Returns the translated string
* `nf(options?: Intl.NumberFormatOptions, locale?: string): Intl.NumberFormat`
  * Creates a number formatter
  * Returns NumberFormat instance
* `df(options?: Intl.DateTimeFormatOptions, locale?: string): Intl.DateTimeFormat`
  * Creates a date formatter
  * Returns DateTimeFormat instance

#### TCustomAttribute

**Static Methods**

* `configureAliases(aliases: string[]): void`
  * Configures custom aliases for the translation attribute

**Properties**

* `service: I18N`
* `element: Element`
* `params: any`

### Configuration Interfaces

#### I18NConfigOptions

```typescript
interface I18NConfigOptions {
  lng?: string;                 // Default language
  fallbackLng?: string;        // Fallback language
  debug?: boolean;             // Enable debug mode
  defaultNS?: string;          // Default namespace
  ns?: string[];              // Available namespaces
  attributes?: string[];      // Custom attribute aliases
  backend?: any;              // Backend configuration
  resources?: any;            // Bundled resources
  skipTranslationOnMissingKey?: boolean; // Keep original content on missing keys
}
```

### Events and Signals

#### Event Aggregator Events

* `i18n:locale:changed`
  * Triggered when locale changes
  * Payload: `{ oldValue: string, newValue: string }`

#### Binding Signals

* `aurelia-translation-signal`
  * Triggers update of all t binding behaviors
* `aurelia-relativetime-signal`
  * Triggers update of relative time bindings

### Value Converters

#### TValueConverter

* Parameters:
  * `value: string`: Translation key
  * `options?: any`: Translation options
  * `locale?: string`: Specific locale (optional)

#### NfValueConverter

* Parameters:
  * `value: number`: Number to format
  * `options?: Intl.NumberFormatOptions`: Formatting options
  * `locale?: string`: Specific locale (optional)

#### DfValueConverter

* Parameters:
  * `value: Date`: Date to format
  * `options?: Intl.DateTimeFormatOptions`: Formatting options
  * `locale?: string`: Specific locale (optional)

#### RtValueConverter

* Parameters:
  * `value: Date`: Date to format relative to now

### Backend Options

#### XHR Backend

```typescript
interface XHRBackendOptions {
  loadPath: string;           // Path template for loading translations
  addPath?: string;          // Path template for adding missing keys
  allowMultiLoading?: boolean; // Allow loading multiple namespaces
  parse?: (data: string) => any; // Custom parser function
  crossDomain?: boolean;     // Enable cross-domain requests
  withCredentials?: boolean; // Send cookies in cross-domain requests
}
```

#### Aurelia Loader Backend

```typescript
interface AureliaLoaderOptions {
  loadPath: string;          // Path template for loading translations
  parse?: (data: string) => any; // Custom parser function
}
```

{% hint style="info" %}
Some features may require additional configuration depending on your build setup and environment.
{% endhint %}

{% hint style="info" %}
All configuration options from i18next are also supported. Refer to i18next documentation for additional options.
{% endhint %}

## Integrations

### Using with Aurelia Validation

```javascript
import {I18N} from 'aurelia-i18n';
import {ValidationMessageProvider} from 'aurelia-validation';

// Configure validation messages to use i18n
ValidationMessageProvider.prototype.getMessage = function(key) {
  const i18n = Container.instance.get(I18N);
  const translation = i18n.tr(`validation.messages.${key}`);
  return this.parser.parse(translation);
};

// Translation file (locales/en/validation.json)
{
  "validation": {
    "messages": {
      "required": "${$displayName} is required",
      "email": "${$displayName} is not a valid email",
      "minLength": "${$displayName} must be at least ${$config.length} characters",
      "maxLength": "${$displayName} cannot be longer than ${$config.length} characters"
    }
  }
}
```

### Integration with Aurelia Dialog

```javascript
// dialog-service.js
import {I18N} from 'aurelia-i18n';
import {DialogService} from 'aurelia-dialog';

export class LocalizedDialogService {
  static inject = [DialogService, I18N];
  
  constructor(dialogService, i18n) {
    this.dialogService = dialogService;
    this.i18n = i18n;
  }
  
  open(viewModel, model) {
    return this.dialogService.open({
      viewModel,
      model: {
        ...model,
        title: this.i18n.tr(model.titleKey),
        message: this.i18n.tr(model.messageKey, model.messageParams)
      }
    });
  }
}

// Usage
this.localizedDialog.open(ConfirmDialog, {
  titleKey: 'dialogs.confirm.title',
  messageKey: 'dialogs.confirm.deleteUser',
  messageParams: { username: user.name }
});
```

### Custom Backend Integration

```javascript
// custom-backend.js
class CustomApiBackend {
  constructor(services, options = {}) {
    this.init(services, options);
  }

  init(services, options) {
    this.services = services;
    this.options = options;
  }

  read(language, namespace, callback) {
    fetch(`${this.options.apiUrl}/translations/${language}/${namespace}`)
      .then(response => response.json())
      .then(data => callback(null, data))
      .catch(error => callback(error, null));
  }
}

// main.js
instance.i18next.use(new CustomApiBackend());
```

## Real-World Examples

### Dynamic Form with Translations

```html
<!-- form-builder.html -->
<template>
  <form submit.delegate="submit()">
    <div repeat.for="field of formFields">
      <label t.bind="field.labelKey"></label>
      
      <div if.bind="field.type === 'text'">
        <input type="text" 
               value.bind="field.value"
               placeholder.bind="field.placeholderKey | t"
               validation-errors.bind="field.errors">
      </div>
      
      <div if.bind="field.type === 'select'">
        <select value.bind="field.value">
          <option repeat.for="option of field.options"
                  value.bind="option.value">
            ${option.labelKey | t}
          </option>
        </select>
      </div>
      
      <div class="error-messages">
        <span repeat.for="error of field.errors"
              class="error-message">
          ${error.messageKey | t:error.params}
        </span>
      </div>
    </div>
    
    <button type="submit" t="forms.submit"></button>
  </form>
</template>
```

```javascript
// form-builder.js
export class FormBuilder {
  static inject = [I18N, ValidationController];
  
  constructor(i18n, validationController) {
    this.i18n = i18n;
    this.validationController = validationController;
    
    this.formFields = [
      {
        type: 'text',
        labelKey: 'forms.user.name.label',
        placeholderKey: 'forms.user.name.placeholder',
        value: '',
        validation: {
          required: true,
          minLength: 3
        }
      },
      {
        type: 'select',
        labelKey: 'forms.user.role.label',
        value: '',
        options: [
          { value: 'admin', labelKey: 'forms.user.role.options.admin' },
          { value: 'user', labelKey: 'forms.user.role.options.user' }
        ]
      }
    ];
  }
}
```

### Dynamic Menu with Language Switching

```html
<!-- nav-menu.html -->
<template>
  <nav class="main-nav">
    <ul>
      <li repeat.for="item of menuItems">
        <a href.bind="item.route" t.bind="item.titleKey"></a>
        
        <ul if.bind="item.children">
          <li repeat.for="child of item.children">
            <a href.bind="child.route" t.bind="child.titleKey"></a>
          </li>
        </ul>
      </li>
    </ul>
    
    <div class="language-selector">
      <select value.bind="currentLanguage" 
              change.delegate="switchLanguage($event.target.value)">
        <option repeat.for="lang of availableLanguages"
                value.bind="lang.code">
          ${lang.name}
        </option>
      </select>
    </div>
  </nav>
</template>
```

```javascript
// nav-menu.js
export class NavMenu {
  static inject = [I18N, EventAggregator];
  
  constructor(i18n, ea) {
    this.i18n = i18n;
    this.ea = ea;
    
    this.availableLanguages = [
      { code: 'en', name: 'English' },
      { code: 'de', name: 'Deutsch' },
      { code: 'fr', name: 'Français' }
    ];
    
    this.menuItems = [
      {
        titleKey: 'nav.home',
        route: '#/',
      },
      {
        titleKey: 'nav.admin',
        route: '#/admin',
        children: [
          {
            titleKey: 'nav.admin.users',
            route: '#/admin/users'
          },
          {
            titleKey: 'nav.admin.settings',
            route: '#/admin/settings'
          }
        ]
      }
    ];
    
    this.currentLanguage = this.i18n.getLocale();
    
    this.ea.subscribe('i18n:locale:changed', payload => {
      this.currentLanguage = payload.newValue;
    });
  }
  
  async switchLanguage(newLanguage) {
    await this.i18n.setLocale(newLanguage);
    // Optional: Save preference to user settings
    localStorage.setItem('preferredLanguage', newLanguage);
  }
}
```

### Localized Data Grid

```html
<!-- data-grid.html -->
<template>
  <div class="grid-controls">
    <div class="search">
      <input type="text" 
             value.bind="searchTerm" 
             placeholder.bind="'grid.search' | t">
    </div>
    
    <div class="per-page">
      <select value.bind="itemsPerPage">
        <option repeat.for="size of pageSizes" value.bind="size">
          ${'grid.itemsPerPage' | t: { count: size }}
        </option>
      </select>
    </div>
  </div>

  <table>
    <thead>
      <tr>
        <th repeat.for="column of columns"
            click.delegate="sort(column)">
          ${column.headerKey | t}
          <i if.bind="sortColumn === column.field" 
             class="sort-icon ${sortDirection}">
          </i>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr repeat.for="item of paginatedItems">
        <td repeat.for="column of columns">
          <!-- Handle different column types -->
          <span if.bind="column.type === 'text'">
            ${item[column.field]}
          </span>
          <span if.bind="column.type === 'date'">
            ${item[column.field] | df:column.format}
          </span>
          <span if.bind="column.type === 'number'">
            ${item[column.field] | nf:column.format}
          </span>
          <span if.bind="column.type === 'translated'">
            ${column.prefix + item[column.field] | t}
          </span>
        </td>
      </tr>
    </tbody>
  </table>

  <div class="pagination">
    <button click.delegate="previousPage()" 
            disabled.bind="currentPage === 1">
      ${'grid.previous' | t}
    </button>
    <span>
      ${'grid.pageInfo' | t: { 
        current: currentPage, 
        total: totalPages 
      }}
    </span>
    <button click.delegate="nextPage()" 
            disabled.bind="currentPage === totalPages">
      ${'grid.next' | t}
    </button>
  </div>
</template>
```

## Best Practices and Common Patterns

### Project Organization

#### Recommended Translation Structure

```
src/
  ├── locales/
  │   ├── en/
  │   │   ├── translation.json     # General translations
  │   │   ├── validation.json      # Form validation messages
  │   │   ├── navigation.json      # Navigation-related texts
  │   │   └── errors.json          # Error messages
  │   └── de/
  │       └── ...
```

#### Translation Key Naming Conventions

```javascript
{
  // Group related translations
  "auth": {
    "login": {
      "title": "Login",
      "emailLabel": "Email Address",
      "passwordLabel": "Password",
      "submitButton": "Sign In",
      "errors": {
        "invalidCredentials": "Invalid email or password"
      }
    }
  },
  
  // Use dots for deeply nested structures
  "common.buttons.save": "Save",
  "common.buttons.cancel": "Cancel"
}
```

### Translation Tips

#### Keep Translations Maintainable

```javascript
// Bad - Hard to maintain and reuse
{
  "welcomeMessage": "Welcome to our app! Click here to get started."
}

// Good - Separate reusable components
{
  "welcome": {
    "greeting": "Welcome to our app!",
    "callToAction": "Click here to get started"
  }
}
```

#### Handle Pluralization Properly

```javascript
{
  "items": {
    "zero": "No items",
    "one": "{{count}} item",
    "other": "{{count}} items"
  }
}
```

#### Use Parameters Effectively

```javascript
// Bad - Hard-coded values
{
  "lastLogin": "You last logged in on Monday at 2:00 PM"
}

// Good - Flexible parameters
{
  "lastLogin": "You last logged in on {{day}} at {{time}}"
}
```

### Performance Considerations

#### Lazy Loading

* Load translations by namespace when needed
* Consider splitting large translation files
* Use route-based translation loading for large applications

```javascript
// Configure lazy loading
instance.setup({
  ns: ['common'],
  partialBundledLanguages: true,
  load: 'languageOnly'
});

// Load additional namespace when needed
i18next.loadNamespaces(['specific-feature'])
```

#### Caching

* Enable localStorage caching for faster subsequent loads
* Configure cache expiration based on your update frequency

```javascript
instance.setup({
  cache: {
    enabled: true,
    expirationTime: 7 * 24 * 60 * 60 * 1000 // 7 days
  }
});
```

### Common Patterns

#### Handling Dynamic Content

```html
<!-- Using interpolation -->
<div t="[html]content.description" t-params.bind="{ 
  link: '<a href=\'https://example.com\'>Click here</a>' 
}"></div>

<!-- In translation file -->
{
  "content": {
    "description": "Learn more about our service. {{link}}"
  }
}
```

#### Form Validation Messages

```javascript
{
  "validation": {
    "required": "{{field}} is required",
    "minLength": "{{field}} must be at least {{min}} characters",
    "email": "Please enter a valid email address"
  }
}
```

#### Error Handling

```javascript
// Handle missing translations gracefully
instance.setup({
  fallbackLng: 'en',
  saveMissing: true,
  missingKeyHandler: (lng, ns, key) => {
    console.warn(`Missing translation: ${key} in ${lng}/${ns}`);
    // Optional: Report to error tracking service
  }
});
```

### Testing

#### Translation Key Coverage

```javascript
// Test helper to verify translation key existence
function verifyTranslationKey(key, locale = 'en') {
  const translation = i18n.tr(key);
  expect(translation).not.toBe(key); // Should not return key itself
  expect(translation.includes('{{}')).toBe(false); // No unprocessed parameters
}
```

#### Automated Checks

* Verify all locales have the same keys
* Check for missing translations
* Validate parameter usage

{% hint style="info" %}
Consider implementing a CI check to ensure translation files are properly formatted and complete.
{% endhint %}

{% hint style="warning" %}
Always test your applications with different locales to catch any internationalization-related issues early.
{% endhint %}
