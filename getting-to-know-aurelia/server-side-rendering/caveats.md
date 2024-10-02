# Caveats

While SSR offers significant advantages, it also introduces some challenges:

## Dual Environment Execution

Your application will run in two different environments:

* **Browser**: The familiar client-side environment with access to the DOM, `window`, and `document`.
* **Node.js**: A server-side environment without native DOM APIs.

To handle these differences, Aurelia uses a **Platform Abstraction Layer (PAL)** that abstracts platform-specific logic. It's crucial to utilize these abstractions to ensure compatibility across both environments.

## Limited Global Objects

In the server environment:

* There is no real DOM, `window`, or `document` globals.
* Libraries that depend on these globals (e.g., jQuery) will not function correctly.

Aurelia uses **jsdom** to emulate the DOM on the server. To maintain compatibility, ensure that all DOM manipulations go through the PAL.

## Development Effort

Creating an application compatible with both client and server rendering requires additional effort:

* **Isomorphic Code**: Writing code that runs in both environments.
* **Testing**: Ensuring that features work seamlessly on both the client and server.

Only adopt SSR if you're prepared to invest the necessary time and resources.

## Resource Consumption

SSR increases server load because the server handles rendering tasks:

* **CPU and Memory**: More CPU cycles and memory are required to render pages on the server.
* **Scalability**: Ensure your server infrastructure can handle the increased load, especially under high traffic.
