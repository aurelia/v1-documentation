# Internationalizing Titles

Aurelia allows you to internationalize your route titles using the `transformTitle` property of the router.

```typescript
import { I18N } from 'aurelia-i18n';

export class App {
  static inject = [I18N];

  constructor(private i18n: I18N) {}

  configureRouter(config, router: Router) {
    config.title = 'My App';
    config.map([
      { route: ['', 'home'], name: 'home', moduleId: PLATFORM.moduleName('home/index'), title: 'Home' },
      { route: 'users', name: 'users', moduleId: PLATFORM.moduleName('users/index'), title: 'Users' }
    ]);

    router.transformTitle = (title) => this.i18n.tr(title);
  }
}
```

In this example, `transformTitle` uses the `tr` (translate) function from `aurelia-i18n` to translate the title.
