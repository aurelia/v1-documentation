# Delegate vs Trigger

When binding to events in Aurelia, you have two primary commands to choose from: `delegate` and `trigger`. Understanding the differences between these two approaches is crucial for writing efficient and performant applications.

## The Golden Rule

{% hint style="success" %}
**Use `delegate` except when you cannot use `delegate`.**
{% endhint %}

This simple rule covers most scenarios, but understanding why and when exceptions occur will help you make informed decisions.

## Understanding Event Delegation

### What is Event Delegation?

Event delegation is a technique where instead of attaching event listeners to individual elements, you attach a single event listener to a parent element (typically the document) and handle events as they bubble up from child elements.

### How Aurelia's Delegate Works

```html
<button click.delegate="handleClick()">Click me</button>
<button click.delegate="handleAnotherClick()">Click me too</button>
```

With `delegate`, Aurelia:

1. Attaches **one** event listener to the document (or nearest shadow DOM boundary)
2. When any click event occurs, it checks if the event target has a delegated handler
3. Executes the appropriate handler for that specific element

This means hundreds of buttons only require **one** actual event listener, making it highly efficient.

### Benefits of Delegation

- **Memory Efficient**: One listener handles multiple elements
- **Performance**: Fewer DOM event subscriptions
- **Dynamic Content**: Works with elements added/removed dynamically
- **Event Bubbling**: Leverages natural browser behavior

## When Delegate Works

Most DOM events bubble up the DOM tree, making them perfect candidates for delegation:

### Common Delegatable Events
- `click`
- `dblclick` 
- `mousedown`
- `mouseup`
- `keydown`
- `keyup`
- `submit`
- `change` (on most form elements)
- `input`

### Example: Multiple Buttons with Delegation

```html
<template>
  <div class="button-group">
    <button click.delegate="save()">Save</button>
    <button click.delegate="cancel()">Cancel</button>
    <button click.delegate="delete()">Delete</button>
    <button click.delegate="export()">Export</button>
  </div>
</template>
```

```javascript
export class ButtonGroup {
  save() {
    console.log('Saving...');
  }

  cancel() {
    console.log('Cancelled');
  }

  delete() {
    console.log('Deleting...');
  }

  export() {
    console.log('Exporting...');
  }
}
```

All four buttons share a single event listener on the document, yet each executes its own handler correctly.

## When to Use Trigger

### Non-Bubbling Events

Some events don't bubble, which means they can't be delegated:

```html
<!-- These events require trigger -->
<input focus.trigger="onFocus()" blur.trigger="onBlur()">
<img load.trigger="onImageLoad()" error.trigger="onImageError()">
<audio ended.trigger="onAudioEnd()">
```

### Common Non-Bubbling Events
- `focus`
- `blur` 
- `load`
- `unload`
- `error`
- `resize` (on elements, not window)
- `scroll` (on elements, not window)

### Disabled Elements

Disabled form elements don't participate in event bubbling properly:

```html
<!-- Disabled buttons should use trigger -->
<button disabled.bind="isProcessing" 
        click.trigger="handleDisabledClick()">
  ${isProcessing ? 'Processing...' : 'Submit'}
</button>
```

### Complex Button Content

Buttons with complex nested content can have event propagation issues:

```html
<!-- Complex button content - use trigger for reliability -->
<button click.trigger="selectProduct(product)">
  <img src.bind="product.image" alt="Product image">
  <div class="product-info">
    <h3>${product.name}</h3>
    <span class="price">${product.price}</span>
    <div class="rating">
      <span repeat.for="star of product.stars">★</span>
    </div>
  </div>
</button>
```

### iOS Click Events

iOS Safari has specific requirements for click events on non-interactive elements:

```html
<!-- iOS requires trigger for non-input elements -->
<div class="custom-button" 
     click.trigger="handleTouch()"
     style="cursor: pointer;">
  Tap me on iOS
</div>
```

## Using Capture

The `capture` command is a specialized option for specific scenarios:

### When Capture is Useful

```html
<div class="modal-container" 
     click.capture="handleModalClick($event)">
  <div class="modal-content">
    <!-- Plugin content that might stop event propagation -->
    <third-party-widget></third-party-widget>
    <button click.delegate="closeModal()">Close</button>
  </div>
</div>
```

```javascript
export class ModalDialog {
  handleModalClick(event) {
    // This runs even if the third-party widget calls event.stopPropagation()
    if (event.target === event.currentTarget) {
      this.closeModal(); // Click on backdrop
    }
  }

  closeModal() {
    // Close modal logic
  }
}
```

### Capture Behavior

- Runs during the **capturing phase** (before the event reaches the target)
- Executes even if `event.stopPropagation()` is called later
- Useful for intercepting events before they reach child elements

{% hint style="warning" %}
Use `capture` sparingly as it can be confusing and is rarely needed in typical applications.
{% endhint %}

## Performance Comparison

### Memory Usage Example

```html
<!-- With trigger: 1000 event listeners -->
<div repeat.for="item of 1000Items">
  <button click.trigger="selectItem(item)">Select ${item.name}</button>
</div>

<!-- With delegate: 1 event listener -->
<div repeat.for="item of 1000Items">
  <button click.delegate="selectItem(item)">Select ${item.name}</button>
</div>
```

The difference becomes significant with large lists or frequently created/destroyed elements.

### Performance Best Practices

1. **Default to `delegate`** for most interactive elements
2. **Use `trigger`** only when delegation doesn't work
3. **Avoid `capture`** unless you specifically need capturing behavior
4. **Test on target devices**, especially iOS for touch events

## Event Object and preventDefault

### Automatic preventDefault Behavior

Aurelia automatically calls `preventDefault()` on events handled with both `delegate` and `trigger`:

```javascript
export class FormHandler {
  handleSubmit(event) {
    // event.preventDefault() is automatically called
    console.log('Form submitted');
    // Return true to disable automatic preventDefault
    return true; // Allows default browser behavior
  }

  handleClick() {
    // preventDefault is called automatically
    console.log('Button clicked');
    // Don't return true - prevent default behavior
  }
}
```

### Accessing the Event Object

```html
<button click.delegate="handleClick($event)">Click me</button>
<input keydown.trigger="handleKeydown($event)">
<form submit.delegate="handleSubmit($event)">
```

```javascript
export class EventHandler {
  handleClick(event) {
    console.log('Clicked element:', event.target);
    console.log('Event type:', event.type);
    // Return true to allow default behavior
  }

  handleKeydown(event) {
    if (event.key === 'Enter') {
      this.submitForm();
      return false; // Prevent default
    }
    return true; // Allow default
  }

  handleSubmit(event) {
    console.log('Form data:', new FormData(event.target));
    // Automatically prevents default form submission
  }
}
```

## Real-World Examples

### Data Table with Row Actions

```html
<table class="data-table">
  <tbody>
    <tr repeat.for="user of users">
      <td>${user.name}</td>
      <td>${user.email}</td>
      <td class="actions">
        <!-- Use delegate for efficiency with many rows -->
        <button click.delegate="editUser(user)" class="btn-edit">
          Edit
        </button>
        <button click.delegate="deleteUser(user)" class="btn-delete">
          Delete
        </button>
        <!-- Disabled state might need trigger -->
        <button disabled.bind="user.isProcessing" 
                click.trigger="processUser(user)">
          ${user.isProcessing ? 'Processing...' : 'Process'}
        </button>
      </td>
    </tr>
  </tbody>
</table>
```

### Media Player Controls

```html
<div class="media-player">
  <video ref="videoElement" 
         load.trigger="onVideoLoad()"
         ended.trigger="onVideoEnd()"
         error.trigger="onVideoError()">
  </video>
  
  <div class="controls">
    <!-- Use delegate for control buttons -->
    <button click.delegate="play()">Play</button>
    <button click.delegate="pause()">Pause</button>
    <button click.delegate="stop()">Stop</button>
    
    <!-- Volume control might need trigger for slider events -->
    <input type="range" 
           input.trigger="updateVolume($event)"
           min="0" max="100" 
           value.bind="volume">
  </div>
</div>
```

### Form with Complex Validation

```html
<form submit.delegate="handleSubmit($event)">
  <div class="form-group">
    <input type="text" 
           value.bind="username"
           blur.trigger="validateUsername()"
           focus.trigger="clearUsernameError()">
    <span if.bind="usernameError" class="error">
      ${usernameError}
    </span>
  </div>
  
  <div class="form-actions">
    <!-- Use delegate for form buttons -->
    <button type="submit" 
            disabled.bind="!isFormValid"
            click.delegate="handleSubmit($event)">
      Submit
    </button>
    <button type="button" 
            click.delegate="resetForm()">
      Reset
    </button>
  </div>
</form>
```

## Troubleshooting Common Issues

### Event Not Firing with Delegate

**Problem**: Event handler not executing with `delegate`

**Solution**: Check if the event bubbles
```html
<!-- Won't work - focus doesn't bubble -->
<input focus.delegate="onFocus()">

<!-- Fix: Use trigger -->
<input focus.trigger="onFocus()">
```

### Event Firing Multiple Times

**Problem**: Handler executing multiple times unexpectedly

**Solution**: Check for event propagation issues
```javascript
handleClick(event) {
  event.stopPropagation(); // Prevent bubbling to parent handlers
  // Your handler logic
}
```

### iOS Touch Events Not Working

**Problem**: Click events not working on iOS

**Solution**: Use `trigger` and ensure proper CSS
```html
<!-- Add cursor pointer and use trigger on iOS -->
<div class="touchable" 
     style="cursor: pointer;"
     click.trigger="handleTouch()">
  Touch me
</div>
```

## Summary

| Scenario | Recommended Command | Reason |
|----------|-------------------|---------|
| Regular buttons | `delegate` | Efficient, leverages bubbling |
| Form inputs (change, input) | `delegate` | Events bubble naturally |
| Focus/Blur events | `trigger` | Events don't bubble |
| Disabled elements | `trigger` | Delegation issues with disabled elements |
| Complex button content | `trigger` | Prevents event target issues |
| iOS non-input elements | `trigger` | iOS Safari requirements |
| Event interception | `capture` | Need to catch events early |
| Large lists/tables | `delegate` | Memory and performance benefits |

Remember: **Use `delegate` except when you cannot use `delegate`.** This approach will serve you well in most Aurelia applications while maintaining optimal performance and memory usage.