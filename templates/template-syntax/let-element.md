# Let element

The `let` element in Aurelia provides a declarative way to create computed values directly in your templates. It's a powerful feature that allows you to define variables and expressions that are automatically updated when their dependencies change, reducing the need for complex view-model logic.

## Understanding the Let Element

The `let` element serves as a template-level variable declaration that:
- Creates computed values that automatically update
- Reduces view-model complexity for display-only calculations
- Provides better template readability and organization
- Offers automatic dependency tracking

### Basic Syntax

```html
<!-- String interpolation syntax -->
<let variable-name="${expression}"></let>

<!-- Binding expression syntax -->
<let variable-name.bind="expression"></let>
```

## Basic Usage Examples

### Simple String Concatenation

```html
<template>
  <let full-name="${firstName} ${lastName}"></let>
  
  <div>
    <input value.bind="firstName" placeholder="First Name">
    <input value.bind="lastName" placeholder="Last Name">
  </div>
  
  <h2>Welcome, ${fullName}!</h2>
</template>
```

### Arithmetic Calculations

```html
<template>
  <let total.bind="price * quantity"></let>
  <let tax.bind="total * 0.08"></let>
  <let final-total.bind="total + tax"></let>
  
  <div class="invoice">
    <p>Price: $${price}</p>
    <p>Quantity: ${quantity}</p>
    <p>Subtotal: $${total.toFixed(2)}</p>
    <p>Tax: $${tax.toFixed(2)}</p>
    <p><strong>Total: $${finalTotal.toFixed(2)}</strong></p>
  </div>
</template>
```

### Conditional Logic

```html
<template>
  <let status-class.bind="isActive ? 'active' : 'inactive'"></let>
  <let status-message="${isActive ? 'Online' : 'Offline'}"></let>
  
  <div class="status ${statusClass}">
    <span class="indicator"></span>
    ${statusMessage}
  </div>
</template>
```

## Advanced Usage Patterns

### Complex Object Manipulation

```html
<template>
  <let user-display.bind="{ 
    name: user.firstName + ' ' + user.lastName,
    initials: user.firstName[0] + user.lastName[0],
    age: new Date().getFullYear() - user.birthYear,
    isMinor: (new Date().getFullYear() - user.birthYear) < 18
  }"></let>
  
  <div class="user-card">
    <div class="avatar">${userDisplay.initials}</div>
    <h3>${userDisplay.name}</h3>
    <p>Age: ${userDisplay.age}</p>
    <span class="badge" if.bind="userDisplay.isMinor">Minor</span>
  </div>
</template>
```

### Array Processing

```html
<template>
  <let filtered-items.bind="items.filter(item => item.category === selectedCategory)"></let>
  <let sorted-items.bind="filteredItems.sort((a, b) => a.name.localeCompare(b.name))"></let>
  <let item-count.bind="sortedItems.length"></let>
  
  <div class="item-list">
    <h3>Items in ${selectedCategory} (${itemCount})</h3>
    <div repeat.for="item of sortedItems" class="item">
      ${item.name} - $${item.price}
    </div>
  </div>
</template>
```

### Date and Time Formatting

```html
<template>
  <let formatted-date.bind="new Date().toLocaleDateString()"></let>
  <let time-ago.bind="Math.round((Date.now() - lastUpdate) / 1000 / 60)"></let>
  <let time-display="${timeAgo < 1 ? 'Just now' : timeAgo + ' minutes ago'}"></let>
  
  <div class="timestamp">
    <span>Today: ${formattedDate}</span>
    <span>Last update: ${timeDisplay}</span>
  </div>
</template>
```

## Binding to View-Model Context

By default, `let` variables are created in the view's override context. To make them available in your view-model, use the `to-binding-context` attribute:

### Default Behavior (Override Context)

```html
<template>
  <let computed-value="${someProperty * 2}"></let>
  <!-- computedValue is available in template but not in view-model -->
</template>
```

```javascript
export class MyViewModel {
  someProperty = 5;
  
  // computedValue is NOT accessible here
  doSomething() {
    console.log(this.computedValue); // undefined
  }
}
```

### Binding to View-Model Context

```html
<template>
  <let to-binding-context computed-value="${someProperty * 2}"></let>
  <!-- computedValue is now available in view-model -->
</template>
```

```javascript
export class MyViewModel {
  someProperty = 5;
  
  // computedValue IS accessible here
  doSomething() {
    console.log(this.computedValue); // 10 (automatically updated)
  }
}
```

### Observable Integration

When binding to the view-model context, you can use `@observable` to react to changes:

```html
<template>
  <let to-binding-context full-name="${firstName} ${lastName}"></let>
  
  <input value.bind="firstName" placeholder="First Name">
  <input value.bind="lastName" placeholder="Last Name">
  <p>Full name: ${fullName}</p>
</template>
```

```javascript
import { observable } from 'aurelia-framework';

export class NameManager {
  firstName = '';
  lastName = '';
  
  @observable fullName;
  
  fullNameChanged(newValue, oldValue) {
    console.log(`Name changed from "${oldValue}" to "${newValue}"`);
    this.updateUserProfile(newValue);
  }
  
  updateUserProfile(name) {
    // Update external services, APIs, etc.
  }
}
```

## Performance Considerations

### Efficient Expressions

```html
<!-- Good: Simple expressions -->
<let display-name="${user.firstName} ${user.lastName}"></let>
<let total-price.bind="price * quantity"></let>

<!-- Avoid: Complex operations that might be expensive -->
<let heavy-calculation.bind="items.map(i => expensiveFunction(i)).reduce((a, b) => a + b, 0)"></let>
```

For expensive calculations, consider using a computed property in your view-model instead:

```javascript
export class ViewModel {
  @computedFrom('items')
  get expensiveCalculation() {
    return this.items.map(i => expensiveFunction(i)).reduce((a, b) => a + b, 0);
  }
}
```

### Dependency Optimization

The `let` element automatically tracks dependencies, but you can optimize by being explicit about what changes:

```html
<!-- Dependencies: firstName, lastName -->
<let full-name="${firstName} ${lastName}"></let>

<!-- Dependencies: price, quantity, taxRate -->
<let final-total.bind="(price * quantity) * (1 + taxRate)"></let>
```

## Real-World Examples

### E-commerce Product Card

```html
<template>
  <let discounted-price.bind="product.price * (1 - (product.discount || 0))"></let>
  <let savings.bind="product.price - discountedPrice"></let>
  <let has-discount.bind="product.discount > 0"></let>
  <let stock-status="${product.stock > 10 ? 'In Stock' : product.stock > 0 ? 'Low Stock' : 'Out of Stock'}"></let>
  <let stock-class.bind="product.stock > 10 ? 'in-stock' : product.stock > 0 ? 'low-stock' : 'out-of-stock'"></let>
  
  <div class="product-card">
    <img src.bind="product.image" alt.bind="product.name">
    <h3>${product.name}</h3>
    
    <div class="pricing">
      <span class="price ${hasDiscount ? 'discounted' : ''}">${discountedPrice.toFixed(2)}</span>
      <span if.bind="hasDiscount" class="original-price">${product.price.toFixed(2)}</span>
      <span if.bind="hasDiscount" class="savings">Save $${savings.toFixed(2)}</span>
    </div>
    
    <div class="stock-status ${stockClass}">
      ${stockStatus}
    </div>
    
    <button class="add-to-cart" 
            disabled.bind="product.stock === 0"
            click.delegate="addToCart(product)">
      ${product.stock === 0 ? 'Sold Out' : 'Add to Cart'}
    </button>
  </div>
</template>
```

### User Dashboard Stats

```html
<template>
  <let total-orders.bind="orders.length"></let>
  <let pending-orders.bind="orders.filter(o => o.status === 'pending').length"></let>
  <let completed-orders.bind="orders.filter(o => o.status === 'completed').length"></let>
  <let total-spent.bind="orders.reduce((sum, order) => sum + order.total, 0)"></let>
  <let avg-order-value.bind="totalOrders > 0 ? totalSpent / totalOrders : 0"></let>
  <let completion-rate.bind="totalOrders > 0 ? (completedOrders / totalOrders * 100) : 0"></let>
  
  <div class="dashboard-stats">
    <div class="stat-card">
      <h4>Total Orders</h4>
      <span class="stat-value">${totalOrders}</span>
    </div>
    
    <div class="stat-card">
      <h4>Pending</h4>
      <span class="stat-value pending">${pendingOrders}</span>
    </div>
    
    <div class="stat-card">
      <h4>Completed</h4>
      <span class="stat-value completed">${completedOrders}</span>
    </div>
    
    <div class="stat-card">
      <h4>Total Spent</h4>
      <span class="stat-value currency">$${totalSpent.toFixed(2)}</span>
    </div>
    
    <div class="stat-card">
      <h4>Avg. Order Value</h4>
      <span class="stat-value currency">$${avgOrderValue.toFixed(2)}</span>
    </div>
    
    <div class="stat-card">
      <h4>Completion Rate</h4>
      <span class="stat-value percentage">${completionRate.toFixed(1)}%</span>
    </div>
  </div>
</template>
```

### Form Validation Summary

```html
<template>
  <let email-valid.bind="email && /^[^@]+@[^@]+\.[^@]+$/.test(email)"></let>
  <let password-strong.bind="password && password.length >= 8 && /[A-Z]/.test(password) && /[0-9]/.test(password)"></let>
  <let passwords-match.bind="password === confirmPassword"></let>
  <let form-valid.bind="emailValid && passwordStrong && passwordsMatch"></let>
  <let validation-summary.bind="[
    { field: 'Email', valid: emailValid, message: 'Valid email required' },
    { field: 'Password', valid: passwordStrong, message: 'Password must be 8+ chars with uppercase and number' },
    { field: 'Confirm', valid: passwordsMatch, message: 'Passwords must match' }
  ]"></let>
  
  <form class="registration-form">
    <input type="email" value.bind="email" placeholder="Email">
    <input type="password" value.bind="password" placeholder="Password">
    <input type="password" value.bind="confirmPassword" placeholder="Confirm Password">
    
    <div class="validation-summary">
      <div repeat.for="validation of validationSummary" 
           class="validation-item ${validation.valid ? 'valid' : 'invalid'}">
        <span class="field">${validation.field}:</span>
        <span class="status">${validation.valid ? '✓' : '✗'}</span>
        <span class="message" if.bind="!validation.valid">${validation.message}</span>
      </div>
    </div>
    
    <button type="submit" 
            disabled.bind="!formValid"
            class="submit-btn">
      Register
    </button>
  </form>
</template>
```

## Integration with Value Converters

You can combine `let` elements with value converters for powerful data transformation:

```html
<template>
  <let formatted-date.bind="currentDate | dateFormat:'MMMM Do, YYYY'"></let>
  <let currency-total.bind="orderTotal | currency:'USD'"></let>
  <let truncated-desc.bind="product.description | truncate:100"></let>
  
  <div class="product-summary">
    <h2>${product.name}</h2>
    <p class="description">${truncatedDesc}</p>
    <p class="price">${currencyTotal}</p>
    <p class="date">Available: ${formattedDate}</p>
  </div>
</template>
```

## Debugging Let Elements

### Development Inspection

```html
<template>
  <let debug-info.bind="{
    firstName: firstName,
    lastName: lastName,
    fullName: firstName + ' ' + lastName,
    timestamp: Date.now()
  }"></let>
  
  <!-- Temporary debugging display -->
  <div if.bind="isDevelopment" class="debug-panel">
    <pre>${debugInfo | json}</pre>
  </div>
</template>
```

### Console Logging in Development

While you can't directly call functions in `let` expressions, you can create helper properties in your view-model:

```javascript
export class DebuggableViewModel {
  get debugLog() {
    return (value, label = 'Debug') => {
      console.log(`${label}:`, value);
      return value; // Return the value so it can be used in expressions
    };
  }
}
```

```html
<template>
  <let logged-value.bind="debugLog(someComplexCalculation, 'Calculation Result')"></let>
  <p>Result: ${loggedValue}</p>
</template>
```

## Best Practices

### 1. Use Let for View-Specific Logic
```html
<!-- Good: Display formatting -->
<let display-price="${price > 0 ? '$' + price.toFixed(2) : 'Free'}"></let>

<!-- Avoid: Business logic that belongs in view-model -->
<let order-status.bind="processOrderPayment(order)"></let>
```

### 2. Keep Expressions Simple and Readable
```html
<!-- Good: Clear and simple -->
<let full-name="${firstName} ${lastName}"></let>

<!-- Avoid: Complex nested expressions -->
<let complex-calc.bind="items.filter(i => i.active).map(i => i.values.reduce((a,b) => a+b)).sort()"></let>
```

### 3. Name Variables Descriptively
```html
<!-- Good: Descriptive names -->
<let discounted-price.bind="originalPrice * (1 - discountRate)"></let>
<let is-premium-user.bind="user.tier === 'premium'"></let>

<!-- Avoid: Unclear abbreviations -->
<let dp.bind="op * (1 - dr)"></let>
<let ipu.bind="u.t === 'premium'"></let>
```

### 4. Consider Performance Impact
```html
<!-- Good: Use let for simple calculations -->
<let total.bind="price + tax"></let>

<!-- Consider view-model computed property for expensive operations -->
<let expensive-result.bind="heavyCalculation()"></let> <!-- Move to view-model -->
```

### 5. Organize Related Let Elements
```html
<template>
  <!-- Group related calculations -->
  <let subtotal.bind="items.reduce((sum, item) => sum + item.price, 0)"></let>
  <let tax-amount.bind="subtotal * taxRate"></let>
  <let shipping.bind="subtotal > 50 ? 0 : 9.99"></let>
  <let total.bind="subtotal + taxAmount + shipping"></let>
  
  <!-- Use computed values -->
  <div class="order-summary">
    <p>Subtotal: $${subtotal.toFixed(2)}</p>
    <p>Tax: $${taxAmount.toFixed(2)}</p>
    <p>Shipping: $${shipping.toFixed(2)}</p>
    <p class="total">Total: $${total.toFixed(2)}</p>
  </div>
</template>
```

The `let` element is a powerful tool for creating clean, maintainable templates with computed values. By understanding its capabilities and limitations, you can create more readable and efficient Aurelia applications with less view-model complexity.