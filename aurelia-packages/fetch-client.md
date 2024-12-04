# Fetch client

The Aurelia Fetch Client (`aurelia-fetch-client`) is a powerful HTTP client that wraps the browser's native Fetch API while providing additional features essential for modern web applications. It offers default configuration management, interceptors, centralized request tracking, and other utilities while maintaining compatibility with the standard Fetch API specification.

### Installation

Install the package using npm:

```bash
npm install aurelia-fetch-client
```

### Key Features

* Fully compatible with the Fetch API specification
* Configurable defaults for requests
* Powerful interceptor system
* Request tracking
* JSON handling utilities
* Base URL configuration
* Credential management
* Error handling helpers

### Basic Usage

#### Creating an HttpClient Instance

To use the Fetch Client, first import and create an instance of `HttpClient`:

```javascript
import { HttpClient } from 'aurelia-fetch-client';

const http = new HttpClient();
```

#### Making HTTP Requests

The `fetch` method is the primary way to make HTTP requests. It accepts the same parameters as the native `fetch` API:

```javascript
// GET request
http.fetch('api/users')
    .then(response => response.json())
    .then(data => {
        console.log(data);
    });

// POST request
http.fetch('api/users', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ name: 'John Doe' })
});
```

#### Working with JSON

Aurelia's Fetch Client provides a `json` helper function to simplify JSON requests:

```javascript
import { HttpClient, json } from 'aurelia-fetch-client';

const http = new HttpClient();

const user = {
    name: 'John Doe',
    email: 'john@example.com'
};

http.fetch('api/users', {
    method: 'POST',
    body: json(user) // Automatically sets Content-Type header and stringifies
});
```

#### Response Handling

Responses can be processed in various formats:

```javascript
http.fetch('api/data')
    .then(response => {
        // Check response status
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        // Different response formats
        response.json();    // Parse JSON response
        response.text();    // Get response as text
        response.blob();    // Handle binary data
        response.formData(); // Handle form data
        
        // Access headers
        response.headers.get('Content-Type');
        
        // Get response status
        console.log(response.status, response.statusText);
    });
```

#### Error Handling

Implement proper error handling using try/catch or Promise chains:

```javascript
http.fetch('api/data')
    .then(response => response.json())
    .then(data => {
        // Handle success
    })
    .catch(error => {
        if (error.name === 'TypeError') {
            // Handle network errors
        } else {
            // Handle other errors
        }
    });
```

### Configuration

#### Global Configuration

Configure the HttpClient instance using the `configure` method:

```javascript
http.configure(config => {
    config
        .useStandardConfiguration()
        .withBaseUrl('https://api.example.com/')
        .withDefaults({
            headers: {
                'Accept': 'application/json',
                'X-Requested-With': 'Fetch'
            }
        });
});
```

#### Available Configuration Options

**Base URL Configuration**

Set a base URL for all requests:

```javascript
config.withBaseUrl('https://api.example.com/');
```

**Default Settings**

Configure default options for all requests:

```javascript
config.withDefaults({
    credentials: 'same-origin',
    headers: {
        'Accept': 'application/json',
        'X-Requested-With': 'Fetch'
    },
    mode: 'cors',
    cache: 'default'
});
```

**Standard Configuration**

Apply recommended default settings:

```javascript
config.useStandardConfiguration();
```

This applies:

* Rejection of error responses (4xx, 5xx status codes)
* Same-origin credentials
* Default headers

**Credentials Configuration**

Configure how credentials are handled:

```javascript
config.withDefaults({
    credentials: 'same-origin' // Or 'include' for cross-origin requests
});
```

Available credential options:

* `'same-origin'`: Send credentials only to same-origin URLs
* `'include'`: Send credentials to all URLs
* `'omit'`: Never send credentials

#### Request Defaults

Set default parameters for specific types of requests:

```javascript
config.withDefaults({
    // Request mode
    mode: 'cors', // or 'same-origin', 'no-cors'
    
    // Cache control
    cache: 'default', // or 'no-cache', 'reload', 'force-cache'
    
    // Redirect handling
    redirect: 'follow', // or 'error', 'manual'
    
    // Referrer policy
    referrerPolicy: 'no-referrer-when-downgrade'
});
```

{% hint style="info" %}
Configuration settings can be overridden on a per-request basis by passing options directly to the fetch call.
{% endhint %}

{% hint style="warning" %}
When using `withBaseUrl`, ensure the URL ends with a forward slash (/) if you want path segments to be appended correctly.
{% endhint %}

The configuration system is chainable, allowing you to combine multiple configuration options:

```javascript
http.configure(config => {
    config
        .useStandardConfiguration()
        .withBaseUrl('https://api.example.com/')
        .withDefaults({
            headers: {
                'Authorization': `Bearer ${token}`
            }
        })
        .rejectErrorResponses();
});
```

### Interceptors

Interceptors provide a powerful way to transform requests and responses, handle errors, and add cross-cutting concerns to HTTP communications. They can intercept requests before they are sent and responses before they are handled by your application.

#### Interceptor Structure

An interceptor can implement any of these four methods:

* `request(request)`
* `requestError(error)`
* `response(response)`
* `responseError(error)`

```javascript
const loggingInterceptor = {
    request(request) {
        console.log(`Requesting ${request.method} ${request.url}`);
        return request;
    },
    response(response) {
        console.log(`Received ${response.status} from ${response.url}`);
        return response;
    }
};
```

#### Adding Interceptors

Add interceptors during configuration:

```javascript
http.configure(config => {
    config.withInterceptor(loggingInterceptor);
    
    // Or inline
    config.withInterceptor({
        request(request) {
            request.headers.set('X-Custom-Header', 'value');
            return request;
        }
    });
});
```

#### Request Interceptors

Request interceptors can modify requests before they are sent:

```javascript
const authInterceptor = {
    request(request) {
        // Add authentication header
        const token = localStorage.getItem('token');
        request.headers.set('Authorization', `Bearer ${token}`);
        
        // You can also create a new request
        return new Request(request.url, {
            ...request,
            headers: request.headers
        });
    }
};
```

#### Response Interceptors

Response interceptors can transform responses before they reach your application:

```javascript
const responseInterceptor = {
    response(response) {
        if (response.status === 404) {
            return Response.json({ message: 'Custom 404 message' });
        }
        
        // Add custom response header
        const modifiedResponse = response.clone();
        modifiedResponse.headers.set('X-Custom-Header', 'value');
        return modifiedResponse;
    }
};
```

#### Error Handling in Interceptors

Handle errors in both request and response phases:

```javascript
const errorInterceptor = {
    requestError(error) {
        console.error('Request error:', error);
        // Either throw error or return a new Request
        throw error;
    },
    
    responseError(error) {
        if (error.response?.status === 401) {
            // Handle unauthorized access
            return refreshToken()
                .then(() => {
                    // Retry the original request
                    return http.fetch(error.request);
                });
        }
        throw error;
    }
};
```

#### Async Interceptors

Interceptors can return promises for asynchronous operations:

```javascript
const asyncInterceptor = {
    async request(request) {
        const token = await getTokenAsync();
        request.headers.set('Authorization', `Bearer ${token}`);
        return request;
    }
};
```

### Advanced Features

#### AbortController Integration

Use `AbortController` to cancel requests:

```javascript
const controller = new AbortController();
const { signal } = controller;

http.fetch('api/longOperation', { signal })
    .then(response => response.json())
    .catch(error => {
        if (error.name === 'AbortError') {
            console.log('Request was cancelled');
        }
    });

// Cancel the request
controller.abort();
```

#### Request Timeout

Implement request timeout using `AbortController`:

```javascript
function fetchWithTimeout(url, options = {}, timeout = 5000) {
    const controller = new AbortController();
    const { signal } = controller;
    
    return Promise.race([
        http.fetch(url, { ...options, signal }),
        new Promise((_, reject) => {
            setTimeout(() => {
                controller.abort();
                reject(new Error('Request timeout'));
            }, timeout);
        })
    ]);
}
```

#### Progress Tracking

Track upload progress using `fetch` with `XMLHttpRequest`:

```javascript
function fetchWithProgress(url, options = {}, onProgress) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open(options.method || 'GET', url);
        
        // Set headers
        Object.keys(options.headers || {}).forEach(key => {
            xhr.setRequestHeader(key, options.headers[key]);
        });
        
        xhr.onload = () => {
            resolve(new Response(xhr.response, {
                status: xhr.status,
                statusText: xhr.statusText
            }));
        };
        
        xhr.onerror = () => reject(new Error('Network request failed'));
        
        xhr.upload.onprogress = onProgress;
        
        xhr.send(options.body);
    });
}

// Usage
fetchWithProgress('api/upload', {
    method: 'POST',
    body: formData
}, (event) => {
    const percent = (event.loaded / event.total) * 100;
    console.log(`Upload progress: ${percent}%`);
});
```

#### Working with FormData

Handle file uploads and form data:

```javascript
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('name', 'example.jpg');

http.fetch('api/upload', {
    method: 'POST',
    body: formData,
    // Don't set Content-Type header, browser will set it with boundary
})
.then(response => response.json())
.then(result => {
    console.log('Upload successful:', result);
});
```

#### Custom Request Initialization

Create custom request configurations for specific use cases:

```javascript
class ApiClient {
    constructor(http) {
        this.http = http;
    }
    
    createRequest(url, options = {}) {
        return new Request(url, {
            ...options,
            headers: new Headers({
                'Content-Type': 'application/json',
                'X-Custom-Header': 'value',
                ...(options.headers || {})
            })
        });
    }
    
    async fetch(url, options = {}) {
        const request = this.createRequest(url, options);
        return this.http.fetch(request);
    }
}
```

{% hint style="warning" %}
Don't manually set the header when working with `FormData` and file uploads. The browser needs to set this automatically to include the correct boundary parameter.
{% endhint %}

{% hint style="info" %}
For better performance with large file uploads, consider using chunked uploads by splitting the file into smaller pieces and tracking progress for each chunk.
{% endhint %}

### API Reference

#### HttpClient Class

```typescript
class HttpClient {
    // Constructor
    constructor();
    
    // Core methods
    configure(config: (config: HttpClientConfiguration) => void): HttpClient;
    fetch(input: string | Request, init?: RequestInit): Promise<Response>;
    
    // Configuration state
    baseUrl: string;
    defaults: RequestInit;
    interceptors: Interceptor[];
    isConfigured: boolean;
}
```

#### Configuration Options

```typescript
interface HttpClientConfiguration {
    // Configuration methods
    withBaseUrl(baseUrl: string): HttpClientConfiguration;
    withDefaults(defaults: RequestInit): HttpClientConfiguration;
    withInterceptor(interceptor: Interceptor): HttpClientConfiguration;
    useStandardConfiguration(): HttpClientConfiguration;
    rejectErrorResponses(): HttpClientConfiguration;
}
```

#### Interceptor Interfaces

```typescript
interface Interceptor {
    request?(request: Request): Request | Response | Promise<Request | Response>;
    requestError?(error: any): Request | Promise<Request>;
    response?(response: Response, request?: Request): Response | Promise<Response>;
    responseError?(error: any, request?: Request): Response | Promise<Response>;
}
```

#### Helper Functions

```typescript
// JSON helper
function json(body: any, replacer?: any): Blob {
    return new Blob([JSON.stringify(body, replacer)], {
        type: 'application/json'
    });
}
```

#### RequestInit Interface

```typescript
interface RequestInit {
    method?: string;
    headers?: HeadersInit;
    body?: BodyInit;
    mode?: RequestMode;
    credentials?: RequestCredentials;
    cache?: RequestCache;
    redirect?: RequestRedirect;
    referrer?: string;
    referrerPolicy?: ReferrerPolicy;
    integrity?: string;
    keepalive?: boolean;
    signal?: AbortSignal;
}
```

### Common Use Cases

#### Authentication

**JWT Authentication**

```javascript
http.configure(config => {
    config.withInterceptor({
        request(request) {
            const token = localStorage.getItem('jwt');
            if (token) {
                request.headers.set('Authorization', `Bearer ${token}`);
            }
            return request;
        },
        responseError(error) {
            if (error.response?.status === 401) {
                // Token expired or invalid
                return refreshToken()
                    .then(newToken => {
                        localStorage.setItem('jwt', newToken);
                        // Retry the original request
                        const request = error.request;
                        request.headers.set('Authorization', `Bearer ${newToken}`);
                        return http.fetch(request);
                    });
            }
            throw error;
        }
    });
});
```

**Basic Authentication**

```javascript
http.configure(config => {
    config.withDefaults({
        headers: {
            'Authorization': 'Basic ' + btoa('username:password')
        }
    });
});
```

#### RESTful API Integration

```javascript
class ApiService {
    constructor() {
        this.http = new HttpClient();
        this.http.configure(config => {
            config
                .useStandardConfiguration()
                .withBaseUrl('https://api.example.com/v1/')
                .withDefaults({
                    headers: {
                        'Accept': 'application/json'
                    }
                });
        });
    }

    async getResource(id) {
        const response = await this.http.fetch(`resources/${id}`);
        return response.json();
    }

    async createResource(data) {
        const response = await this.http.fetch('resources', {
            method: 'POST',
            body: json(data)
        });
        return response.json();
    }

    async updateResource(id, data) {
        const response = await this.http.fetch(`resources/${id}`, {
            method: 'PUT',
            body: json(data)
        });
        return response.json();
    }

    async deleteResource(id) {
        await this.http.fetch(`resources/${id}`, {
            method: 'DELETE'
        });
    }
}
```

#### File Downloads

```javascript
class DownloadService {
    constructor(http) {
        this.http = http;
    }

    async downloadFile(url, filename) {
        const response = await this.http.fetch(url);
        const blob = await response.blob();
        
        // Create download link
        const downloadUrl = window.URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = downloadUrl;
        link.download = filename;
        
        // Trigger download
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        
        // Cleanup
        window.URL.revokeObjectURL(downloadUrl);
    }

    // Download with progress
    async downloadWithProgress(url, filename, onProgress) {
        const response = await this.http.fetch(url);
        const reader = response.body.getReader();
        const contentLength = +response.headers.get('Content-Length');

        let receivedLength = 0;
        const chunks = [];

        while(true) {
            const {done, value} = await reader.read();
            
            if (done) break;
            
            chunks.push(value);
            receivedLength += value.length;
            
            onProgress((receivedLength / contentLength) * 100);
        }

        const blob = new Blob(chunks);
        const downloadUrl = window.URL.createObjectURL(blob);
        
        // Download logic same as above
    }
}
```

#### Request Cancellation Pattern

```javascript
class CancellableRequest {
    constructor(http) {
        this.http = http;
        this.controller = null;
    }

    async execute(url, options = {}) {
        // Cancel any existing request
        this.cancel();
        
        // Create new controller
        this.controller = new AbortController();
        
        try {
            const response = await this.http.fetch(url, {
                ...options,
                signal: this.controller.signal
            });
            return response.json();
        } catch (error) {
            if (error.name === 'AbortError') {
                console.log('Request cancelled');
            }
            throw error;
        }
    }

    cancel() {
        if (this.controller) {
            this.controller.abort();
            this.controller = null;
        }
    }
}

// Usage
const searchRequest = new CancellableRequest(http);

// In your view model
async search(term) {
    try {
        const results = await searchRequest.execute(`search?q=${term}`);
        this.results = results;
    } catch (error) {
        // Handle error
    }
}

// Cancel when component is destroyed
deactivate() {
    searchRequest.cancel();
}
```

{% hint style="info" %}
When implementing file downloads, always check the Content-Disposition header to get the server-suggested filename if available.
{% endhint %}

{% hint style="warning" %}
Consider implementing chunking and resume capabilities for large file downloads to handle network interruptions gracefully.
{% endhint %}

