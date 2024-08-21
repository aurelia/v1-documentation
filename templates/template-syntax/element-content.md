# Element content

When manipulating element content in Aurelia, you have several options at your disposal. Let's explore how to bind values to `innerHTML`, `textContent`, and more.

In the previous section on text interpolation, we showed how you can use interpolation to display values, which is the equivalent of binding to the `textContent` property.

## Using textContent Binding

For more control, you can bind directly to `textContent`:

```markup
<p textcontent.bind="message"></p>
```

This is useful when you want to set the entire text content of an element.

## HTML content binding

To bind HTML content, use the `innerHTML` attribute:

```markup
<div innerhtml.bind="htmlContent"></div>
```

## Controlling text direction

```markup
<p textcontent.bind="message" dir.bind="textDirection"></p>
```

In your view-model:

```javascript
export class MyViewModel {
  message = "Hello, Aurelia!";
  textDirection = "rtl"; // or "ltr"
}
```

## Raw HTML Binding

If you need to bind raw HTML and you're sure it's safe, Aurelia provides a `innerhtml` binding command:

```markup
<div innerhtml.bind="trustedHtmlContent | sanitizeHTML"></div>
```

{% hint style="danger" %}
Always use HTML sanitization. We provide a simple converter as a placeholder. However, it does NOT provide security against a wide variety of sophisticated XSS attacks and should not be relied upon for sanitizing input from unknown sources. You can replace the built-in sanitizer by registering your own implementation of [HTMLSanitizer](https://github.com/aurelia/templating-resources/blob/master/src/html-sanitizer.js) with the app at startup. For example, `aurelia.use.singleton(HTMLSanitizer, BetterHTMLSanitizer);` We recommend using a library such as DOMPurify or sanitize-html for your implementation.
{% endhint %}
