# HTTP client

Aurelia provides a powerful HTTP client through the `aurelia-http-client` package. This client offers a comfortable interface to the browser's XMLHttpRequest object, providing an easy way to make HTTP requests in your Aurelia applications.

### Installation

Since the HTTP client is not included in Aurelia's default installation, you'll need to install it separately:

```bash
# Using npm
npm install aurelia-http-client

# Using yarn
yarn add aurelia-http-client
```

### Basic Concepts

The HTTP client in Aurelia provides:

* A fluent API for making HTTP requests
* Support for all standard HTTP methods
* Request/response interceptors
* Configurable defaults
* JSONP support
* Progress tracking for uploads and downloads

{% hint style="info" %}
By default, the HTTP client assumes you are expecting JSON responses and will automatically parse JSON response bodies.
{% endhint %}

### Getting Started

#### Setup with Dependency Injection

The recommended way to use the HTTP client in Aurelia is through dependency injection. Here's how to set it up:

```javascript
import { inject } from 'aurelia-framework';
import { HttpClient } from 'aurelia-http-client';

@inject(HttpClient)
export class MyClass {
    constructor(httpClient) {
        this.http = httpClient;
    }

    async getData() {
        try {
            const response = await this.http.get('api/data');
            return response.content;
        } catch (error) {
            console.error('Error fetching data:', error);
        }
    }
}
```

#### Alternative Setup

You can also create a new instance of the HTTP client directly, although this approach is less common in Aurelia applications:

```javascript
import { HttpClient } from 'aurelia-http-client';

export class MyClass {
    constructor() {
        this.http = new HttpClient();
    }
}
```

#### Simple Request Examples

Here are some common examples of making HTTP requests:

```javascript
// GET request
async function getUser(id) {
    const response = await this.http.get(`api/users/${id}`);
    return response.content;
}

// POST request with data
async function createUser(userData) {
    const response = await this.http.post('api/users', userData);
    return response.content;
}

// PUT request
async function updateUser(id, userData) {
    const response = await this.http.put(`api/users/${id}`, userData);
    return response.content;
}

// DELETE request
async function deleteUser(id) {
    const response = await this.http.delete(`api/users/${id}`);
    return response.content;
}
```

Each request returns a Promise that resolves with an `HttpResponseMessage` object. The `content` property of this object contains the parsed response data.

{% hint style="info" %}
You don't need to parse response data manually when working with JSON APIs. The HTTP client automatically handles JSON parsing for you when the response contains the appropriate content-type header.
{% endhint %}

#### Error Handling

It's important to implement proper error handling when making HTTP requests:

```javascript
import { inject } from 'aurelia-framework';
import { HttpClient } from 'aurelia-http-client';

@inject(HttpClient)
export class MyClass {
    constructor(httpClient) {
        this.http = httpClient;
    }

    async fetchData() {
        try {
            const response = await this.http.get('api/data');
            return response.content;
        } catch (error) {
            if (error.statusCode === 404) {
                console.error('Resource not found');
            } else if (error.statusCode === 401) {
                console.error('Unauthorized access');
            } else {
                console.error('An error occurred:', error);
            }
            throw error;
        }
    }
}
```

### Making HTTP Requests

The HTTP client provides methods for all standard HTTP verbs and JSONP requests. Each method returns a Promise that resolves to an `HttpResponseMessage`.

#### Available HTTP Methods

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
    }

    // GET request
    get(url) {
        return this.http.get(url);
    }

    // POST request with data
    post(url, data) {
        return this.http.post(url, data);
    }

    // PUT request with data
    put(url, data) {
        return this.http.put(url, data);
    }

    // PATCH request with data
    patch(url, data) {
        return this.http.patch(url, data);
    }

    // DELETE request
    delete(url) {
        return this.http.delete(url);
    }

    // HEAD request
    head(url) {
        return this.http.head(url);
    }

    // OPTIONS request
    options(url) {
        return this.http.options(url);
    }
}
```

#### JSONP Requests

JSONP requests are useful when you need to make cross-origin requests to services that don't support CORS. The HTTP client provides a dedicated `jsonp` method:

```javascript
@inject(HttpClient)
export class ExternalService {
    constructor(http) {
        this.http = http;
    }

    getExternalData() {
        return this.http.jsonp('http://api.example.com/data', 'callback')
            .then(response => response.content);
    }
}
```

The second parameter (`'callback'`) specifies the callback parameter name expected by the JSONP service.

#### Working with Request/Response Data

The `HttpResponseMessage` object provides several properties for handling response data:

```javascript
@inject(HttpClient)
export class DataService {
    constructor(http) {
        this.http = http;
    }

    fetchData() {
        return this.http.get('api/data')
            .then(response => {
                // Access various response properties
                console.log(response.statusCode);    // HTTP status code
                console.log(response.statusText);    // Status text
                console.log(response.isSuccess);     // Boolean indicating success
                console.log(response.headers);       // Response headers
                console.log(response.content);       // Parsed response content
                console.log(response.response);      // Raw response

                return response.content;
            });
    }
}
```

### Configuration

The HTTP client can be configured globally or per request. Configuration options include base URLs, headers, parameters, and more.

#### Global Configuration

Configure the HTTP client once for all requests:

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
        
        this.http.configure(config => {
            config.withBaseUrl('https://api.example.com/');
            config.withHeader('Authorization', 'Bearer ' + token);
            config.withHeader('Content-Type', 'application/json');
            config.withCredentials(true);
        });
    }
}
```

#### Request-Specific Configuration

Configure individual requests using the `createRequest` method:

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
    }

    customRequest() {
        return this.http.createRequest('api/data')
            .asGet()
            .withBaseUrl('https://api.example.com')
            .withParams({ category: 'books', page: 1 })
            .withHeader('Authorization', 'Bearer ' + token)
            .withTimeout(5000)
            .send();
    }
}
```

#### Available Configuration Options

The configuration API provides several chainable methods:

| Method                           | Description                                    |
| -------------------------------- | ---------------------------------------------- |
| `withBaseUrl(url)`               | Sets the base URL for requests                 |
| `withHeader(key, value)`         | Adds a header to the request                   |
| `withCredentials(value)`         | Enables/disables credentials                   |
| `withTimeout(value)`             | Sets request timeout in milliseconds           |
| `withParams(params)`             | Adds query parameters                          |
| `withResponseType(type)`         | Sets the expected response type                |
| `withReviver(reviver)`           | Sets a reviver function for JSON parsing       |
| `withReplacer(replacer)`         | Sets a replacer function for JSON stringifying |
| `withProgressCallback(callback)` | Sets a progress callback function              |

### Request Building

The RequestBuilder provides a fluent API for creating and configuring individual HTTP requests with fine-grained control.

#### Using RequestBuilder

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
    }

    complexRequest() {
        return this.http.createRequest('users')
            .asGet()                                    // Set HTTP method
            .withBaseUrl('https://api.example.com')     // Set base URL
            .withParams({ 
                page: 1, 
                limit: 10, 
                sort: 'desc' 
            })                                          // Add query parameters
            .withHeader('X-Custom-Header', 'value')     // Add custom header
            .withTimeout(30000)                         // Set timeout
            .withCredentials(true)                      // Enable credentials
            .send();                                    // Send the request
    }
}
```

#### Available Request Builder Methods

| Category             | Methods                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------- |
| HTTP Methods         | `asDelete()`, `asGet()`, `asHead()`, `asOptions()`, `asPatch()`, `asPost()`, `asPut()`, `asJsonp()` |
| URL & Content        | `withUrl()`, `withBaseUrl()`, `withContent()`, `withParams()`                                       |
| Headers & Auth       | `withHeader()`, `withCredentials()`                                                                 |
| Response Handling    | `withResponseType()`, `withReviver()`, `withReplacer()`                                             |
| Callbacks & Timeouts | `withProgressCallback()`, `withTimeout()`, `withCallbackParameterName()`                            |

### Response Handling

The HTTP client provides comprehensive response handling capabilities through the `HttpResponseMessage` object.

#### Response Message Structure

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
    }

    handleResponse() {
        return this.http.get('api/data')
            .then(response => {
                if (response.isSuccess) {
                    // Response properties
                    const {
                        content,        // Parsed response content
                        response,       // Raw response
                        responseType,   // Response type
                        statusCode,     // HTTP status code
                        statusText,     // Status text
                        headers,        // Response headers
                        requestMessage  // Original request
                    } = response;

                    return content;
                }
                throw new Error(`Request failed: ${response.statusText}`);
            });
    }
}
```

#### Error Handling

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
    }

    handleErrors() {
        return this.http.get('api/data')
            .then(response => {
                if (!response.isSuccess) {
                    throw new Error(`Server returned ${response.statusCode}`);
                }
                return response.content;
            })
            .catch(error => {
                // Handle network errors or thrown errors
                console.error('Request failed:', error);
                throw error;
            });
    }
}
```

### Interceptors

Interceptors allow you to modify requests and responses globally, useful for adding authentication, logging, or error handling.

#### Creating and Configuring Interceptors

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;

        this.http.configure(config => {
            config.withInterceptor({
                request(request) {
                    console.log(`Requesting ${request.url}`);
                    // Add timestamp to all requests
                    request.headers.set('X-Timestamp', new Date().getTime());
                    return request;
                },

                requestError(error) {
                    console.error('Request error:', error);
                    throw error;
                },

                response(response) {
                    console.log(`Received ${response.statusCode} from ${response.requestMessage.url}`);
                    return response;
                },

                responseError(error) {
                    console.error('Response error:', error);
                    throw error;
                }
            });
        });
    }
}
```

#### Authentication Interceptor Example

```javascript
const authInterceptor = {
    request(request) {
        const token = localStorage.getItem('auth_token');
        if (token) {
            request.headers.set('Authorization', `Bearer ${token}`);
        }
        return request;
    }
};

@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
        
        this.http.configure(config => {
            config.withInterceptor(authInterceptor);
        });
    }
}
```

{% hint style="warning" %}
Interceptors are called in the order they are added. Each interceptor in the chain receives the result of the previous interceptor.
{% endhint %}

#### Multiple Interceptors

```javascript
@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;

        this.http.configure(config => {
            // Add multiple interceptors
            config
                .withInterceptor(authInterceptor)
                .withInterceptor(loggingInterceptor)
                .withInterceptor(errorHandlingInterceptor);
        });
    }
}
```

These interceptors can be used to implement cross-cutting concerns like:

* Authentication
* Logging
* Error handling
* Request/Response transformation
* Performance monitoring
* Cache handling

### Advanced Usage

This section covers advanced scenarios and patterns for using the HTTP client effectively in your Aurelia applications.

#### Working with Different Content Types

```javascript
@inject(HttpClient)
export class ContentService {
    constructor(http) {
        this.http = http;
    }

    // Handling form data
    submitFormData(formData) {
        return this.http.createRequest('api/upload')
            .asPost()
            .withContent(formData)
            .withHeader('Content-Type', 'multipart/form-data')
            .send();
    }

    // Handling plain text
    fetchTextContent() {
        return this.http.createRequest('api/text')
            .asGet()
            .withResponseType('text')
            .send();
    }

    // Handling binary data
    downloadFile() {
        return this.http.createRequest('api/file')
            .asGet()
            .withResponseType('blob')
            .send()
            .then(response => {
                const blob = new Blob([response.response], { 
                    type: response.headers.get('content-type') 
                });
                return blob;
            });
    }
}
```

#### Progress Tracking

```javascript
@inject(HttpClient)
export class UploadService {
    constructor(http) {
        this.http = http;
    }

    uploadWithProgress(file) {
        const formData = new FormData();
        formData.append('file', file);

        return this.http.createRequest('api/upload')
            .asPost()
            .withContent(formData)
            .withProgressCallback(progress => {
                if (progress.lengthComputable) {
                    const percentComplete = (progress.loaded / progress.total) * 100;
                    console.log(`Upload progress: ${percentComplete}%`);
                }
            })
            .send();
    }
}
```

#### Timeout Handling

```javascript
@inject(HttpClient)
export class TimeoutService {
    constructor(http) {
        this.http = http;
    }

    fetchWithTimeout() {
        return this.http.createRequest('api/slow-endpoint')
            .asGet()
            .withTimeout(5000) // 5 seconds timeout
            .send()
            .catch(error => {
                if (error.name === 'TimeoutError') {
                    console.log('Request timed out');
                }
                throw error;
            });
    }
}
```

#### Retry Logic with Interceptors

```javascript
class RetryInterceptor {
    constructor(maxRetries = 3) {
        this.maxRetries = maxRetries;
    }

    response(message) {
        return message;
    }

    responseError(error, retryCount = 0) {
        if (retryCount < this.maxRetries && this.shouldRetry(error)) {
            retryCount++;
            console.log(`Retrying request (${retryCount}/${this.maxRetries})`);
            
            return new Promise(resolve => {
                setTimeout(() => {
                    resolve(error.requestMessage.send()
                        .catch(err => this.responseError(err, retryCount)));
                }, this.getDelayTime(retryCount));
            });
        }
        
        throw error;
    }

    shouldRetry(error) {
        return error.statusCode === 503 || error.statusCode === 429;
    }

    getDelayTime(retryCount) {
        return Math.pow(2, retryCount) * 1000; // Exponential backoff
    }
}

@inject(HttpClient)
export class ApiService {
    constructor(http) {
        this.http = http;
        
        this.http.configure(config => {
            config.withInterceptor(new RetryInterceptor());
        });
    }
}
```

#### Caching Responses

```javascript
class CacheInterceptor {
    constructor() {
        this.cache = new Map();
    }

    request(request) {
        if (request.method === 'GET') {
            const cachedResponse = this.cache.get(request.url);
            if (cachedResponse) {
                return Promise.resolve(cachedResponse);
            }
        }
        return request;
    }

    response(response) {
        if (response.requestMessage.method === 'GET') {
            this.cache.set(response.requestMessage.url, response);
        }
        return response;
    }
}

@inject(HttpClient)
export class CachingService {
    constructor(http) {
        this.http = http;
        
        this.http.configure(config => {
            config.withInterceptor(new CacheInterceptor());
        });
    }
}
```

#### Authentication with Refresh Token

```javascript
class AuthInterceptor {
    request(request) {
        const token = localStorage.getItem('access_token');
        if (token) {
            request.headers.set('Authorization', `Bearer ${token}`);
        }
        return request;
    }

    responseError(error) {
        if (error.statusCode === 401) {
            return this.refreshToken()
                .then(newToken => {
                    localStorage.setItem('access_token', newToken);
                    error.requestMessage.headers.set('Authorization', `Bearer ${newToken}`);
                    return error.requestMessage.send();
                })
                .catch(() => {
                    // Handle refresh token failure
                    localStorage.removeItem('access_token');
                    window.location.href = '/login';
                    throw error;
                });
        }
        throw error;
    }

    refreshToken() {
        const refreshToken = localStorage.getItem('refresh_token');
        // Implement token refresh logic here
        return Promise.resolve('new_token');
    }
}
```

{% hint style="warning" %}
Be careful with caching mechanisms as they might lead to stale data. Consider implementing cache invalidation strategies based on your application's needs.
{% endhint %}

These advanced patterns demonstrate handling complex scenarios while keeping your code organized and maintainable. The HTTP client's flexibility allows extensive customization to meet various application requirements.
