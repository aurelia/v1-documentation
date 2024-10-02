# Server side rendering

Server Side Rendering (SSR) is a technique where the server is responsible for rendering your Aurelia application into HTML before sending it to the client. This approach offers several benefits:

* **Faster Initial Load**: Users see a fully rendered page more quickly since the server generates the HTML.
* **Improved SEO**: Search engines can index your content more effectively because the content is present in the initial HTML response.
* **Enhanced Performance**: It reduces the time to the first meaningful paint, providing a better user experience, especially on slower networks or devices.

Aurelia supports SSR by leveraging its isomorphic capabilities, allowing the same application code to run seamlessly on both the client and server.

