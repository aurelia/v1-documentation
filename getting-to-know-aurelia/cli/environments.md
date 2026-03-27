# Environments

The Aurelia CLI supports multiple environments to manage different development, staging, and production configurations. By default, three environments are set up: `dev`, `stage`, and `prod`.

To run or build your app in a specific environment, use the `--env` flag:

```bash
au run --env prod
```

When you specify an environment, the CLI replaces `src/environment.js` or `src/environment.ts` with the corresponding file (`dev.js`, `stage.js`, or `prod.js`) from the `aurelia_project/environments` directory.

## Environment File Structure

Your project contains environment files at:

```
aurelia_project/
  environments/
    dev.js        (or dev.ts)
    stage.js      (or stage.ts)
    prod.js       (or prod.ts)
```

Each file exports a configuration object:

```javascript
// aurelia_project/environments/dev.js
export default {
  debug: true,
  testing: true,
  apiUrl: 'http://localhost:3000/api'
};
```

```javascript
// aurelia_project/environments/prod.js
export default {
  debug: false,
  testing: false,
  apiUrl: 'https://api.myapp.com'
};
```

## Using Environment Values

Import the environment configuration anywhere in your application:

```javascript
import environment from './environment';

export class ApiService {
  constructor() {
    this.baseUrl = environment.apiUrl;
  }
}
```

You can also use it during app startup to conditionally enable features:

```javascript
import environment from './environment';

export function configure(aurelia) {
  aurelia.use.standardConfiguration();

  if (environment.debug) {
    aurelia.use.developmentLogging();
  }

  aurelia.start().then(() => aurelia.setRoot());
}
```

## Creating Custom Environments

You can create additional environments by adding new files to the `aurelia_project/environments` directory. For example, to add a `qa` environment:

1. Create `aurelia_project/environments/qa.js`
2. Run or build with `--env qa`:

```bash
au run --env qa
au build --env qa
```

## Default Environment

When no `--env` flag is provided, the CLI uses the `dev` environment by default:

```bash
au run          # Uses dev environment
au build        # Uses dev environment
```
