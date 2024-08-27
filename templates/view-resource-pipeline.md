# View Resource pipeline

The basic idea behind the View Resource Pipeline is that we're not limited to HTML or JavaScript. A basic example would be pulling in Bootstrap:

{% code title="hello.html" %}
```markup
<template>
  <require from="bootstrap/css/bootstrap.min.css"></require>
  <p class="lead">Hello, World!</p>
  <p>Lorem Ipsum...</p>
</template>
```
{% endcode %}

The `<require>` tag isn't limited to HTML or view models; it can also handle CSS files. Aurelia's View Resource Pipeline intelligently recognizes and processes these different resource types. What's truly powerful is that you can extend this pipeline, allowing you to create custom handlers for any view resource you might need. This flexibility is one of Aurelia's standout features.

