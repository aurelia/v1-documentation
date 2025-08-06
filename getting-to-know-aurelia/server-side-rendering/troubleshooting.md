# Troubleshooting

This guide covers common issues you may encounter when implementing SSR with Aurelia and their solutions.

## Common Setup Issues

### Build Errors

#### Error: "Cannot resolve 'aurelia-pal'"

**Symptoms:**
```
Module not found: Error: Can't resolve 'aurelia-pal' in '/path/to/project'
```

**Solution:**
Ensure you have installed the Node.js PAL package:

```bash
npm install aurelia-pal-nodejs
```

And initialize it in your server entry point:

```typescript
import { initialize } from 'aurelia-pal-nodejs';
initialize();
```

#### Error: "aurelia-bootstrapper not found"

**Symptoms:**
```
Module not found: Error: Can't resolve 'aurelia-bootstrapper'
```

**Solution:**
This usually happens when webpack is trying to load the client bootstrapper on the server. Ensure your server configuration uses the correct entry point:

```javascript
// Server webpack config
{
  entry: './src/server-main.ts', // Not aurelia-bootstrapper
  target: 'node'
}
```

### Runtime Errors

#### Error: "window is not defined"

**Symptoms:**
```
ReferenceError: window is not defined
```

**Cause:** Your code is trying to access browser globals in the server environment.

**Solution:**
1. Use environment checks:

```javascript
import { PLATFORM } from 'aurelia-pal';

if (PLATFORM.isBrowser) {
  // Browser-only code
  window.analytics.track('page_view');
}
```

2. Use PAL abstractions:

```javascript
import { DOM } from 'aurelia-pal';

// Instead of: window.location.href
const currentUrl = DOM.location.href;
```

#### Error: "document is not defined"

**Symptoms:**
```
ReferenceError: document is not defined
```

**Solution:**
Replace direct DOM access with PAL:

```javascript
import { DOM } from 'aurelia-pal';

// Instead of: document.getElementById('myElement')
const element = DOM.getElementById('myElement');

// Instead of: document.createElement('div')
const div = DOM.createElement('div');
```

#### Error: "localStorage is not defined"

**Symptoms:**
```
ReferenceError: localStorage is not defined
```

**Solution:**
Add environment checks or provide server-side alternatives:

```javascript
import { PLATFORM } from 'aurelia-pal';

export class StorageService {
  get(key: string): string | null {
    if (PLATFORM.isBrowser && window.localStorage) {
      return localStorage.getItem(key);
    }
    // Return null or implement server-side storage
    return null;
  }

  set(key: string, value: string): void {
    if (PLATFORM.isBrowser && window.localStorage) {
      localStorage.setItem(key, value);
    }
    // Implement server-side storage if needed
  }
}
```

## Memory and Performance Issues

### Memory Leaks

#### Symptoms:
- Server memory usage continuously increases
- Server becomes unresponsive over time
- "Out of memory" errors

#### Debugging Memory Leaks:

1. **Monitor memory usage:**

```javascript
// Add to your server code
setInterval(() => {
  const used = process.memoryUsage();
  console.log('Memory usage:', {
    rss: Math.round(used.rss / 1024 / 1024 * 100) / 100,
    heapTotal: Math.round(used.heapTotal / 1024 / 1024 * 100) / 100,
    heapUsed: Math.round(used.heapUsed / 1024 / 1024 * 100) / 100,
    external: Math.round(used.external / 1024 / 1024 * 100) / 100
  });
}, 10000); // Every 10 seconds
```

2. **Check for common causes:**

```javascript
// Bad: Timer not cleared
let timer = setInterval(() => {}, 1000);
// Good: Clear timer in unbind/detached
clearInterval(timer);

// Bad: Event listener not removed
element.addEventListener('click', handler);
// Good: Remove in unbind/detached
element.removeEventListener('click', handler);

// Bad: Circular references
obj1.ref = obj2;
obj2.ref = obj1;
// Good: Use WeakMap or break references
```

### Slow Rendering Performance

#### Symptoms:
- Server response times are slow
- High CPU usage during rendering

#### Solutions:

1. **Profile your rendering:**

```javascript
// Add timing to your middleware
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const duration = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${duration}ms`);
});
```

2. **Optimize heavy operations:**

```javascript
// Move expensive operations to background
export class ExpensiveComponent {
  bind() {
    // Don't do expensive work in bind for SSR
    if (PLATFORM.isBrowser) {
      this.loadExpensiveData();
    }
  }
  
  attached() {
    // This only runs on client
    this.loadExpensiveData();
  }
}
```

## Rendering Issues

### Content Not Appearing

#### Symptoms:
- Page renders but content is missing
- "Loading..." or placeholder content shows

#### Solutions:

1. **Check async operations:**

```javascript
// Bad: Async operation in bind without waiting
bind() {
  this.loadData(); // Promise not awaited
}

// Good: Return promise or use attached
async bind() {
  await this.loadData();
}

// Or defer to attached for client-only data
attached() {
  this.loadData();
}
```

2. **Verify template syntax:**

```html
<!-- Ensure interpolations are correctly formatted -->
<div>${data.property}</div>

<!-- Check for typos in binding expressions -->
<div textcontent.bind="message"></div>
```

### Hydration Mismatches

#### Symptoms:
- Content flickers when client takes over
- Console warnings about hydration mismatches

#### Solutions:

1. **Ensure consistent rendering:**

```javascript
// Bad: Different content on server vs client
get currentTime() {
  return new Date().toISOString(); // Always different
}

// Good: Use static or consistent data for SSR
get displayTime() {
  if (PLATFORM.isBrowser) {
    return new Date().toISOString();
  }
  return this.serverTime; // Set during SSR
}
```

2. **Use proper conditional rendering:**

```html
<!-- Bad: Conditional based on browser-only features -->
<div if.bind="window.innerWidth > 768">Desktop content</div>

<!-- Good: Use consistent conditions -->
<div if.bind="isDesktop">Desktop content</div>
```

## Debugging Techniques

### Enable Debug Logging

Add detailed logging to understand the rendering process:

```javascript
// In your server-main.ts
if (process.env.NODE_ENV === 'development') {
  aurelia.use.developmentLogging('debug');
}
```

### Use Node.js Debugger

1. **Start with debugging enabled:**

```bash
node --inspect-brk server/index.js
```

2. **Open Chrome DevTools:**
   - Navigate to `chrome://inspect`
   - Click "Open dedicated DevTools for Node"

### Server-Side Console Output

Add strategic console.log statements:

```javascript
export class MyComponent {
  bind() {
    console.log('SSR: MyComponent binding with data:', this.data);
  }
  
  attached() {
    console.log('Client: MyComponent attached');
  }
}
```

### Error Boundary Implementation

Create error boundaries to catch and handle SSR errors gracefully:

```javascript
// In your middleware configuration
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  onError: (error, request) => {
    console.error('SSR Error:', error);
    console.error('Request:', request.url);
    
    // Return fallback content
    return `
      <div class="ssr-error">
        <h1>Content Unavailable</h1>
        <p>Please enable JavaScript to view this page.</p>
      </div>
    `;
  }
}));
```

## Configuration Issues

### Webpack Bundle Problems

#### Symptoms:
- "Module not found" errors in production
- Different behavior between development and production

#### Solutions:

1. **Check externals configuration:**

```javascript
// Server webpack config
externals: [nodeExternals({
  // Whitelist client-only modules that should be bundled
  whitelist: ['aurelia-*']
})]
```

2. **Verify output configuration:**

```javascript
// Ensure proper output format for Node.js
output: {
  libraryTarget: 'commonjs2', // Important for server bundles
  path: path.resolve(__dirname, 'dist-server'),
  filename: 'server.js'
}
```

### Middleware Configuration

#### Common middleware issues:

```javascript
// Bad: Incorrect path resolution
app.use(render({
  main: './dist-server/server.js', // Relative path might not work
  template: indexTemplate
}));

// Good: Absolute path resolution
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate
}));
```

## Deployment Issues

### Production Environment Problems

1. **Environment variable configuration:**

```bash
# Set appropriate environment
NODE_ENV=production

# Configure logging
LOG_LEVEL=error
```

2. **Process management:**

```bash
# Use process managers like PM2
npm install -g pm2
pm2 start server/index.js --name "aurelia-ssr"
```

### Docker Considerations

If using Docker, ensure proper Node.js and build setup:

```dockerfile
FROM node:14-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000
CMD ["npm", "start"]
```

## Getting Help

If you're still experiencing issues:

1. **Check the browser console** for client-side errors
2. **Review server logs** for detailed error information  
3. **Test with JavaScript disabled** to verify SSR is working
4. **Compare with a minimal SSR setup** to isolate the problem
5. **Search the Aurelia GitHub issues** for similar problems
6. **Ask on the Aurelia Discord** or forums for community help

## Performance Monitoring

Set up monitoring to catch issues early:

```javascript
// Basic performance monitoring
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const duration = Date.now() - start;
  
  if (duration > 1000) { // Log slow requests
    console.warn(`Slow SSR render: ${ctx.url} took ${duration}ms`);
  }
});
```

Remember that SSR debugging can be complex due to the dual-environment nature. Always test both server-side rendering and client-side hydration to ensure your application works correctly in all scenarios.