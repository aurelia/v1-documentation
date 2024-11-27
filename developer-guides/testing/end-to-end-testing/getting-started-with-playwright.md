# Getting started with Playwright

### Installation

To begin E2E testing with Playwright in an Aurelia 1 project, you'll need to install the necessary dependencies:

```bash
# Install Playwright
npm install --save-dev @playwright/test

# Install browser drivers
npx playwright install
```

### Project Setup

#### 1. Configuration File

Create a `playwright.config.js` in your project root:

```javascript
const { defineConfig } = require('@playwright/test');

module.exports = defineConfig({
  // Test directory
  testDir: './tests/e2e',
  
  // Browser configurations
  use: {
    baseURL: 'http://localhost:9000', // Typical Aurelia dev server
    trace: 'on-first-retry',
    screenshot: 'only-on-failure'
  },
  
  // Optional: Run in headless or headed mode
  headless: true,
  
  // Reporters
  reporter: [
    ['html', { open: 'never' }],
    ['line']
  ],
  
  // Timeout settings
  timeout: 30000,
  expect: {
    timeout: 5000
  }
});
```

#### 2. Project Structure

Recommended E2E test structure:

```
project-root/
│
├── tests/
│   └── e2e/
│       ├── page-objects/
│       │   ├── login.page.js
│       │   └── dashboard.page.js
│       ├── specs/
│       │   ├── login.spec.js
│       │   └── dashboard.spec.js
│       └── helpers/
│           └── test-utils.js
```

### Basic Concepts

#### Writing Your First Test

```javascript
// tests/e2e/specs/login.spec.js
const { test, expect } = require('@playwright/test');
const { LoginPage } = require('../page-objects/login.page');

test('successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  
  await loginPage.navigate();
  await loginPage.login('username', 'password');
  
  // Assert successful login
  expect(await loginPage.isLoggedIn()).toBeTruthy();
});
```

### Running Tests

Add scripts to your `package.json`:

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug"
  }
}
```

### Aurelia-Specific Considerations

{% hint style="warning" %}
**Aurelia Routing Compatibility** Ensure your Aurelia application's routing is fully loaded before interacting with elements. You might need to add wait conditions specific to Aurelia's initialization.
{% endhint %}

### Browser Support

Playwright supports multiple browsers out of the box:

* Chromium
* Firefox
* WebKit

Configure in `playwright.config.js`:

```javascript
module.exports = defineConfig({
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
    { name: 'firefox', use: { browserName: 'firefox' } },
    { name: 'webkit', use: { browserName: 'webkit' } }
  ]
});
```

{% hint style="info" %}
**Best Practice**: Start with Chromium for development, then expand to other browsers for comprehensive testing.
{% endhint %}

### Recommended Next Steps

1. Set up your initial configuration
2. Create page object classes
3. Write your first test scenarios
4. Integrate with your CI/CD pipeline
