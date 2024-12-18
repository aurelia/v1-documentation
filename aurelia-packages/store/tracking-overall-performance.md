# Tracking overall performance

The Aurelia Store plugin provides a way to measure the performance of your state updates. This can be helpful for identifying performance bottlenecks in your actions or middleware.

`**measurePerformance` Option:\*\*

You can enable performance tracking using the `measurePerformance` option during plugin registration:

```typescript
// main.ts
aurelia.use.plugin('aurelia-store', {
  initialState,
  measurePerformance: 'all',
});
```

**Values:**

The `measurePerformance` option can have the following values:

* `**'all'`:\*\* Logs the total duration of the dispatch cycle, as well as the duration of each individual middleware function and the action itself.
* `**'startEnd'`:\*\* Logs only the total duration of the dispatch cycle.
* `**undefined` or not specified:\*\* Performance tracking is disabled (default).

**Output:**

Performance measurements are logged to the console using `console.group` and `console.time`/`console.timeEnd`. The output will look something like this (with `measurePerformance: 'all'`):

```javascript
group 'Dispatching: myAction'
  debug 'myMiddleware1' took 1.234 ms
  debug 'myMiddleware2' took 0.567 ms
  debug 'myAction' took 2.345 ms
groupEnd 'Dispatching: myAction'
debug 'Total duration:' 4.146 ms
```

{% hint style="info" %}
Performance measurements are only logged for successful state updates. If an action or middleware interrupts the execution (by returning `false` or throwing an error), no performance information will be logged for that dispatch cycle.
{% endhint %}

