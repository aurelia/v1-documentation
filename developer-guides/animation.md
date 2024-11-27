---
description: >-
  Animations bring dynamic visual interactions to your Aurelia applications,
  enhancing user experience by smoothly transitioning elements' styles over
  time.
---

# Animation

Animations enhance the interactivity and visual appeal of your Aurelia applications by allowing elements to transition smoothly between different styles, such as changes in size, color, or position. Aurelia's animation system is built around a simple animation interface within its templating library, providing flexibility to use any animation library or custom implementations.

This documentation focuses on using the official CSS animation plugin, `aurelia-animator-css`, for Aurelia. Alternatively, you can opt for other plugins like `aurelia-animator-velocity` or create your own animator based on Aurelia's animation interface.

## Installing the Animation Plugin

To incorporate CSS animations into your Aurelia project, install the `aurelia-animator-css` plugin via NPM:

```bash
npm install aurelia-animator-css
```

{% hint style="info" %}
If you created your application using the Aurelia CLI, the plugin is likely already installed as a dependency.
{% endhint %}

## Configuring the Animation Plugin

After installation, configure the plugin within your application's main configuration file, typically `main.ts` located in the `src` folder.

```typescript
import { PLATFORM } from 'aurelia-pal';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .plugin(PLATFORM.moduleName('aurelia-animator-css')); // Add the animation plugin here

  aurelia.start().then(a => a.setRoot());
}
```

Alternatively, you can configure the plugin to handle animation completion explicitly:

```typescript
.plugin('aurelia-animator-css', cfg => cfg.useAnimationDoneClasses = true)
```

{% hint style="warning" %}
Ensure `PLATFORM.moduleName` is used when configuring the plugin if you're utilizing Webpack.
{% endhint %}

## Defining Animations

Start by defining keyframe animations in your CSS file, such as `animations.css`. These animations describe the transformation from one style to another over a specified duration.

```css
@keyframes SlideInRight {
  from {
    transform: translateX(100%);
  }
  to {
    transform: translateX(0);
  }
}

@keyframes SlideOutRight {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(100%);
  }
}

@keyframes SlideInLeft {
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(0);
  }
}

@keyframes SlideOutLeft {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(-100%);
  }
}

@keyframes FadeIn {
  to {
    opacity: 1;
  }
}

@keyframes FadeOut {
  to {
    opacity: 0;
  }
}
```

Next, create CSS classes that utilize these keyframe animations. These classes follow a naming convention with specific suffixes to integrate with Aurelia's animation lifecycle.

```css
.animate-slide-in-right.au-enter {
  transform: translateX(100%);
}

.animate-slide-in-right.au-enter-active {
  animation: SlideInRight 1s;
}

.animate-slide-out-right.au-leave-active {
  animation: SlideOutRight 1s;
}

.animate-slide-in-left.au-enter {
  transform: translateX(-100%);
}

.animate-slide-in-left.au-enter-active {
  animation: SlideInLeft 1s;
}

.animate-slide-out-left.au-leave-active {
  animation: SlideOutLeft 1s;
}

.animate-fade-in.au-enter {
  opacity: 0;
}

.animate-fade-in.au-enter-active {
  animation: FadeIn 1s;
}

.animate-fade-out.au-leave-active {
  animation: FadeOut 1s;
}

.animate-fade-out.au-left {
  opacity: 0;
}
```

These classes utilize the following lifecycle hooks:

| Method          | Description            | Preparation | Animation Activation | Animation Ended |
| --------------- | ---------------------- | ----------- | -------------------- | --------------- |
| **Enter**       | Element enters the DOM | `au-enter`  | `au-enter-active`    | `au-entered`    |
| **Leave**       | Element leaves the DOM | `au-leave`  | `au-leave-active`    | `au-left`       |
| **addClass**    | Adds a CSS class       |             | `[className]-add`    | `[className]`   |
| **removeClass** | Removes a CSS class    |             | `[className]-remove` |                 |

{% hint style="info" %}
The `au-entered` and `au-left` classes are only added if the `useAnimationDoneClasses` property is configured. The `au-left` class is useful for animating the disappearance of an element to prevent flickering between the removal of the `au-leave-active` class and the DOM node removal.
{% endhint %}

## Applying Animations to Elements

To activate animations on specific elements, apply the `au-animate` class along with the desired animation classes you defined earlier. Aurelia will automatically handle the addition and removal of the appropriate animation classes based on the element's lifecycle events.

### Example: Animating a Repeater

Consider a list of to-do items rendered using Aurelia's `repeat.for` binding. To animate each item as it enters and leaves the DOM, apply the `au-animate` class along with the specific animation classes.

```html
<!-- todos.html -->
<template>
  <ul class="au-stagger">
    <li
      repeat.for="todo of todos"
      class="au-animate animate-fade-in animate-fade-out"
    >
      <input type="checkbox" checked.bind="todo.done">
      <span style="text-decoration: ${todo.done ? 'line-through' : 'none'}">
        ${todo.description}
      </span>
      <button click.trigger="removeTodo(todo)">Remove</button>
    </li>
  </ul>
</template>
```

In the above example:

* The `au-stagger` class on the `<ul>` element delays the animation of each `<li>` to prevent simultaneous animations.
* Each `<li>` has the `au-animate` class along with `animate-fade-in` and `animate-fade-out` to handle entering and leaving animations.

```css
/* animations.css */
.au-stagger {
  animation-delay: 500ms;
}
```

### Example: Animating Router Views

To animate views rendered within a `router-view`, add the `au-animate` class and the desired animation classes to the first child of the view template.

```html
<!-- todos.html -->
<template>
  <div class="au-animate animate-slide-in-right animate-slide-out-left">
    <!-- View content goes here -->
  </div>
</template>
```

Additionally, configure the `router-view` to manage the order of animations between views:

```html
<!-- app.html -->
<router-view swap-order="enter-leave"></router-view>
```

For more details on the `swap-order` attribute and available options, refer to the Router Documentation.

## Using Animation Methods

Aurelia's animator provides methods to programmatically add or remove CSS classes, triggering animations as needed.

#### `addClass`

Adds a specified CSS class to an element, initiating the corresponding animation if defined.

```javascript
export class Example {
  static inject = [Element, Animator];
  
  constructor(element, animator) {
    this.element = element;
    this.animator = animator;
  }
  
  triggerAnimation() {
    this.animator.addClass(this.element, 'animate-custom');
  }
}
```

### `removeClass`

Removes a specified CSS class from an element, triggering the corresponding exit animation if defined.

```javascript
triggerAnimation() {
  this.animator.removeClass(this.element, 'animate-custom');
}
```

{% hint style="info" %}
The `removeClass` method will not trigger an animation if the element does not have either the `[className]` or `[className]-add` CSS class.
{% endhint %}



## Examples

### Animating a Repeater

The following example demonstrates animating a list of to-do items with fade-in and fade-out effects using Aurelia's repeater.

```html
<!-- todos.html -->
<template>
  <ul class="au-stagger">
    <li
      repeat.for="todo of todos"
      class="au-animate animate-fade-in animate-fade-out"
    >
      <input type="checkbox" checked.bind="todo.done">
      <span style="text-decoration: ${todo.done ? 'line-through' : 'none'}">
        ${todo.description}
      </span>
      <button click.trigger="removeTodo(todo)">Remove</button>
    </li>
  </ul>
</template>
```

```css
/* animations.css */
.animate-fade-in.au-enter {
  opacity: 0;
}

.animate-fade-in.au-enter-active {
  animation: FadeIn 1s;
}

.animate-fade-out.au-leave-active {
  animation: FadeOut 1s;
}

.au-stagger {
  animation-delay: 500ms;
}
```

**In this setup:**

* New to-do items fade in as they are added.
* Removed to-do items fade out before being removed from the DOM.
* The `au-stagger` class ensures that multiple items do not animate simultaneously, creating a staggered effect.

### Animating Router Views

Add animation classes to the view templates to animate transitions between different views managed by Aurelia's router.

```html
<!-- todos.html -->
<template>
  <div class="au-animate animate-slide-in-right animate-slide-out-left">
    <!-- View content -->
  </div>
</template>
```

Configure the `router-view` in your application's main template:

```html
<!-- app.html -->
<router-view swap-order="enter-leave"></router-view>
```

This configuration ensures that when navigating between views:

* The new view slides in from the right.
* The old view slides out to the left.
* The `swap-order` attribute manages the timing of entering and leaving animations to create a smooth transition.

## Custom Animations

While the `aurelia-animator-css` plugin provides a convenient way to use CSS animations, Aurelia's animation system is designed to be flexible. You can create custom animation plugins by implementing Aurelia's animation interface, allowing integration with libraries like GreenSock (GSAP) or custom animation logic.

### Creating a Custom Animator

1.  **Implement the Animation Interface:**

    Create a new class that implements the required methods (`enter`, `leave`, `addClass`, `removeClass`).

    ```javascript
    export class CustomAnimator {
      enter(element, anchor, settings) {
        // Custom enter animation logic
      }
      
      leave(element, anchor, settings) {
        // Custom leave animation logic
      }
      
      addClass(element, className, settings) {
        // Custom addClass animation logic
      }
      
      removeClass(element, className, settings) {
        // Custom removeClass animation logic
      }
    }
    ```
2.  **Register the Custom Animator:**

    Configure Aurelia to use your custom animator in `main.ts`.

    ```typescript
    import { PLATFORM } from 'aurelia-pal';
    import { CustomAnimator } from './custom-animator';

    export function configure(aurelia) {
      aurelia.use
        .standardConfiguration()
        .developmentLogging()
        .plugin(PLATFORM.moduleName('your-custom-animator-plugin'));

      aurelia.start().then(a => a.setRoot());
    }
    ```
3.  **Utilize the Custom Animator:**

    Apply the `au-animate` class and your custom animation classes to elements as needed, similar to using the CSS animator.

## Summary

Aurelia's animation system offers a robust and flexible way to enhance your applications with smooth, visually appealing transitions. By leveraging the `aurelia-animator-css` plugin, you can easily implement standard CSS animations by defining keyframes and applying specific classes to your elements. For more advanced scenarios, Aurelia's interface allows the integration of custom animation libraries or the creation of bespoke animation solutions, empowering you to tailor animations precisely to your application's needs.
