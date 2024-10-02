# Isomorphic code

Isomorphic code refers to application code that runs seamlessly in both client and server environments. To achieve this in Aurelia:

## Avoid Environment-Specific Globals

Instead of directly accessing browser-specific globals like `document` or `window`, use Aurelia's PAL:

```javascript
import { DOM } from 'aurelia-pal';

export class MyComponent {
  activate() {
    const div = DOM.createElement('div');
    DOM.appendChild(DOM.body, div);
  }
}
```

## Platform Abstraction Layer (PAL)

Aurelia's PAL provides a consistent API for DOM manipulation and other platform-specific tasks, ensuring your code remains environment-agnostic.

#### Example

Without PAL (Not Isomorphic):

```javascript
export class MyPage {
  activate() {
    document.body.appendChild(document.createElement('div'));
  }
}
```

With PAL (Isomorphic):

```javascript
import { DOM } from 'aurelia-pal';

export class MyPage {
  activate() {
    DOM.appendChild(DOM.body, DOM.createElement('div'));
  }
}
```

## Best Practices

* **Use PAL for DOM Operations**: Always use `aurelia-pal` for any DOM-related tasks.
*   **Conditional Logic**: If certain code should only run on the client or server, use environment checks to prevent execution in the other environment.

    ```javascript
    import { PLATFORM } from 'aurelia-pal';

    if (PLATFORM.isBrowser) {
      // Code specific to the browser
    }
    ```
