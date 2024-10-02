# Different entry points

Managing different configurations for client and server requires separate entry points:

## Client Entry Point

Typically named `main.js` or `main.ts`, this file initializes Aurelia for the client-side application.

```javascript
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from 'aurelia-pal';

export function configure(aurelia: Aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  aurelia.start().then(() => aurelia.setRoot(PLATFORM.moduleName('app')));
}
```

## Server Entry Point

Create a separate entry point, such as `server-main.js` or `server-main.ts`, tailored for server-side rendering.

```javascript
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from 'aurelia-pal';
import bootstrapper from './ssr-bootstrapper-webpack';

async function configure(aurelia: Aurelia) {
  aurelia.use
    .standardConfiguration();

  await aurelia.start();
  await aurelia.setRoot(PLATFORM.moduleName('app'));
}

module.exports = bootstrapper(configure);
```

## Webpack Configuration

Configure Webpack to recognize and build client and server bundles with their respective entry points.

```json
"entry": {
  "client": "./src/main",
  "server": "./src/server-main"
},
```

Ensure each bundle is optimized for its target environment, appropriately handling dependencies and optimisations.
