# Memory management

Efficient memory management is crucial in SSR to prevent memory leaks and ensure optimal performance.

## Avoid Long-Lived References

Ensure that server-side instances of your application do not hold onto references longer than necessary:

### **Dispose of Timers**: Clear any timers (`setTimeout`, `setInterval`) once they're no longer needed.

```javascript
let timer = setTimeout(() => {
  // Task
}, 1000);

// Later in the code
clearTimeout(timer);
```

### **Remove Event Listeners**: Detach any event listeners added during rendering.

```javascript
function onEvent() {
  // Handler
}

element.addEventListener('click', onEvent);

// Later in the code
element.removeEventListener('click', onEvent);
```

## Avoid Modifying Global Prototypes

Never override or extend global prototypes (e.g., `Array.prototype`) as it can lead to memory leaks and unpredictable behavior across different instances.

```javascript
// Bad Practice
Array.prototype.push = function(item) {
  // Custom push logic
};
```

## Use Weak References

When possible, use weak references to allow garbage collection of unused objects.

```javascript
const weakMap = new WeakMap();
let obj = {};
weakMap.set(obj, 'Some value');
// obj can be garbage collected when no longer referenced elsewhere
```
