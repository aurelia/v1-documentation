# Advanced Techniques

### Handling Async Operations

Playwright provides robust methods for managing asynchronous behaviors:

```javascript
test('handle async operations', async ({ page }) => {
  // Wait for network idle
  await page.waitForLoadState('networkidle');

  // Wait for specific network requests
  await page.waitForRequest('**/api/data');
  await page.waitForResponse('**/api/data');

  // Custom wait with timeout
  await page.waitForFunction(() => {
    return document.querySelector('#loadIndicator').textContent === 'Ready';
  }, { timeout: 5000 });
});
```

### Mocking and Stubbing

#### Network Request Mocking

```javascript
test('mock API responses', async ({ page }) => {
  // Intercept and mock network requests
  await page.route('**/api/users', route => {
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'Test User' }
      ])
    });
  });

  // Navigate and verify mocked data
  await page.goto('/users');
  const userElements = await page.$$('.user-item');
  expect(userElements.length).toBe(1);
});
```

#### Geolocation and Permissions Mocking

```javascript
test('mock geolocation', async ({ context, page }) => {
  // Set geolocation before navigation
  await context.setGeolocation({ 
    latitude: 40.7128, 
    longitude: -74.0060 
  });

  // Mock permission
  await context.grantPermissions(['geolocation']);

  await page.goto('/location-based-app');
});
```

### Performance Testing

#### Page Load Performance

```javascript
test('measure page performance', async ({ page }) => {
  const metrics = await page.evaluate(() => {
    const timing = performance.timing;
    return {
      loadTime: timing.loadEventEnd - timing.navigationStart,
      domInteractive: timing.domInteractive - timing.navigationStart,
      firstContentfulPaint: performance.getEntriesByType('paint')
        .find(e => e.name === 'first-contentful-paint')?.startTime
    };
  });

  expect(metrics.loadTime).toBeLessThan(3000);
});
```

### Advanced Interaction Techniques

#### Complex User Scenarios

```javascript
test('complex user interaction', async ({ page }) => {
  // Simulate drag and drop
  const source = await page.$('#draggable-element');
  const target = await page.$('#drop-target');

  await source.hover();
  await page.mouse.down();
  await target.hover();
  await page.mouse.up();

  // Multi-step interaction with delays
  await page.keyboard.press('Shift');
  await page.type('#search-input', 'complex query');
  await page.keyboard.press('Enter');
});
```

### State Management and Isolation

```javascript
test.describe('Isolated Test Suite', () => {
  test.beforeEach(async ({ context }) => {
    // Create a fresh browser context for each test
    const page = await context.newPage();
    
    // Clear local storage and cookies
    await page.evaluate(() => {
      localStorage.clear();
      sessionStorage.clear();
    });
  });

  test('test with isolated state', async ({ page }) => {
    // Test logic with clean state
  });
});
```

### Accessibility Testing

```javascript
const { test, expect } = require('@playwright/test');
const { injectAxe, checkA11y } = require('axe-playwright');

test('check page accessibility', async ({ page }) => {
  await page.goto('/page');
  await injectAxe(page);
  
  const results = await checkA11y(page);
  expect(results.violations.length).toBe(0);
});
```

### Error Tracking and Screenshots

```javascript
test('capture errors', async ({ page }, testInfo) => {
  try {
    await page.goto('/critical-page');
    // Test logic
  } catch (error) {
    // Automatically capture screenshot on failure
    await testInfo.attach('screenshot', {
      body: await page.screenshot(),
      contentType: 'image/png'
    });
    throw error;
  }
});
```

{% hint style="warning" %}
**Performance Consideration** Advanced techniques like mocking and extensive interactions can slow down test execution. Use judiciously and focus on critical paths.
{% endhint %}

### Best Practices

* Keep tests independent and isolated
* Use page object models for complex interactions
* Minimize hard waits, prefer dynamic waiting
* Mock external dependencies
* Capture meaningful metrics and errors

{% hint style="info" %}
**Pro Tip**: Combine these advanced techniques to create robust, comprehensive E2E test suites that cover complex user scenarios and ensure application reliability.
{% endhint %}
