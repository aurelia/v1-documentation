# Text interpolation

Text interpolation allows you to display values within your template views. By leveraging `${}`, a dollar sign followed by opening and closing curly braces, you can display values inside your views. The syntax will be familiar to you if you are familiar with [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template\_literals).

## Basic usage

To use text interpolation, wrap your expression in curly braces `${}`:

```markup
<p>Hello, ${name}!</p>
```

In this example, `name` is a property in your view-model. Aurelia will automatically update the text whenever `name` changes.

### Working with Expressions

Aurelia 1 supports basic JavaScript expressions within interpolation:

```markup
<p>The answer is ${2 + 2}.</p>
<p>Your balance: $${balance * exchangeRate}</p>
```

You'll need to use value converters for more complex operations like string manipulation.

## Tips

1. Keep your expressions simple. Move complex logic to your view model or use value converters.
2. For better performance with large lists, consider using the `textContent` attribute:
3. Combine multiple value converters for complex transformations:
