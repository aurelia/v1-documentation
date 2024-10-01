# Router pipelines (hooks)

Aurelia's router uses a pipeline concept for processing navigation. You can add your own steps to this pipeline for things like authorization, loading indicators, or analytics tracking.

Steps are added using methods like `addAuthorizeStep()`, `addPreActivateStep()`, `addPreRenderStep()`, or `addPostRenderStep()`.

Here's an example of an authorization step:

```typescript
import { RouterConfiguration, NavigationInstruction, Next, Redirect } from 'aurelia-router';

export class App {
  configureRouter(config: RouterConfiguration): void {
    config.addAuthorizeStep(AuthorizeStep);
    // ... other configuration
  }
}

class AuthorizeStep {
  run(navigationInstruction: NavigationInstruction, next: Next): Promise<any> {
    if (navigationInstruction.getAllInstructions().some(i => i.config.settings.auth)) {
      var isLoggedIn = // ... check if user is logged in
      if (!isLoggedIn) {
        return next.cancel(new Redirect('login'));
      }
    }

    return next();
  }
}
```

In this example, the `AuthorizeStep` checks if any of the routes being activated require authentication (by checking a hypothetical `auth` property in the route's `settings`). If authentication is required and the user isn't logged in, it redirects to a login page.
