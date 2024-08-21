# Contextual properties

Context properties in Aurelia 1 are special variables that allow you to access data and functionality from parent or ancestor components in your view templates. They're incredibly useful for navigating the component hierarchy without explicitly passing data through multiple levels.

## Common context properties

### $this

Represents the current binding context (view-model).

```markup
<button click.delegate="$this.doSomething()">Click me</button>
```

### $parent

Accesses the outer scope from within a compose or repeat template. It's chainable for accessing higher-level scopes.

```markup
<p>${$parent.someProperty}</p>
```

### $event

Provides access to the DOM Event in delegate or trigger bindings.

```markup
<button click.delegate="handleClick($event)">Click me</button>
```

## Properties for repeat.for

The following properties are available within a `repeat.for` template:

**$index**

The index of the current item in the collection.

**$first**

True if the current item is the first in the array.

**$last**

True if the current item is the last in the array.

**$even**

True if the current item has an even-numbered index.

**$odd**

True if the current item has an odd-numbered index.

Example usage:

```markup
<ul>
  <li repeat.for="item of items">
    ${$index}: ${item}
    <span if.bind="$first">(First item!)</span>
    <span if.bind="$last">(Last item!)</span>
    <span if.bind="$even">(Even index)</span>
    <span if.bind="$odd">(Odd index)</span>
  </li>
</ul>
```

## Best practices

1. Don't overuse `$parent`. It can make your code harder to maintain. Consider using proper data binding or a state management solution for complex scenarios.
2. Remember, `$parent` in a repeater (`repeat.for`) refers to the repeater's parent, not the list's previous item.
3. You can chain `$parent` like `$parent.$parent` Use this sparingly to go up multiple levels as it creates tight coupling.
4. `$event` is great for handling DOM events, but remember to keep complex logic in your view-model.
