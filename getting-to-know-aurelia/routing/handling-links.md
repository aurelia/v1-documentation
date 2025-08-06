# Handling Links

Aurelia's router automatically intercepts clicks on `<a>` elements to provide smooth single-page application navigation without full page reloads. This is handled by the `DefaultLinkHandler` in the `aurelia-history-browser` package.

## Basic Link Behavior

When you use Aurelia's router, most links are processed by Aurelia instead of triggering traditional browser navigation:

```html
<!-- These links will be handled by Aurelia router -->
<a href="/users">Users</a>
<a href="/products/123">Product Detail</a>
<a href="./relative-path">Relative Link</a>

<!-- Traditional navigation menu -->
<ul>
  <li repeat.for="nav of router.navigation">
    <a href.bind="nav.href" class="${nav.isActive ? 'active' : ''}">${nav.title}</a>
  </li>
</ul>
```

This provides fast navigation between routes without full page reloads, maintaining application state and providing a smoother user experience.

## Automatic Link Interception Rules

The `DefaultLinkHandler` automatically skips click hijacking in the following situations:

### 1. Non-Primary Button Clicks

Only left mouse button clicks (primary button for right-handed users) are intercepted:

```html
<!-- Only left-clicks will be intercepted -->
<a href="/users">Users</a>
```

Right-clicks, middle-clicks, or clicks with mouse buttons other than the primary button will trigger normal browser behavior.

### 2. Modifier Key Combinations

Links clicked while holding modifier keys are not intercepted:

```html
<!-- These will open in new tabs/windows when Ctrl/Cmd+Click -->
<a href="/users">Users (Ctrl+Click to open in new tab)</a>
<a href="/products">Products (Shift+Click for new window)</a>
```

The following modifier keys will skip interception:
- `Alt` key
- `Ctrl` key (Windows/Linux) or `Cmd` key (Mac)
- `Meta` key
- `Shift` key

### 3. External and Hash Links

Links that don't represent internal navigation are skipped:

```html
<!-- External links - not intercepted -->
<a href="https://example.com">External Site</a>
<a href="ftp://files.example.com">FTP Link</a>
<a href="mailto:contact@example.com">Email Link</a>

<!-- Hash links - not intercepted -->
<a href="#section1">Jump to Section 1</a>
<a href="#top">Back to Top</a>
```

### 4. Links with Target Attributes

Links that specify a target other than the current window are not intercepted:

```html
<!-- These will not be intercepted -->
<a href="/users" target="_blank">Users (New Tab)</a>
<a href="/products" target="popup-window">Products (Named Window)</a>

<!-- These WILL be intercepted -->
<a href="/users">Users (Same Window)</a>
<a href="/users" target="_self">Users (Explicit Same Window)</a>
<a href="/users" target="current-window-name">Users (Current Named Window)</a>
```

### 5. Special Attributes

Links with certain attributes are automatically skipped:

```html
<!-- Download links - not intercepted -->
<a href="/files/document.pdf" download>Download PDF</a>
<a href="/files/image.jpg" download="">Download Image</a>

<!-- Explicitly ignored links -->
<a href="/external-page" router-ignore>Skip Router</a>
<a href="/legacy-page" router-ignore="">Skip Router</a>
<a href="/third-party" data-router-ignore>Skip Router</a>
<a href="/old-system" data-router-ignore="">Skip Router</a>
```

## Conditional Link Skipping

You can conditionally skip router interception using data binding with the `data-router-ignore` attribute:

```typescript
// component.ts
export class MyComponent {
  shouldSkipRouter: boolean = false;
  
  toggleRouterHandling() {
    this.shouldSkipRouter = !this.shouldSkipRouter;
  }
}
```

```html
<!-- component.html -->
<template>
  <a href="/some-page" data-router-ignore.bind="shouldSkipRouter || null">
    Conditional Router Skip
  </a>
  
  <button click.delegate="toggleRouterHandling()">
    Toggle Router Handling
  </button>
</template>
```

{% hint style="info" %}
**Important:** Use `|| null` in the binding expression. Aurelia only removes data attributes when the bound value becomes `null` or `undefined`. It doesn't remove the attribute for `false`, `0`, or empty string values.
{% endhint %}

### Why `router-ignore` vs `data-router-ignore`

```html
<!-- This does NOT work with conditional binding -->
<a href="/some-page" router-ignore.bind="condition || null">Does not work</a>

<!-- This WORKS with conditional binding -->
<a href="/some-page" data-router-ignore.bind="condition || null">Works correctly</a>
```

Aurelia's automatic data attribute creation only works for `data-*` and `aria-*` attributes. The `router-ignore` attribute cannot be dynamically added/removed through binding.

## Programmatic Link Handling

You can also control link handling programmatically:

```typescript
import { Router } from 'aurelia-router';

export class MyComponent {
  static inject = [Router];
  
  constructor(private router: Router) {}
  
  handleCustomLink(event: MouseEvent, href: string) {
    // Prevent default browser navigation
    event.preventDefault();
    
    // Use router for navigation
    this.router.navigate(href);
  }
  
  handleExternalLink(event: MouseEvent, url: string) {
    // Allow default browser behavior for external links
    // or handle them specially
    if (this.shouldOpenInNewTab(url)) {
      event.preventDefault();
      window.open(url, '_blank');
    }
  }
  
  private shouldOpenInNewTab(url: string): boolean {
    return url.startsWith('http') && !url.startsWith(window.location.origin);
  }
}
```

```html
<template>
  <a href="/internal-page" click.delegate="handleCustomLink($event, '/internal-page')">
    Custom Handled Internal Link
  </a>
  
  <a href="https://external.com" click.delegate="handleExternalLink($event, 'https://external.com')">
    Custom Handled External Link
  </a>
</template>
```

## Best Practices

### 1. Use Standard HTML Links

Let Aurelia handle links automatically when possible:

```html
<!-- Good - let Aurelia handle automatically -->
<a href="/users/123">User Profile</a>

<!-- Unnecessary - avoid manual handling when not needed -->
<a click.delegate="navigateToUser(123)">User Profile</a>
```

### 2. Provide Visual Feedback for External Links

Make it clear when links will leave the application:

```html
<a href="https://external-site.com" target="_blank">
  External Site <i class="fa fa-external-link"></i>
</a>
```

### 3. Handle Download Links Properly

Always use the `download` attribute for file downloads:

```html
<!-- Correct - will not be intercepted -->
<a href="/api/reports/monthly.pdf" download>Download Monthly Report</a>

<!-- Incorrect - might be intercepted by router -->
<a href="/api/reports/monthly.pdf">Download Monthly Report</a>
```

### 4. Use Router Navigation for Complex Logic

For complex navigation scenarios, use the router directly:

```typescript
export class NavigationComponent {
  constructor(private router: Router) {}
  
  async navigateWithConfirmation(route: string) {
    const confirmed = await this.showConfirmDialog('Navigate away?');
    if (confirmed) {
      this.router.navigate(route);
    }
  }
}
```

### 5. Consider Accessibility

Ensure links work properly for keyboard and screen reader users:

```html
<!-- Good - proper semantic link -->
<a href="/users" role="button" tabindex="0">Users</a>

<!-- Avoid - not accessible -->
<span click.delegate="navigate('/users')">Users</span>
```

## Debugging Link Handling

If links aren't being handled as expected, check:

1. **Console errors** - Look for JavaScript errors that might prevent link interception
2. **Link attributes** - Verify no `target`, `download`, or `router-ignore` attributes are present
3. **URL format** - Ensure links are relative or internal absolute paths
4. **Event propagation** - Check if other event handlers are preventing default behavior

```typescript
// Debug link handling
export class DebugComponent {
  handleLinkClick(event: MouseEvent) {
    console.log('Link clicked:', {
      href: (event.target as HTMLAnchorElement).href,
      ctrlKey: event.ctrlKey,
      shiftKey: event.shiftKey,
      altKey: event.altKey,
      metaKey: event.metaKey,
      button: event.button
    });
  }
}
```

Understanding Aurelia's link handling behavior helps you create intuitive navigation experiences while maintaining control over when and how different types of links are processed.