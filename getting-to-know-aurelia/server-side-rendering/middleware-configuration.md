# Middleware configuration

This guide provides detailed examples and configuration options for setting up Aurelia SSR middleware with various server frameworks and scenarios.

## Basic Koa Middleware Setup

The most common setup uses Koa with `aurelia-middleware-koa`:

```javascript
const Koa = require('koa');
const render = require('aurelia-middleware-koa');
const path = require('path');
const fs = require('fs');

const app = new Koa();

// Load HTML template
const indexTemplate = fs.readFileSync(
  path.join(__dirname, '../index.html'), 
  'utf-8'
);

// Basic SSR middleware configuration
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate
}));

app.listen(3000);
```

## Advanced Configuration Options

### Complete Configuration Object

```javascript
app.use(render({
  // Required: Path to your server bundle
  main: path.join(__dirname, '../dist-server/server.js'),
  
  // Required: HTML template
  template: indexTemplate,
  
  // Optional: Custom template insertion logic
  insertInto: (template, rendered) => {
    return template.replace('<!--aurelia-app-->', rendered);
  },
  
  // Optional: Error handling
  onError: (error, request) => {
    console.error('SSR Error:', error);
    return '<div>Error loading page</div>';
  },
  
  // Optional: Request preprocessing
  beforeRender: (request) => {
    // Modify request before rendering
    console.log(`Rendering: ${request.url}`);
    return request;
  },
  
  // Optional: Response post-processing
  afterRender: (html, request) => {
    // Modify rendered HTML
    return html.replace('{{timestamp}}', new Date().toISOString());
  },
  
  // Optional: Skip SSR for certain routes
  skipRoutes: ['/api', '/admin'],
  
  // Optional: Cache configuration
  cache: {
    enabled: process.env.NODE_ENV === 'production',
    ttl: 60000, // 1 minute cache
    key: (request) => request.url
  }
}));
```

## Environment-Specific Configuration

### Development Configuration

```javascript
const isDevelopment = process.env.NODE_ENV === 'development';

app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  // Enable detailed error reporting in development
  onError: isDevelopment 
    ? (error, request) => {
        console.error('SSR Error:', error.stack);
        return `
          <div style="color: red; padding: 20px;">
            <h2>SSR Error (Development Only)</h2>
            <pre>${error.stack}</pre>
            <p>URL: ${request.url}</p>
          </div>
        `;
      }
    : (error, request) => {
        console.error('SSR Error:', error.message);
        return '<div>Page temporarily unavailable</div>';
      },
  
  // Disable caching in development
  cache: {
    enabled: false
  }
}));
```

### Production Configuration

```javascript
const isProduction = process.env.NODE_ENV === 'production';

if (isProduction) {
  app.use(render({
    main: path.join(__dirname, '../dist-server/server.js'),
    template: indexTemplate,
    
    // Graceful error handling
    onError: (error, request) => {
      // Log error for monitoring
      console.error(`SSR Error [${new Date().toISOString()}]:`, {
        message: error.message,
        url: request.url,
        userAgent: request.headers['user-agent']
      });
      
      // Return fallback content
      return `
        <div class="fallback-content">
          <h1>Welcome</h1>
          <p>Please enable JavaScript to view this application.</p>
          <noscript>
            <p>This application requires JavaScript to function properly.</p>
          </noscript>
        </div>
      `;
    },
    
    // Enable production caching
    cache: {
      enabled: true,
      ttl: 5 * 60 * 1000, // 5 minutes
      key: (request) => {
        // Create cache key based on URL and user agent
        return `${request.url}-${request.headers['user-agent']}`;
      }
    }
  }));
}
```

## Custom Template Insertion

### Multiple Insertion Points

```javascript
const template = `
<!DOCTYPE html>
<html>
<head>
  <title><!--title-placeholder--></title>
  <meta name="description" content="<!--description-placeholder-->">
</head>
<body>
  <div id="app"><!--content-placeholder--></div>
  <script>window.__INITIAL_STATE__ = <!--state-placeholder-->;</script>
</body>
</html>
`;

app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: template,
  
  insertInto: (template, rendered, request, initialState) => {
    return template
      .replace('<!--title-placeholder-->', initialState.title || 'Default Title')
      .replace('<!--description-placeholder-->', initialState.description || 'Default description')
      .replace('<!--content-placeholder-->', rendered)
      .replace('<!--state-placeholder-->', JSON.stringify(initialState));
  }
}));
```

### Dynamic Meta Tags

```javascript
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  insertInto: (template, rendered, request, state) => {
    // Generate meta tags based on rendered content
    const metaTags = generateMetaTags(state, request);
    
    const htmlWithMeta = template.replace(
      '<head>',
      `<head>${metaTags}`
    );
    
    return htmlWithMeta.replace('<!--aurelia-app-->', rendered);
  }
}));

function generateMetaTags(state, request) {
  const meta = [];
  
  if (state.pageTitle) {
    meta.push(`<title>${state.pageTitle}</title>`);
  }
  
  if (state.pageDescription) {
    meta.push(`<meta name="description" content="${state.pageDescription}">`);
  }
  
  // Open Graph tags
  if (state.ogImage) {
    meta.push(`<meta property="og:image" content="${state.ogImage}">`);
  }
  
  return meta.join('\n');
}
```

## Request Preprocessing

### Authentication Handling

```javascript
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  beforeRender: (request) => {
    // Extract authentication information
    const authToken = request.headers.authorization;
    const userId = extractUserIdFromToken(authToken);
    
    // Add user context to request
    request.userContext = {
      isAuthenticated: !!userId,
      userId: userId,
      permissions: getUserPermissions(userId)
    };
    
    return request;
  }
}));

function extractUserIdFromToken(authToken) {
  // Implement your JWT/token parsing logic
  try {
    const token = authToken?.replace('Bearer ', '');
    // Parse and validate token
    return parseJWT(token)?.userId;
  } catch (error) {
    return null;
  }
}
```

### Internationalization

```javascript
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  beforeRender: (request) => {
    // Determine user's language preference
    const acceptLanguage = request.headers['accept-language'];
    const userLang = parseLanguageHeader(acceptLanguage);
    
    // Add language context
    request.i18nContext = {
      locale: userLang,
      fallbackLocale: 'en',
      translations: loadTranslations(userLang)
    };
    
    return request;
  }
}));

function parseLanguageHeader(acceptLanguage) {
  // Parse Accept-Language header
  const languages = acceptLanguage?.split(',').map(lang => {
    const [code, priority = '1'] = lang.trim().split(';q=');
    return { code: code.split('-')[0], priority: parseFloat(priority) };
  }).sort((a, b) => b.priority - a.priority);
  
  return languages?.[0]?.code || 'en';
}
```

## Response Post-processing

### Performance Monitoring

```javascript
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  afterRender: (html, request, renderTime) => {
    // Add performance metrics
    const perfScript = `
      <script>
        window.__SSR_METRICS__ = {
          renderTime: ${renderTime},
          timestamp: ${Date.now()},
          url: '${request.url}'
        };
      </script>
    `;
    
    return html.replace('</body>', `${perfScript}</body>`);
  }
}));
```

### Content Security Policy

```javascript
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  afterRender: (html, request) => {
    // Generate nonce for inline scripts
    const nonce = generateNonce();
    
    // Add CSP header
    request.ctx.set('Content-Security-Policy', 
      `default-src 'self'; script-src 'self' 'nonce-${nonce}'; style-src 'self' 'unsafe-inline';`
    );
    
    // Update inline scripts with nonce
    return html.replace(
      /<script>/g, 
      `<script nonce="${nonce}">`
    );
  }
}));

function generateNonce() {
  return require('crypto').randomBytes(16).toString('base64');
}
```

## Route-Specific Configuration

### Skip SSR for API Routes

```javascript
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  skipRoutes: [
    '/api',           // Skip all API routes
    '/admin',         // Skip admin panel
    /\.json$/,        // Skip JSON endpoints
    /\.xml$/,         // Skip XML endpoints
    (request) => {    // Custom logic
      return request.headers['x-requested-with'] === 'XMLHttpRequest';
    }
  ]
}));
```

### Route-Specific Templates

```javascript
const templates = {
  admin: fs.readFileSync(path.join(__dirname, '../admin.html'), 'utf-8'),
  mobile: fs.readFileSync(path.join(__dirname, '../mobile.html'), 'utf-8'),
  default: fs.readFileSync(path.join(__dirname, '../index.html'), 'utf-8')
};

app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  
  template: (request) => {
    if (request.url.startsWith('/admin')) {
      return templates.admin;
    }
    
    if (isMobileUserAgent(request.headers['user-agent'])) {
      return templates.mobile;
    }
    
    return templates.default;
  }
}));
```

## Caching Configuration

### Memory Cache Implementation

```javascript
const NodeCache = require('node-cache');
const cache = new NodeCache({ 
  stdTTL: 600, // 10 minutes default
  checkperiod: 120 // Check for expired keys every 2 minutes
});

app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  cache: {
    enabled: true,
    
    get: (key) => {
      return cache.get(key);
    },
    
    set: (key, value, ttl) => {
      cache.set(key, value, ttl / 1000); // Convert to seconds
    },
    
    key: (request) => {
      // Create cache key based on URL and user context
      const userKey = request.userContext?.userId || 'anonymous';
      return `${request.url}-${userKey}`;
    },
    
    ttl: (request) => {
      // Different TTL for different routes
      if (request.url.includes('/news')) {
        return 60 * 1000; // 1 minute for news
      }
      if (request.url === '/') {
        return 300 * 1000; // 5 minutes for homepage
      }
      return 600 * 1000; // 10 minutes default
    }
  }
}));
```

### Redis Cache Implementation

```javascript
const redis = require('redis');
const client = redis.createClient();

app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  cache: {
    enabled: true,
    
    get: async (key) => {
      try {
        const value = await client.get(`ssr:${key}`);
        return value ? JSON.parse(value) : null;
      } catch (error) {
        console.error('Cache get error:', error);
        return null;
      }
    },
    
    set: async (key, value, ttl) => {
      try {
        await client.setex(
          `ssr:${key}`, 
          Math.floor(ttl / 1000), 
          JSON.stringify(value)
        );
      } catch (error) {
        console.error('Cache set error:', error);
      }
    },
    
    key: (request) => {
      return `${request.method}:${request.url}`;
    }
  }
}));
```

## Error Handling Strategies

### Graceful Degradation

```javascript
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  onError: (error, request) => {
    // Log error details
    console.error(`SSR Error [${new Date().toISOString()}]:`, {
      error: error.message,
      stack: error.stack,
      url: request.url,
      userAgent: request.headers['user-agent']
    });
    
    // Different fallbacks based on error type
    if (error.code === 'MODULE_NOT_FOUND') {
      return generateModuleErrorFallback(request);
    }
    
    if (error.name === 'TimeoutError') {
      return generateTimeoutFallback(request);
    }
    
    // Default fallback
    return generateDefaultFallback(request);
  }
}));

function generateModuleErrorFallback(request) {
  return `
    <div class="error-fallback">
      <h1>Service Unavailable</h1>
      <p>The requested page is temporarily unavailable.</p>
      <p><a href="/">Return to homepage</a></p>
    </div>
  `;
}

function generateTimeoutFallback(request) {
  return `
    <div class="loading-fallback">
      <h1>Loading...</h1>
      <p>This page is taking longer than usual to load.</p>
      <script>
        // Attempt client-side rendering after delay
        setTimeout(function() {
          window.location.reload();
        }, 3000);
      </script>
    </div>
  `;
}
```

## Integration with Other Middleware

### Combining with Static File Serving

```javascript
const Koa = require('koa');
const serve = require('koa-static');
const render = require('aurelia-middleware-koa');

const app = new Koa();

// Serve static files first
app.use(serve(path.join(__dirname, '../dist'), {
  maxAge: 86400000, // 1 day cache for static files
  gzip: true
}));

// Then SSR for remaining routes
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate
}));

app.listen(3000);
```

### With Authentication Middleware

```javascript
const session = require('koa-session');
const passport = require('koa-passport');

// Session middleware
app.use(session(app));
app.use(passport.initialize());
app.use(passport.session());

// Authentication middleware
app.use(async (ctx, next) => {
  if (ctx.isAuthenticated()) {
    ctx.state.user = ctx.state.user || {};
    ctx.state.user.isAuthenticated = true;
  }
  await next();
});

// SSR middleware with user context
app.use(render({
  main: path.join(__dirname, '../dist-server/server.js'),
  template: indexTemplate,
  
  beforeRender: (request) => {
    // Pass user authentication state to SSR
    request.authState = {
      isAuthenticated: request.ctx.isAuthenticated(),
      user: request.ctx.state.user
    };
    return request;
  }
}));
```

This comprehensive middleware configuration guide should help you set up SSR for various scenarios and requirements. Remember to test your configuration thoroughly in both development and production environments.