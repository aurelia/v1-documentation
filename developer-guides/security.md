---
description: >-
  Learn how to build secure Aurelia applications with proper authentication,
  authorization, and security best practices for both client-side and server-side
  implementation.
---

# Security

Building secure web applications requires a comprehensive approach that encompasses both client-side and server-side considerations. This guide provides essential security practices and patterns specific to Aurelia applications, helping you protect your users and data.

{% hint style="warning" %}
**Critical Security Principle**: The client cannot be trusted. Always implement primary security measures on the server side, with client-side measures serving as user experience enhancements only.
{% endhint %}

## Core Security Principles

### 1. Server-Side Security First

All critical security decisions must be made on the server side:

- **Authentication**: Verify user identity on the server
- **Authorization**: Control access to resources server-side
- **Data validation**: Validate all inputs on the server
- **Business logic protection**: Keep sensitive logic on the server

### 2. Defense in Depth

Implement multiple layers of security:

- Network security (HTTPS, CORS)
- Application security (authentication, authorization)
- Data security (validation, sanitization)
- Infrastructure security (server hardening, monitoring)

## Authentication Best Practices

### HTTPS Requirements

Always transmit authentication data over HTTPS:

```javascript
// Configure HTTP client for secure communication
import { HttpClient } from 'aurelia-fetch-client';
import { autoinject } from 'aurelia-framework';

@autoinject
export class AuthService {
  constructor(private http: HttpClient) {
    this.http.configure(config => {
      config
        .useStandardConfiguration()
        .withBaseUrl('https://api.yourdomain.com/') // Always use HTTPS
        .withDefaults({
          credentials: 'same-origin',
          headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        });
    });
  }
}
```

### Secure Authentication Implementation

Never send passwords in plain text:

```javascript
export class LoginService {
  async login(username, password) {
    // Hash password client-side before transmission (optional additional security)
    const hashedPassword = await this.hashPassword(password);
    
    try {
      const response = await this.http.fetch('/auth/login', {
        method: 'POST',
        body: JSON.stringify({
          username,
          password: hashedPassword
        })
      });
      
      if (response.ok) {
        const { token, expires } = await response.json();
        this.storeAuthToken(token, expires);
        return true;
      }
      return false;
    } catch (error) {
      console.error('Login failed:', error.message);
      return false;
    }
  }

  private storeAuthToken(token: string, expires: number) {
    // Store in secure, httpOnly cookies when possible
    // Or use secure session storage with appropriate expiration
    sessionStorage.setItem('auth_token', token);
    sessionStorage.setItem('auth_expires', expires.toString());
  }
}
```

### Strong Password Requirements

Implement and enforce strong password policies:

```javascript
export class PasswordValidator {
  static validateStrength(password) {
    const requirements = {
      minLength: password.length >= 12,
      hasUppercase: /[A-Z]/.test(password),
      hasLowercase: /[a-z]/.test(password),
      hasNumbers: /\d/.test(password),
      hasSpecialChars: /[!@#$%^&*(),.?":{}|<>]/.test(password),
      notCommon: !this.isCommonPassword(password)
    };

    const strength = Object.values(requirements).filter(Boolean).length;
    return {
      isValid: strength >= 5,
      strength,
      requirements
    };
  }

  static isCommonPassword(password) {
    const commonPasswords = [
      'password', '123456', 'password123', 'admin', 'qwerty'
      // Add more common passwords
    ];
    return commonPasswords.includes(password.toLowerCase());
  }
}
```

### Rate Limiting and Account Protection

Implement client-side rate limiting as a UX enhancement:

```javascript
export class LoginAttemptTracker {
  private attempts = new Map();
  private readonly maxAttempts = 5;
  private readonly lockoutDuration = 15 * 60 * 1000; // 15 minutes

  canAttemptLogin(username) {
    const userAttempts = this.attempts.get(username);
    if (!userAttempts) return true;

    if (userAttempts.count >= this.maxAttempts) {
      const timeSinceLastAttempt = Date.now() - userAttempts.lastAttempt;
      if (timeSinceLastAttempt < this.lockoutDuration) {
        return false;
      }
      // Reset attempts after lockout period
      this.attempts.delete(username);
    }
    return true;
  }

  recordFailedAttempt(username) {
    const current = this.attempts.get(username) || { count: 0, lastAttempt: 0 };
    this.attempts.set(username, {
      count: current.count + 1,
      lastAttempt: Date.now()
    });
  }

  clearAttempts(username) {
    this.attempts.delete(username);
  }
}
```

## Authorization and Route Protection

### Router Pipeline Authorization

Implement authorization using Aurelia's router pipeline:

```javascript
import { Redirect } from 'aurelia-router';
import { autoinject } from 'aurelia-framework';
import { AuthService } from './auth-service';

@autoinject
export class AuthorizeStep {
  constructor(private auth: AuthService) {}

  run(navigationInstruction, next) {
    const instructions = navigationInstruction.getAllInstructions();
    
    for (let instruction of instructions) {
      const config = instruction.config;
      
      if (config.auth === true) {
        if (!this.auth.isAuthenticated()) {
          return next.cancel(new Redirect('login'));
        }
      }
      
      if (config.roles && config.roles.length > 0) {
        if (!this.auth.hasAnyRole(config.roles)) {
          return next.cancel(new Redirect('unauthorized'));
        }
      }
      
      if (config.permissions && config.permissions.length > 0) {
        if (!this.auth.hasAnyPermission(config.permissions)) {
          return next.cancel(new Redirect('forbidden'));
        }
      }
    }
    
    return next();
  }
}

// Configure in app.js
export class App {
  configureRouter(config, router) {
    config.addPipelineStep('authorize', AuthorizeStep);
    config.map([
      { 
        route: 'admin/*', 
        name: 'admin', 
        moduleId: 'admin/index', 
        auth: true,
        roles: ['admin', 'superuser']
      },
      { 
        route: 'profile', 
        name: 'profile', 
        moduleId: 'user/profile', 
        auth: true 
      }
    ]);
  }
}
```

### Component-Level Authorization

Protect individual components and their functionality:

```javascript
import { autoinject, computedFrom } from 'aurelia-framework';
import { AuthService } from './auth-service';

@autoinject
export class SecureComponent {
  constructor(private auth: AuthService) {}

  @computedFrom('auth.currentUser', 'auth.currentUser.roles')
  get canEdit() {
    return this.auth.hasRole('editor') || this.auth.hasRole('admin');
  }

  @computedFrom('auth.currentUser', 'auth.currentUser.roles')
  get canDelete() {
    return this.auth.hasRole('admin');
  }

  @computedFrom('auth.currentUser')
  get canView() {
    return this.auth.isAuthenticated();
  }
}
```

```html
<template>
  <div if.bind="canView">
    <h2>Secure Content</h2>
    
    <button if.bind="canEdit" click.delegate="editItem()">
      Edit
    </button>
    
    <button if.bind="canDelete" click.delegate="deleteItem()">
      Delete
    </button>
  </div>
  
  <div else>
    <p>You must be logged in to view this content.</p>
  </div>
</template>
```

## Cross-Origin Resource Sharing (CORS)

Configure CORS properly for your API endpoints:

```javascript
export class HttpClientConfig {
  configure() {
    // Configure your HTTP client with appropriate CORS settings
    const httpClient = new HttpClient();
    
    httpClient.configure(config => {
      config
        .withBaseUrl('https://api.yourdomain.com/')
        .withDefaults({
          credentials: 'include', // Include cookies for authenticated requests
          mode: 'cors',
          headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json',
            'X-Requested-With': 'XMLHttpRequest' // CSRF protection
          }
        })
        .withInterceptor({
          request(request) {
            // Add authentication tokens
            const token = this.getAuthToken();
            if (token) {
              request.headers.set('Authorization', `Bearer ${token}`);
            }
            return request;
          },
          
          response(response) {
            if (response.status === 401) {
              // Handle unauthorized responses
              this.redirectToLogin();
            }
            return response;
          }
        });
    });
  }
}
```

## Client-Side Security Considerations

### Input Validation and Sanitization

Always validate user inputs client-side for UX, but remember server-side validation is mandatory:

```javascript
export class InputValidator {
  static sanitizeHtml(input) {
    // Basic HTML sanitization - use a proper library like DOMPurify for production
    return input
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#x27;')
      .replace(/\//g, '&#x2F;');
  }

  static validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email) && email.length <= 254;
  }

  static validateInput(input, rules) {
    const errors = [];
    
    if (rules.required && (!input || input.trim().length === 0)) {
      errors.push('This field is required');
    }
    
    if (rules.maxLength && input.length > rules.maxLength) {
      errors.push(`Maximum length is ${rules.maxLength} characters`);
    }
    
    if (rules.pattern && !rules.pattern.test(input)) {
      errors.push('Invalid format');
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }
}
```

### Avoiding innerHTML Binding Vulnerabilities

Be extremely cautious with HTML content binding:

```javascript
// DANGEROUS - Never do this with user content
export class UnsafeComponent {
  userContent = '<script>alert("XSS")</script>'; // Malicious content
}
```

```html
<!-- DANGEROUS - Avoid innerHTML with user content -->
<div innerhtml.bind="userContent"></div>
```

```javascript
// SAFE - Always sanitize HTML content
import DOMPurify from 'dompurify';

export class SafeComponent {
  userContent = '<script>alert("XSS")</script>';
  
  get sanitizedContent() {
    return DOMPurify.sanitize(this.userContent);
  }
}
```

```html
<!-- SAFE - Use sanitized content -->
<div innerhtml.bind="sanitizedContent"></div>

<!-- SAFER - Use textContent for plain text -->
<div textcontent.bind="userContent"></div>
```

### Secure Data Storage

Handle sensitive data storage carefully:

```javascript
export class SecureStorage {
  // Avoid storing sensitive data in localStorage (persistent)
  // Use sessionStorage for temporary, session-based data
  static storeTemporary(key, value, encrypted = false) {
    const data = encrypted ? this.encrypt(value) : value;
    sessionStorage.setItem(key, JSON.stringify(data));
  }

  static retrieveTemporary(key, encrypted = false) {
    const data = sessionStorage.getItem(key);
    if (!data) return null;
    
    const parsed = JSON.parse(data);
    return encrypted ? this.decrypt(parsed) : parsed;
  }

  // Never store sensitive data like passwords, private keys, or tokens
  static isDataSensitive(key) {
    const sensitivePatterns = [
      /password/i, /token/i, /key/i, /secret/i, /auth/i
    ];
    return sensitivePatterns.some(pattern => pattern.test(key));
  }

  private static encrypt(data) {
    // Implement proper encryption - this is a placeholder
    // Consider using Web Crypto API for client-side encryption
    return btoa(JSON.stringify(data));
  }

  private static decrypt(data) {
    // Implement proper decryption
    return JSON.parse(atob(data));
  }
}
```

## Deployment Security

### Bundling and Minification

Protect your source code in production:

```javascript
// aurelia.json configuration
{
  "build": {
    "targets": [
      {
        "id": "web",
        "displayName": "Web",
        "output": "scripts",
        "index": "index.html",
        "baseDir": ".",
        "minify": "stage & prod", // Minify in staging and production
        "sourcemap": "stage & prod" // Generate sourcemaps for debugging
      }
    ]
  }
}
```

### Environment-Specific Configuration

Use different configurations for different environments:

```javascript
// config/environment.ts
export default {
  debug: false, // Disable debug in production
  testing: false,
  apiUrl: 'https://api.production.com',
  logLevel: 'error', // Only log errors in production
  auth: {
    tokenExpiry: 3600, // 1 hour
    refreshThreshold: 300 // Refresh 5 minutes before expiry
  }
};
```

### Content Security Policy (CSP)

Implement CSP headers to prevent XSS attacks:

```html
<!-- Add to your index.html -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline'; 
               style-src 'self' 'unsafe-inline'; 
               img-src 'self' data: https:; 
               font-src 'self' https:; 
               connect-src 'self' https://api.yourdomain.com">
```

## Logging and Monitoring

### Security Event Logging

Log security-relevant events for monitoring:

```javascript
export class SecurityLogger {
  static logLoginAttempt(username, success, ip) {
    console.info(`Login attempt: ${username}, Success: ${success}, IP: ${ip}`);
    
    // Send to logging service
    this.sendToLogger({
      event: 'login_attempt',
      username,
      success,
      ip,
      timestamp: new Date().toISOString()
    });
  }

  static logAuthorizationFailure(resource, user, reason) {
    console.warn(`Authorization failed: ${user} -> ${resource}, Reason: ${reason}`);
    
    this.sendToLogger({
      event: 'authorization_failure',
      resource,
      user,
      reason,
      timestamp: new Date().toISOString()
    });
  }

  private static sendToLogger(data) {
    // Implement secure logging to your backend
    // Never log sensitive data like passwords or tokens
    fetch('/api/logs/security', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    }).catch(error => {
      console.error('Failed to send security log:', error);
    });
  }
}
```

### Error Handling

Handle errors securely without exposing sensitive information:

```javascript
export class ErrorHandler {
  handleError(error, context = '') {
    // Log detailed error information securely
    console.error(`Error in ${context}:`, error);
    
    // Send to monitoring service
    this.reportError(error, context);
    
    // Return generic error message to user
    return this.getUserFriendlyMessage(error);
  }

  private getUserFriendlyMessage(error) {
    // Never expose internal error details to users
    const genericMessages = {
      'network': 'A network error occurred. Please try again.',
      'auth': 'Authentication failed. Please check your credentials.',
      'permission': 'You do not have permission to perform this action.',
      'validation': 'Please check your input and try again.',
      'default': 'An unexpected error occurred. Please try again later.'
    };

    return genericMessages[this.categorizeError(error)] || genericMessages.default;
  }

  private categorizeError(error) {
    if (error.status === 401) return 'auth';
    if (error.status === 403) return 'permission';
    if (error.status === 400) return 'validation';
    if (!navigator.onLine) return 'network';
    return 'default';
  }
}
```

## Security Checklist

Use this checklist to ensure your Aurelia application follows security best practices:

### Authentication & Authorization
- [ ] All authentication happens over HTTPS
- [ ] Passwords are never stored or transmitted in plain text
- [ ] Strong password policies are enforced
- [ ] Account lockout mechanisms are implemented
- [ ] Session tokens have appropriate expiration times
- [ ] Authorization is implemented server-side
- [ ] Router pipeline authorization is configured
- [ ] Component-level authorization is implemented where needed

### Data Security
- [ ] All user inputs are validated on both client and server
- [ ] HTML content is properly sanitized
- [ ] SQL injection protection is implemented server-side
- [ ] XSS protection measures are in place
- [ ] CSRF protection is implemented
- [ ] Sensitive data is not stored in client-side storage
- [ ] API endpoints are properly secured

### Infrastructure
- [ ] HTTPS is enforced for all communications
- [ ] CORS is properly configured
- [ ] Content Security Policy headers are implemented
- [ ] Application is properly minified and bundled for production
- [ ] Source maps are not exposed in production
- [ ] Debug information is disabled in production
- [ ] Security headers are configured (HSTS, X-Frame-Options, etc.)

### Monitoring & Response
- [ ] Security events are properly logged
- [ ] Error messages don't expose sensitive information
- [ ] Monitoring and alerting systems are in place
- [ ] Incident response procedures are defined
- [ ] Regular security updates and patches are applied
- [ ] Security testing is included in the development process

{% hint style="danger" %}
**Remember**: Security is an ongoing process, not a one-time implementation. Regularly review and update your security measures, stay informed about new vulnerabilities, and conduct security audits of your applications.
{% endhint %}

## Additional Resources

- [OWASP Top 10 Security Risks](https://owasp.org/www-project-top-ten/)
- [Web Security Guidelines](https://infosec.mozilla.org/guidelines/web_security)
- [Content Security Policy Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [JWT Security Best Practices](https://tools.ietf.org/html/rfc8725)