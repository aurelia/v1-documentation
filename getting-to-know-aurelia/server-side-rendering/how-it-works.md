# How it works

Aurelia's SSR implementation involves several key components and processes:

## Core Libraries

* **`aurelia-middleware-koa`**: A middleware for Koa that handles rendering requests.
* **`aurelia-ssr-bootstrapper-webpack`**: Manages the initialization and termination of the Aurelia application on the server.
* **`aurelia-ssr-engine`**: Coordinates the rendering process and interacts with the middleware.

## Rendering Process

1. **Incoming Request**: When a request arrives at the server, `aurelia-middleware-koa` intercepts it.
2. **Render Invocation**: The middleware invokes `aurelia-ssr-engine` to render the requested page.
3. **Aurelia Instance**: A new Aurelia instance is created, and the application starts rendering based on the request URL.
4. **HTML Response**: The fully rendered HTML is sent back to the client once rendering is complete.
5. **Cleanup**: The Aurelia instance is stopped and released from memory to free resources.

## Client-Side Hydration

After the server sends the rendered HTML:

1. **Initial Render**: The client displays the HTML immediately, providing a fast initial view.
2. **Bundle Loading**: The client downloads the necessary JavaScript bundles.
3. **Application Startup**: Aurelia initializes on the client, taking over interactivity from the server-rendered HTML.
4. **Preboot Integration**: During the period between HTML rendering and client application startup, any user interactions are captured by **Preboot** and replayed once the application is active, ensuring seamless user experience.

## Entry Points

Aurelia maintains separate bundles for the client and server:

* **Client Bundle**: Contains the code necessary for client-side interactivity.
* **Server Bundle**: Contains the code required for server-side rendering.

This separation allows different configurations and optimizations tailored to each environment.
