---
description: >-
  Slotted content allows you to build flexible and reusable components by
  enabling users to insert their own content into specified areas of your
  component.
---

# Slotted content

Aurelia supports slotted content through a feature known as `content projection`. By defining slots in your components, you can make parts of the component's template available for external content insertion. This allows the component to dictate structure and style, while consumers can customize specific content pieces.

## Defining Slots in Components

To define a slot within an Aurelia component, use the `<slot>` element in your component's template file. Here’s how you can create a basic slotted component:

### Example

**HTML Template (`my-component.html`):**

```markup
<template>
  <header>
    <slot name="header">Default Header</slot>
  </header>
  
  <main>
    <slot>Default Content</slot>
  </main>
  
  <footer>
    <slot name="footer">Default Footer</slot>
  </footer>
</template>
```

In this example, `my-component` contains three slots: a named "header" slot, an unnamed default slot, and a named "footer" slot.

## Using Slots in Parent Components

To utilize the slots defined in a component, simply include the component in the parent view and provide elements with the `slot` attribute:

**Parent Template (`app.html`):**

```markup
<template>
  <my-component>
    <div slot="header">Custom Header Content</div>
    
    <div>Custom Main Content</div>
    
    <div slot="footer">Custom Footer Content</div>
  </my-component>
</template>
```

In this example, the content of the `div` elements is projected into the respective slots of `my-component`.

## Default Slot Content

If no content is provided for a slot, the default content declared within the slot tag in the component's template will be used. This ensures that components remain functional, even when the parent component supplies no content.

### Example

If the parent component renders `my-component` without providing any content:

**Parent Template Without Slots Provided:**

```markup
<template>
  <my-component></my-component>
</template>
```

The output will be:

```bash
Default Header
Default Content
Default Footer
```

## Multiple Slot Projections

A component can support multiple named slots, allowing for even more granular control over content projection:

**Expanded Component Template (multiple-slot-component.html):**

```markup
<template>
  <div class="title">
    <slot name="title">Default Title</slot>
  </div>
  
  <div class="body">
    <slot name="body">Default Body</slot>
  </div>
  
  <div class="actions">
    <slot name="actions">Default Actions</slot>
  </div>
</template>
```

**Parent Template Using Multiple Slots:**

```markup
<template>
  <multiple-slot-component>
    <h1 slot="title">Custom Title</h1>
    
    <p slot="body">Custom Body Content with detailed information.</p>
    
    <button slot="actions">Custom Action Button</button>
  </multiple-slot-component>
</template>
```

Using slots in Aurelia components enables developers to create modular, reusable, and flexible UI components. By defining slots, you specify areas for external content while maintaining a default structure. This allows you to build applications more efficiently with a clear separation of component structure and content.
