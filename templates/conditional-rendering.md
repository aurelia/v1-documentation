# Conditionals

Aurelia offers two key methods for conditional display: if and show. While both control element visibility, they operate differently:

• if: Completely removes the element from the DOM\
• show: Toggles the aurelia-hide CSS class, affecting visibility

Choosing between if and show can impact performance and user experience:

• if is more resource-intensive for frequent state changes, as it repeatedly removes and adds elements to the DOM\
• show is generally faster but may be less suitable for large templates with numerous data-bound elements

Consider your specific use case when deciding which method to use. Here's a simple example that demonstrates conditional display:

{% code title="if-template.html" %}
```markup
<template>
  <label for="greet">Would you like me to greet the world?</label>
  <input type="checkbox" id="greet" checked.bind="greet">
  <div if.bind="greet">
    Hello, World!
  </div>
</template>
```
{% endcode %}

However, if we just want to hide the element from view instead of removing it from the DOM completely, we should use `show` instead of `if`.

{% code title="show-template" %}
```markup
<template>
  <label for="greet">Would you like me to greet the world?</label>
  <input type="checkbox" id="greet" checked.bind="greet">
  <div show.bind="greet">
    Hello, World!
  </div>
</template>
```
{% endcode %}

By default, the "Hello World" div gets an `aurelia-hide` class when unchecked, setting display: none if you're using Aurelia's CSS libraries. Don't like that? No problem. You can create custom CSS rules for aurelia-hide, such as visibility: none or height: 0px.

Want to lock in conditional elements when they're first created? Use one-time binding for your conditionals:

{% code title="conditional-one-time-template.html" %}
```markup
<template>
  <div if.one-time="greet">
    Hello, World!
  </div>
  <div if.one-time="!greet">
    Some other time.
  </div>
</template>
```
{% endcode %}

{% code title="bind-template.js" %}
```javascript
export class ConditionalOneTimeTemplate {
  greet = (Math.random() > 0.5);
}
```
{% endcode %}

When the page loads, there's an equal chance of greeting the world or deferring it. This state becomes fixed due to one-time data binding. While 'show.one-time' might seem logical, it's not ideal here. It would apply a static CSS class to hide an element, which is inefficient. In most scenarios, 'if' is preferable as it prevents the creation of unused elements altogether.

The 'else' directive complements 'if', rendering content when the 'if' condition is false.

{% code title="if-else-template.html" %}
```markup
<template>
  <div if.bind="showMessage">
    <span>${message}</span>
  </div>
  <div else>
    <span>Nothing to see here</span>
  </div>
</template>
```
{% endcode %}

Elements using the `else` template modifier must follow an `if` bound element to make contextual sense and function properly.

### Caching the view instance when condition changes

By default, the 'if' directive caches view model instances. Even when an element is removed from the DOM and undergoes detached and unbind lifecycle events, its instance remains in memory. This allows for quick re-rendering when the element is shown again, without reinitializing the component.

To disable caching, explicitly set the 'cache' binding on the 'if' directive. This is particularly useful for custom elements that require fresh initialization each time they appear.

{% code title="if-template-without-cache.html" %}
```markup
<template>
  <div if="condition.bind: showMessage; cache: false">
    <span>${message}</span>
  </div>
</template>
```
{% endcode %}

