# Setup and installation

This guide will walk you through setting up Server Side Rendering (SSR) for your Aurelia application from scratch.

## Prerequisites

Before setting up SSR, ensure you have:

- **Node.js** (version 8.9.0 or later)
- **npm** or **yarn** package manager
- **Existing Aurelia CLI project** or willingness to create one

## Installing Required Packages

Install the core SSR packages for Aurelia:

```bash
npm install --save aurelia-middleware-koa aurelia-ssr-bootstrapper-webpack aurelia-ssr-engine
```

### Additional Dependencies

You'll also need these supporting packages:

```bash
npm install --save koa koa-static koa-router jsdom aurelia-pal-nodejs
npm install --save-dev webpack-node-externals
```

## Project Structure

A typical SSR-enabled Aurelia project structure looks like this:

```
my-aurelia-app/
├── src/
│   ├── main.ts                 # Client entry point
│   ├── server-main.ts          # Server entry point
│   ├── app.ts                  # Shared app component
│   ├── app.html
│   └── components/
├── server/
│   ├── index.js                # Server setup
│   └── ssr-server.js           # SSR configuration
├── webpack.config.js           # Webpack configuration
├── package.json
└── aurelia_project/
```

## Step 1: Create Server Entry Point

Create a server-specific entry point (`src/server-main.ts`):

```typescript
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from 'aurelia-pal';
import 'aurelia-polyfills';
import bootstrapper from 'aurelia-ssr-bootstrapper-webpack';

async function configure(aurelia: Aurelia): Promise<void> {
  aurelia.use
    .standardConfiguration()
    .plugin(PLATFORM.moduleName('aurelia-templating-binding'))
    .plugin(PLATFORM.moduleName('aurelia-templating-resources'))
    .plugin(PLATFORM.moduleName('aurelia-history-browser'))
    .plugin(PLATFORM.moduleName('aurelia-templating-router'));

  // Optionally configure logging
  if (process.env.NODE_ENV === 'development') {
    aurelia.use.developmentLogging();
  }

  await aurelia.start();
  await aurelia.setRoot(PLATFORM.moduleName('app'));
}

module.exports = bootstrapper(configure);
```

## Step 2: Update Client Entry Point

Ensure your client entry point (`src/main.ts`) is properly configured:

```typescript
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from 'aurelia-pal';
import 'aurelia-polyfills';

export function configure(aurelia: Aurelia) {
  aurelia.use
    .standardConfiguration();

  if (process.env.NODE_ENV === 'development') {
    aurelia.use.developmentLogging();
  }

  aurelia.start().then(() => {
    aurelia.setRoot(PLATFORM.moduleName('app'));
  });
}
```

## Step 3: Configure Webpack

Update your `webpack.config.js` to support both client and server builds:

```javascript
const path = require('path');
const nodeExternals = require('webpack-node-externals');
const { AureliaPlugin } = require('aurelia-webpack-plugin');

const commonConfig = {
  resolve: {
    extensions: ['.ts', '.js'],
    modules: ['src', 'node_modules']
  },
  module: {
    rules: [
      { test: /\.ts$/, loader: 'ts-loader' },
      { test: /\.html$/, loader: 'aurelia-webpack-plugin/html-loader' },
      { test: /\.css$/, loader: ['style-loader', 'css-loader'] }
    ]
  }
};

module.exports = [
  // Client configuration
  {
    ...commonConfig,
    target: 'web',
    entry: {
      app: ['aurelia-bootstrapper']
    },
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: '[name].js',
      publicPath: '/dist/'
    },
    plugins: [
      new AureliaPlugin({
        aureliaApp: 'main'
      })
    ]
  },
  
  // Server configuration  
  {
    ...commonConfig,
    target: 'node',
    entry: './src/server-main.ts',
    output: {
      path: path.resolve(__dirname, 'dist-server'),
      filename: 'server.js',
      libraryTarget: 'commonjs2'
    },
    externals: [nodeExternals()],
    plugins: [
      new AureliaPlugin({
        aureliaApp: undefined,
        dist: 'native-modules'
      })
    ]
  }
];
```

## Step 4: Create Server Setup

Create the server file (`server/index.js`):

```javascript
const Koa = require('koa');
const serve = require('koa-static');
const path = require('path');
const ssrServer = require('./ssr-server');

const app = new Koa();
const port = process.env.PORT || 3000;

// Serve static files
app.use(serve(path.join(__dirname, '../dist')));

// Configure SSR
ssrServer(app);

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```

## Step 5: Configure SSR Server

Create the SSR configuration (`server/ssr-server.js`):

```javascript
const path = require('path');
const fs = require('fs');
const render = require('aurelia-middleware-koa');

module.exports = function(app) {
  // Read the main HTML template
  const indexTemplate = fs.readFileSync(
    path.join(__dirname, '../index.html'), 
    'utf-8'
  );

  // Configure SSR middleware
  app.use(render({
    // Path to server bundle
    main: path.join(__dirname, '../dist-server/server.js'),
    
    // HTML template
    template: indexTemplate,
    
    // Optional: customize how HTML is merged
    insertInto: (template, rendered) => {
      return template.replace('<!--aurelia-app-->', rendered);
    },
    
    // Optional: handle errors gracefully
    onError: (error) => {
      console.error('SSR Error:', error);
      return `<div>Error rendering page</div>`;
    }
  }));
};
```

## Step 6: Update HTML Template

Update your `index.html` to include the placeholder for SSR content:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Aurelia SSR App</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <div aurelia-app="main">
    <!--aurelia-app-->
  </div>
  <script src="/dist/app.js"></script>
</body>
</html>
```

## Step 7: Update Package Scripts

Add build and run scripts to your `package.json`:

```json
{
  "scripts": {
    "build:client": "webpack --config webpack.config.js --env.target=web",
    "build:server": "webpack --config webpack.config.js --env.target=node",
    "build": "npm run build:client && npm run build:server",
    "start:dev": "npm run build && node server/index.js",
    "start": "node server/index.js"
  }
}
```

## Step 8: Platform Abstraction Layer Setup

Ensure PAL is properly initialized for Node.js in your server entry point:

```typescript
// At the top of server-main.ts
import { initialize } from 'aurelia-pal-nodejs';
import { JSDOM } from 'jsdom';

// Initialize PAL for Node.js environment
initialize();
```

## Development Workflow

1. **Build the application:**
   ```bash
   npm run build
   ```

2. **Start the server:**
   ```bash
   npm start
   ```

3. **For development with auto-rebuild:**
   ```bash
   # Terminal 1: Watch and rebuild
   webpack --watch
   
   # Terminal 2: Run server
   npm start
   ```

## Verification

To verify SSR is working:

1. **Start your server** and navigate to your application
2. **Disable JavaScript** in your browser
3. **Refresh the page** - you should still see rendered content
4. **View page source** - you should see rendered HTML instead of just the loading template

## Next Steps

Once you have basic SSR working:

- Review the [Caveats](caveats.md) section for important considerations
- Learn about [Isomorphic Code](isomorphic-code.md) best practices
- Implement proper [Memory Management](memory-management.md)
- Configure [Different Entry Points](different-entry-points.md) for advanced scenarios

## Troubleshooting

If you encounter issues during setup:

- Ensure all dependencies are correctly installed
- Verify webpack configurations are correct for both client and server
- Check that PAL is properly initialized
- Review server logs for detailed error messages
- Consult the [Troubleshooting Guide](troubleshooting.md) for common issues