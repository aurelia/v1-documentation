# HTML & SVG attributes

Aurelia supports binding HTML and SVG attributes to JavaScript expressions. Attribute-binding declarations have three parts and take the form `attribute.command="expression"`.

* `attribute`: an HTML or SVG attribute name.
* `command`: one of Aurelia's attribute binding commands:
  * `one-time`: flows data one direction, from the view-model to the view, **once**.
  * `to-view` / `one-way`: flows data one direction, from the view-model to the view.
  * `from-view`: flows data one direction, from the view to the view-model.
  * `two-way`: flows data both ways, from view-model to view and from view to view-model.
  * `bind`: automatically chooses the binding mode. Uses two-way binding for form controls and to-view binding for almost everything else.
* `expression`: a JavaScript expression.

Typically you'll use the `bind` command since it does what you intend most of the time. Consider using `one-time` in performance critical situations where the data never changes because it skips the overhead of observing the view-model for changes. Below are a few examples.

```markup
<input type="text" value.bind="firstName">
<input type="text" value.two-way="lastName">
<input type="text" value.from-view="middleName">

<a class="external-link" href.bind="profile.blogUrl">Blog</a>
<a class="external-link" href.to-view="profile.twitterUrl">Twitter</a>
<a class="external-link" href.one-time="profile.linkedInUrl">LinkedIn</a>
```

In this example, we demonstrate various binding modes:

1. The first input uses `bind`, creating an automatic two-way binding for the value attribute.
2. The second and third inputs explicitly set binding modes with `two-way` and `from-view` respectively.
3. Changes in either the input or the view-model property for the first two inputs will update the other.
4. The third input only updates the view-model when changed; it doesn't reflect view-model changes.
5. The first anchor uses `bind`, which defaults to a to-view binding for href attributes.
6. The other two anchors explicitly use `to-view` and `one-time` binding modes.

These examples showcase Aurelia's flexibility in handling different binding scenarios, allowing you to choose the most appropriate mode for your needs.
