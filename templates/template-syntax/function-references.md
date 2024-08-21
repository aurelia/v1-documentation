# Function references

In Aurelia, you can use the `.call` binding command to invoke functions directly from your view. This is particularly useful when you need to pass specific arguments or control the context in which the function is called.

```markup
<element click.call="myFunction(arg1, arg2)"></element>
```

## How It Works

When you use `.call`, Aurelia will:

1. Evaluate the expression in the context of your view-model
2. Invoke the resulting function with the provided arguments
3. Set the function's `this` context to the view-model

## Examples

### Basic usage

```markup
<button click.call="greet('World')">Say Hello</button>
```

```javascript
export class MyViewModel {
  greet(name) {
    alert(`Hello, ${name}!`);
  }
}
```

### Passing Event Object

You can pass the event object using `$event`:

```markup
<input keyup.call="handleKeyUp($event)">
```

```javascript
export class MyViewModel {
  handleKeyUp(event) {
    console.log('Key pressed:', event.key);
  }
}
```

### Tips

* Use `.call` when you need to pass specific arguments to your function.
* It's great for one-time function calls that don't need to update bindings.
* Remember, `.call` doesn't create a persistent binding like `.bind` does.
