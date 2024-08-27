---
description: >-
  Aurelia's repeat.for feature is your go-to tool for handling arrays and maps
  efficiently. It's like JavaScript's for loop, but tailored for the view.
---

# List rendering (repeaters)

Aurelia excels at working with diverse data structures, making your development process smoother and more flexible. When it comes to collections, Aurelia supports iteration over Arrays, Sets, and Maps directly in your templates. This means you're not limited to just one type of data structure – you can choose the most appropriate one for your specific use case.

Arrays are great for ordered lists, Sets are perfect for unique values, and Maps offer key-value pairs. Whatever your data needs, Aurelia's template system adapts effortlessly. This versatility empowers you to write cleaner, more efficient code while keeping your templates intuitive and easy to maintain.

## Arrays

Aurelia excels at rendering arrays by repeating elements for each item. This powerful feature simplifies the creation of dynamic lists and data-driven interfaces.

{% code title="repeater-template.html" %}
```markup
<template>
  <p repeat.for="friend of friends">Hello, ${friend}!</p>
</template>
```
{% endcode %}

{% code title="repeater-template.js" %}
```javascript
export class RepeaterTemplate {
  constructor() {
    this.friends = [
      'Alice',
      'Bob',
      'Carol',
      'Dana'
    ];
  }
}
```
{% endcode %}

We can use this approach to greet our friends individually instead of attempting a global greeting. It's more personal and efficient.

Remember, we can also use the template element for repetition, but it needs to be wrapped in a surrogate `<template>` tag:

{% code title="repeater-template.html" %}
```markup
<template>
  <template repeat.for="friend of friends">
    <p>Hello, ${friend}!</p>
  </template>
</template>
```
{% endcode %}

{% hint style="info" %}
Aurelia will not be able to observe changes to arrays using the `array[index] = value` syntax. To ensure that Aurelia can observe the changes on your array, make use of the Array methods: `Array.prototype.push`, `Array.prototype.pop` and `Array.prototype.splice`.
{% endhint %}

When working with arrays in two-way binding scenarios, be aware that the standard for-of loop syntax won't cut it. Simply using `repeat.for="item of dataArray"` will leave you with one-way binding, meaning any changes made in input fields won't be reflected in your data.

Instead, use this syntax to achieve proper two-way binding with arrays:

{% code title="repeater-template-input.html" %}
```html
<template>
  <div repeat.for="i of dataArray.length">
  <input type="text" value.bind="$parent.dataArray[i]">
  </div>
</template>
```
{% endcode %}

## Range

We can also iterate over a numerical range:

{% code title="repeater-template.html" %}
```markup
<template>
  <p repeat.for="i of 10">${10-i}</p>
  <p>Blast off!</p>
</template>
```
{% endcode %}

Note that the range will start at 0 with a length of 10, so our countdown really does start at 10 and end at 1 before blast off.

## Set

We can also use an ES6 Set in the same way:

{% code title="repeater-template.html" %}
```markup
<template>
  <p repeat.for="friend of friends">Hello, ${friend}!</p>
</template>
```
{% endcode %}

{% code title="repeater-template.js" %}
```javascript
export class RepeaterTemplate {
  constructor() {
    this.friends = new Set();
    this.friends.add('Alice');
    this.friends.add('Bob');
    this.friends.add('Carol');
    this.friends.add('Dana');
  }
}
```
{% endcode %}

We can use repeaters with arrays, which is useful - but we can also use repeaters with other iterable data types, including objects, plus new ES6 standards such as Map and Set.

## Map

Maps are particularly handy iterables, as they allow you to directly decompose keys and values into separate variables within the repeater. While you can iterate over objects easily, Maps offer smoother two-way binding. For this reason, it's often better to opt for Maps when possible.

{% code title="repeater-template.html" %}
```markup
<template>
  <p repeat.for="[greeting, friend] of friends">${greeting}, ${friend.name}!</p>
</template>
```
{% endcode %}

{% code title="repeater-template.js" %}
```javascript
export class RepeaterTemplate {
  constructor() {
    this.friends = new Map();
    this.friends.set('Hello',
      { name : 'Alice' });
    this.friends.set('Hola',
      { name : 'Bob' });
    this.friends.set('Ni Hao',
      { name : 'Carol' });
    this.friends.set('Molo',
      { name : 'Dana' });
  }
}
```
{% endcode %}

Notice the `[greeting, friend]` syntax in the example. This uses the dereference operator to split the map's key-value pair. 'greeting' becomes the key, while 'friend' represents the value. Since each value is an object with a 'name' property, we can easily access it using `${friend.name}`, just like in regular JavaScript.

## Objects

Let's do the same thing, except with a traditional JavaScript object in our view-model:

{% code title="repeater-template.html" %}
```markup
<template>
    <p repeat.for="greeting of friends | keys">${greeting}, ${friends[greeting].name}!</p>
</template>
```
{% endcode %}

{% code title="repeater-template.js" %}
```javascript
export class RepeaterTemplate {
  constructor() {
    this.friends = {
      'Hello':
        { name : 'Alice' },
      'Hola':
        { name : 'Bob' },
      'Ni Hao':
        { name : 'Carol' },
      'Molo':
        { name : 'Dana' }
    }
  }
}

export class KeysValueConverter {
  toView(obj) {
    return Reflect.ownKeys(obj);
  }
}
```
{% endcode %}

Value converters are a powerful feature in Aurelia. In this example, we're using a 'keys' value converter to transform our 'friends' object into an iterable array of keys. Aurelia looks for a `KeysValueConverter` class and calls its `toView()` method with our 'friends' object. This method returns an array of keys that we can then iterate over, providing a workaround for iterating over Objects.

You might wonder why we can't use \[key, value] dereferencing like we do with Maps. The reason is that JavaScript objects aren't inherently iterable like Arrays, Maps, or Sets. To iterate over objects, we need to transform them into an iterable format. The approach varies depending on your specific needs. For those who prefer the \[key, value] syntax, there's a plugin available that converts Objects into iterable maps, enabling this functionality.
