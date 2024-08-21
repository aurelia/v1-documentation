# View and Compilation Spies

If you've installed the `aurelia-testing` plugin, you have access to two special templating behaviors:

{% code title="hello.html" %}
```markup
<template>
  <p view-spy compile-spy>Hello!</p>
</template>
```
{% endcode %}

`view-spy` drops Aurelia's copy of the View object into the console while `compile-spy` emits the compiler's TargetInstruction. This is especially useful for debugging any new View Resources you've created using the View Resource Pipeline.
