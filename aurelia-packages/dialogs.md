# Dialogs

### Introduction

#### Overview

The Aurelia Dialog plugin provides a flexible and powerful way to display modal dialogs in your Aurelia applications. It supports dynamic content, customizable styling, and offers a robust API for managing dialog interactions.

#### Key Features

* Dynamic content loading
* Customizable styling and layouts
* Promise-based API for handling dialog results
* Built-in components for common dialog elements
* Lifecycle hooks for fine-grained control
* Support for both modal and non-modal dialogs
* Keyboard navigation support
* Native dialog renderer support

### Installation and Setup

#### Installing the Plugin

Install the dialog plugin using npm:

```bash
npm install aurelia-dialog
```

#### Basic Configuration

To use the dialog plugin, you need to configure it in your application's main entry point. First, ensure you're using manual bootstrapping by modifying your `index.html`:

```html
<body aurelia-app="main">
  <!-- Your app content -->
</body>
```

Then, configure the plugin in your `main.js`:

```javascript
import { PLATFORM } from 'aurelia-pal';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .plugin(PLATFORM.moduleName('aurelia-dialog'));

  aurelia.start().then(a => a.setRoot());
}
```

{% hint style="warning" %}
When using Webpack, always use `PLATFORM.moduleName()` to mark dynamic dependencies.
{% endhint %}

Configuration Options

For more control, you can provide a configuration callback:

```javascript
import { PLATFORM } from 'aurelia-pal';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .plugin(PLATFORM.moduleName('aurelia-dialog'), config => {
      config.useDefaults();
      config.settings.lock = true;
      config.settings.centerHorizontalOnly = false;
      config.settings.startingZIndex = 5;
      config.settings.keyboard = true;
    });

  aurelia.start().then(a => a.setRoot());
}
```

### Basic Usage

#### Dialog Service Overview

The Dialog Service is the main entry point for creating and managing dialogs. Import it into your view-model:

```javascript
import { DialogService } from 'aurelia-dialog';
```

#### Opening Dialogs

There are two main approaches to opening dialogs:

1. **Using a Simple Prompt**:

```javascript
import { DialogService } from 'aurelia-dialog';
import { Prompt } from './prompt';

export class Welcome {
  static inject = [DialogService];
  
  constructor(dialogService) {
    this.dialogService = dialogService;
  }
  
  openDialog() {
    this.dialogService.open({
      viewModel: Prompt,
      model: 'Are you sure?',
      lock: false
    });
  }
}
```

2. **Using Custom Dialog Components**:

```javascript
import { DialogService } from 'aurelia-dialog';
import { EditPerson } from './edit-person';

export class Welcome {
  static inject = [DialogService];
  
  constructor(dialogService) {
    this.dialogService = dialogService;
  }
  
  person = { firstName: 'John', lastName: 'Doe' };
  
  editPerson() {
    this.dialogService.open({
      viewModel: EditPerson,
      model: this.person,
      lock: true
    });
  }
}
```

#### Handling Dialog Results

Dialog operations return promises that resolve when the dialog closes. Use the `whenClosed` method to handle results:

```javascript
this.dialogService.open({
  viewModel: EditPerson,
  model: this.person
}).whenClosed(response => {
  if (!response.wasCancelled) {
    // User clicked OK
    console.log('Dialog output:', response.output);
  } else {
    // User clicked cancel or clicked outside
    console.log('Dialog cancelled');
  }
});
```

#### Using Dialog Controller

When creating custom dialog components, use the DialogController to manage the dialog:

```javascript
import { DialogController } from 'aurelia-dialog';

export class EditPerson {
  static inject = [DialogController];
  
  constructor(controller) {
    this.controller = controller;
  }
  
  activate(person) {
    this.person = person;
  }
  
  // Example template
  /*
  <ux-dialog>
    <ux-dialog-body>
      <input value.bind="person.firstName">
    </ux-dialog-body>
    <ux-dialog-footer>
      <button click.trigger="controller.cancel()">Cancel</button>
      <button click.trigger="controller.ok(person)">Save</button>
    </ux-dialog-footer>
  </ux-dialog>
  */
}
```

{% hint style="info" %}
The DialogController provides methods like `ok()`, `cancel()`, and `error()` for closing the dialog with different results.
{% endhint %}

### Dialog Components

#### Built-in Components

The dialog plugin provides several built-in components for constructing dialog interfaces:

* `<ux-dialog>`: The main container component
* `<ux-dialog-header>`: Header section of the dialog
* `<ux-dialog-body>`: Main content area
* `<ux-dialog-footer>`: Footer section, typically for buttons

Example of a complete dialog template:

```html
<template>
  <ux-dialog>
    <ux-dialog-header>
      <h2>${title}</h2>
    </ux-dialog-header>

    <ux-dialog-body>
      <div class="form-group">
        <label>First Name</label>
        <input type="text" value.bind="person.firstName">
      </div>
    </ux-dialog-body>

    <ux-dialog-footer>
      <button click.trigger="controller.cancel()">Cancel</button>
      <button click.trigger="controller.ok(person)">Save</button>
    </ux-dialog-footer>
  </ux-dialog>
</template>
```

#### attach-focus Attribute

The `attach-focus` attribute allows you to automatically focus an element when the dialog opens:

```html
<!-- Simple usage -->
<input attach-focus value.bind="searchTerm">

<!-- Conditional focus -->
<input attach-focus.bind="isNewUser" value.bind="email">
<input attach-focus.bind="!isNewUser" value.bind="username">
```

{% hint style="info" %}
The `attach-focus` attribute only works during the initial attachment. For dynamic focus changes, use Aurelia's `focus` attribute instead.
{% endhint %}

#### Controlling Default Resources

You can control which default resources are registered:

```javascript
export function configure(aurelia) {
  aurelia.use
    .plugin(PLATFORM.moduleName('aurelia-dialog'), config => {
      // Register only specific resources
      config.useResource('attach-focus');
      // Or register none
      config.useResource(false);
    });
}
```

### Configuration and Settings

#### Global Settings

Configure global settings that apply to all dialogs:

```javascript
export function configure(aurelia) {
  aurelia.use
    .plugin(PLATFORM.moduleName('aurelia-dialog'), config => {
      config.useDefaults();
      config.settings = {
        lock: true,                     // Lock modal behind dialog
        centerHorizontalOnly: false,    // Center dialog horizontally
        startingZIndex: 1000,          // Starting z-index value
        keyboard: true,                 // Enable keyboard navigation
        overlayDismiss: false          // Click outside to close
      };
    });
}
```

#### Per-dialog Settings

Configure settings for individual dialogs:

```javascript
export class DialogViewModel {
  static inject = [DialogController];
  
  constructor(controller) {
    this.controller = controller;
    
    // Configure this specific dialog
    controller.settings = {
      viewModel: EditPerson,            // View model class, URL, or instance
      model: personData,                // Data to pass to dialog
      host: customElement,              // Custom host element (optional)
      childContainer: container,        // Custom DI container (optional)
      lock: true,                       // Lock modal behind dialog
      keyboard: ['Escape', 'Enter'],    // Keyboard keys that close dialog
      overlayDismiss: true,            // Click outside to close
      centerHorizontalOnly: false,      // Center horizontally only
      ignoreTransitions: false,         // Disable CSS animations
      rejectOnCancel: false            // Reject promise on cancel
    };
  }
}
```

#### Position Callback

Use the position callback for custom positioning:

```javascript
this.dialogService.open({
  viewModel: CustomDialog,
  position: (modalContainer, modalOverlay) => {
    const { clientHeight, clientWidth } = document.documentElement;
    const { offsetHeight, offsetWidth } = modalContainer;
    
    modalContainer.style.left = `${(clientWidth - offsetWidth) / 2}px`;
    modalContainer.style.top = `${(clientHeight - offsetHeight) / 2}px`;
  }
});
```

### Dialog Lifecycle

#### Lifecycle Hooks Overview

Dialogs support several lifecycle hooks that execute in a specific order:

**canActivate(model)**

Controls whether the dialog can be opened:

```javascript
export class EditDialog {
  canActivate(model) {
    // Return false to prevent dialog from opening
    return userHasPermission(model.id);
  }
}
```

**activate(model)**

Initializes the dialog with the provided model:

```javascript
export class EditDialog {
  activate(model) {
    this.originalData = {...model};
    this.editableData = {...model};
  }
}
```

**canDeactivate(result)**

Controls whether the dialog can be closed:

```javascript
export class EditDialog {
  canDeactivate(result) {
    if (result.wasCancelled) {
      return true;
    }
    // Prevent closing if form is invalid
    return this.form.checkValidity();
  }
}
```

**deactivate(result)**

Cleanup when dialog is closing:

```javascript
export class EditDialog {
  deactivate(result) {
    if (!result.wasCancelled) {
      this.saveChanges();
    }
    this.resetForm();
  }
}
```

#### Lifecycle Execution Order

1. Constructor
2. `canActivate()`
3. `activate()`
4. `created()`
5. `bind()`
6. `attached()`
7. `canDeactivate()`
8. `deactivate()`
9. `detached()`
10. `unbind()`

```javascript
export class CompleteDialog {
  constructor(controller) {
    this.controller = controller;
  }

  canActivate(model) {
    return Promise.resolve(true);
  }

  activate(model) {
    this.data = model;
  }

  created(owningView, myView) {
    // View creation complete
  }

  bind(bindingContext, overrideContext) {
    // Binding phase
  }

  attached() {
    // Element attached to DOM
  }

  canDeactivate(result) {
    return this.hasUnsavedChanges ? 
      confirm('Discard changes?') : 
      true;
  }

  deactivate(result) {
    // Cleanup
    this.data = null;
  }
}
```

{% hint style="warning" %}
When using `DialogController.prototype.error()`, the `canDeactivate` hook will be skipped.
{% endhint %}

All lifecycle hooks can return promises, which will be awaited before proceeding to the next phase. This allows for asynchronous operations during the dialog lifecycle.

### Customization

#### Styling Dialogs

**Default Styles Override**

The default styles are included when calling `config.useDefaults()`. To customize styles, you can override the default CSS:

```javascript
export function configure(aurelia) {
  aurelia.use
    .plugin(PLATFORM.moduleName('aurelia-dialog'), config => {
      config.useRenderer(defaultRenderer)
        .useCSS('') // Remove default styles
        .useStandardResources();
    });
}
```

**Custom CSS Examples**

```css
/* Basic dialog styling */
ux-dialog-container {
  display: flex;
  align-items: center;
  justify-content: center;
}

ux-dialog {
  max-width: 500px;
  min-width: 300px;
  background: white;
  border-radius: 4px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.15);
}

/* Overlay customization */
ux-dialog-overlay.active {
  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(2px);
}

/* Dialog sections */
ux-dialog-header {
  padding: 16px;
  border-bottom: 1px solid #eee;
}

ux-dialog-body {
  padding: 16px;
}

ux-dialog-footer {
  padding: 16px;
  border-top: 1px solid #eee;
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}
```

#### Custom Renderers

Create a custom renderer by implementing the `DialogRenderer` interface:

```javascript
import { DialogRenderer } from 'aurelia-dialog';

export class CustomDialogRenderer extends DialogRenderer {
  getDialogContainer() {
    const container = document.createElement('div');
    container.classList.add('custom-dialog-container');
    return container;
  }

  showDialog(dialogController) {
    // Custom showing logic
    return super.showDialog(dialogController);
  }

  hideDialog(dialogController) {
    // Custom hiding logic
    return super.hideDialog(dialogController);
  }
}

// Register custom renderer
aurelia.use
  .plugin(PLATFORM.moduleName('aurelia-dialog'), config => {
    config.useRenderer(CustomDialogRenderer);
  });
```

#### Using Native Dialog Renderer

The plugin supports native HTML `<dialog>` element:

```javascript
import { NativeDialogRenderer } from 'aurelia-dialog';

aurelia.use
  .plugin(PLATFORM.moduleName('aurelia-dialog'), config => {
    config.useRenderer(NativeDialogRenderer);
  });
```

### Advanced

#### Accessing Dialog Controller API

Access and control dialogs from the opening context:

```javascript
export class Welcome {
  static inject = [DialogService];
  
  constructor(dialogService) {
    this.dialogService = dialogService;
  }

  async openDialog() {
    const openDialogResult = await this.dialogService.open({
      viewModel: EditPerson,
      model: this.person
    });

    // Access dialog controller
    const controller = openDialogResult.controller;

    // Set up external control
    this.closeDialog = () => controller.cancel();
    
    // Wait for dialog result
    const result = await openDialogResult.closeResult;
    
    if (!result.wasCancelled) {
      await this.savePerson(result.output);
    }
  }
}
```

#### Custom Dialog Implementations

Create complex dialogs with multiple steps:

```javascript
export class WizardDialog {
  static inject = [DialogController];
  
  constructor(controller) {
    this.controller = controller;
    this.steps = ['basic', 'details', 'confirm'];
    this.currentStep = 0;
    this.data = {};
  }

  nextStep() {
    if (this.currentStep < this.steps.length - 1) {
      this.currentStep++;
      return true;
    }
    return this.controller.ok(this.data);
  }

  previousStep() {
    if (this.currentStep > 0) {
      this.currentStep--;
    }
  }

  canDeactivate() {
    if (this.hasUnsavedChanges) {
      return confirm('Are you sure you want to close the wizard?');
    }
    return true;
  }
}
```

```html
<template>
  <ux-dialog>
    <ux-dialog-header>
      <h2>Step ${currentStep + 1}: ${steps[currentStep]}</h2>
    </ux-dialog-header>

    <ux-dialog-body>
      <div if.bind="currentStep === 0">
        <!-- Basic Info Form -->
      </div>
      <div if.bind="currentStep === 1">
        <!-- Details Form -->
      </div>
      <div if.bind="currentStep === 2">
        <!-- Confirmation -->
      </div>
    </ux-dialog-body>

    <ux-dialog-footer>
      <button click.trigger="controller.cancel()">Cancel</button>
      <button click.trigger="previousStep()" 
              disabled.bind="currentStep === 0">Previous</button>
      <button click.trigger="nextStep()">
        ${currentStep === steps.length - 1 ? 'Finish' : 'Next'}
      </button>
    </ux-dialog-footer>
  </ux-dialog>
</template>
```

#### Dynamic Dialog Content

Load dialog content dynamically:

```javascript
export class DynamicDialog {
  static inject = [DialogController];

  constructor(controller) {
    this.controller = controller;
  }

  activate(model) {
    this.componentType = model.type;
    this.componentData = model.data;
  }
}
```

```html
<template>
  <ux-dialog>
    <ux-dialog-body>
      <compose
        view-model.bind="componentType"
        model.bind="componentData">
      </compose>
    </ux-dialog-body>
  </ux-dialog>
</template>
```

Usage:

```javascript
this.dialogService.open({
  viewModel: DynamicDialog,
  model: {
    type: PLATFORM.moduleName('./components/special-form'),
    data: { id: 123 }
  }
});
```

{% hint style="info" %}
When using dynamic content loading, ensure all potential components are properly bundled and available in your application.
{% endhint %}

These advanced features allow you to create sophisticated dialog implementations that can handle complex scenarios while maintaining clean, maintainable code.
