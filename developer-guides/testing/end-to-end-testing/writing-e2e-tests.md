# Writing E2E tests

### Test Structure

Playwright tests follow a consistent, easy-to-understand structure:

```javascript
const { test, expect } = require('@playwright/test');

test('description of test scenario', async ({ page }) => {
  // Test implementation
});

// Group related tests
test.describe('Feature Group', () => {
  test('specific test case', async ({ page }) => {
    // Test implementation
  });
});
```

### Locating Elements

Playwright provides multiple strategies for element selection:

```javascript
// By CSS Selector
await page.locator('.login-button');

// By Text
await page.getByText('Submit');

// By Role
await page.getByRole('button', { name: 'Login' });

// By Test ID (Recommended for stability)
await page.getByTestId('login-submit');
```

#### Best Practices for Element Locators

{% hint style="info" %}
* Prefer data-testid attributes for most reliable selection
* Avoid using complex CSS selectors or brittle locators
{% endhint %}

### Interactions

Common user interactions:

```javascript
// Clicking
await page.click('.submit-button');

// Typing
await page.fill('#username', 'testuser');

// Handling dropdowns
await page.selectOption('select#role', 'admin');

// Checkbox and radio buttons
await page.check('#agree-terms');

// Hover and complex interactions
await page.hover('.dropdown-menu');
```

### Assertions

Comprehensive assertion capabilities:

```javascript
// Basic existence
await expect(page.locator('.error-message')).toBeVisible();

// Text content
await expect(page.locator('.welcome-message'))
  .toHaveText('Welcome, John Doe');

// Attribute checks
await expect(page.locator('button'))
  .toHaveAttribute('disabled');

// Multiple assertions
test('login workflow', async ({ page }) => {
  await expect(page).toHaveTitle('Login Page');
  await expect(page.locator('.login-form')).toBeVisible();
});
```

### Page Object Model

Recommended pattern for organizing tests:

```javascript
// page-objects/login.page.js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('.login-button');
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
    await this.page.waitForNavigation();
  }

  async isLoggedIn() {
    return await this.page.locator('.user-dashboard').isVisible();
  }
}

// test implementation
test('successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.login('validuser', 'validpassword');
  expect(await loginPage.isLoggedIn()).toBeTruthy();
});
```

### Handling Async Operations

```javascript
test('async data loading', async ({ page }) => {
  // Wait for network request
  await page.waitForResponse('**/api/data');

  // Wait for element to appear
  await page.waitForSelector('.data-loaded');

  // Custom wait with timeout
  await page.waitForFunction(() => 
    document.querySelector('.loading-indicator') === null
  );
});
```

### Common Scenarios

```javascript
test.describe('Authentication Workflows', () => {
  // Login scenarios
  test('successful login', async ({ page }) => { /* ... */ });
  test('failed login - invalid credentials', async ({ page }) => { /* ... */ });

  // Form interactions
  test('form validation', async ({ page }) => { /* ... */ });

  // Navigation
  test('navigation between pages', async ({ page }) => { /* ... */ });
});
```

{% hint style="warning" %}
**Important**: Always handle potential async operations and unexpected behaviors in your tests.
{% endhint %}
