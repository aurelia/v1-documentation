# Computed properties

Computed properties in Aurelia allow you to create derived values that automatically update when their dependencies change. They're implemented using JavaScript getter methods and can be optimized using the `@computedFrom` decorator for enhanced performance.

## Understanding Computed Properties

Computed properties are JavaScript getter methods that derive their values from other properties. Unlike regular properties, they're calculated on-demand and can automatically update when their underlying dependencies change.

### Basic Computed Property Example

```javascript
export class Person {
  firstName = 'John';
  lastName = 'Doe';

  get fullName() {
    console.log('Calculating full name...'); // This runs every time fullName is accessed
    return `${this.firstName} ${this.lastName}`;
  }
}
```

```html
<template>
  <div>
    <p>Full Name: ${fullName}</p>
    <input value.bind="firstName" placeholder="First Name">
    <input value.bind="lastName" placeholder="Last Name">
  </div>
</template>
```

In this example, `fullName` automatically updates whenever `firstName` or `lastName` changes, and the UI reflects these changes immediately.

## Performance Considerations

### The Dirty Checking Problem

By default, Aurelia uses "dirty checking" to detect changes in computed properties. This means:

- Aurelia periodically checks property values (approximately every 120ms)
- Getter functions are executed multiple times during each check cycle
- Complex calculations can significantly impact performance
- The system doesn't know which properties the getter depends on

### Dirty Checking Example

```javascript
export class ExpensiveCalculation {
  items = [1, 2, 3, 4, 5];
  multiplier = 2;

  get expensiveTotal() {
    console.log('Performing expensive calculation...'); // Called frequently
    
    // Simulate expensive operation
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += this.items.reduce((sum, item) => sum + (item * this.multiplier), 0);
    }
    
    return result;
  }
}
```

Without optimization, this getter runs multiple times per second, causing performance issues.

## Optimizing with @computedFrom

The `@computedFrom` decorator solves the dirty checking performance problem by explicitly declaring dependencies.

### Importing and Basic Usage

```javascript
import { computedFrom } from 'aurelia-framework';

export class OptimizedPerson {
  firstName = 'John';
  lastName = 'Doe';

  @computedFrom('firstName', 'lastName')
  get fullName() {
    console.log('Computing full name...'); // Only runs when firstName or lastName changes
    return `${this.firstName} ${this.lastName}`;
  }
}
```

### Benefits of @computedFrom

- **Eliminates dirty checking**: Only recalculates when dependencies change
- **Improves performance**: Especially beneficial for complex calculations
- **Explicit dependencies**: Makes code more maintainable and predictable
- **Automatic optimization**: Aurelia's binding system becomes more efficient

## Advanced @computedFrom Usage

### Multiple Dependencies

```javascript
import { computedFrom } from 'aurelia-framework';

export class ShoppingCart {
  items = [
    { name: 'Apple', price: 1.50, quantity: 3 },
    { name: 'Banana', price: 0.80, quantity: 5 },
    { name: 'Orange', price: 2.00, quantity: 2 }
  ];
  
  taxRate = 0.08;
  discount = 0.10;

  @computedFrom('items', 'taxRate', 'discount')
  get total() {
    const subtotal = this.items.reduce((sum, item) => 
      sum + (item.price * item.quantity), 0
    );
    
    const discountedAmount = subtotal * (1 - this.discount);
    const totalWithTax = discountedAmount * (1 + this.taxRate);
    
    return totalWithTax.toFixed(2);
  }

  @computedFrom('items')
  get itemCount() {
    return this.items.reduce((sum, item) => sum + item.quantity, 0);
  }

  @computedFrom('total')
  get formattedTotal() {
    return `$${this.total}`;
  }
}
```

```html
<template>
  <div class="shopping-cart">
    <h3>Shopping Cart (${itemCount} items)</h3>
    
    <ul>
      <li repeat.for="item of items">
        ${item.name}: $${item.price} × ${item.quantity} = $${(item.price * item.quantity).toFixed(2)}
      </li>
    </ul>
    
    <div class="cart-summary">
      <p>Tax Rate: ${(taxRate * 100).toFixed(1)}%</p>
      <p>Discount: ${(discount * 100).toFixed(1)}%</p>
      <h4>Total: ${formattedTotal}</h4>
    </div>
    
    <div class="controls">
      <input type="number" value.bind="taxRate" step="0.01" min="0" max="1">
      <input type="number" value.bind="discount" step="0.01" min="0" max="1">
    </div>
  </div>
</template>
```

### Nested Property Dependencies

For complex object structures, you can specify nested property paths:

```javascript
import { computedFrom } from 'aurelia-framework';

export class UserProfile {
  user = {
    profile: {
      firstName: 'Jane',
      lastName: 'Smith'
    },
    preferences: {
      displayFormat: 'formal'
    }
  };

  @computedFrom('user.profile.firstName', 'user.profile.lastName', 'user.preferences.displayFormat')
  get displayName() {
    const { firstName, lastName } = this.user.profile;
    const format = this.user.preferences.displayFormat;
    
    switch (format) {
      case 'formal':
        return `${lastName}, ${firstName}`;
      case 'casual':
        return `${firstName} ${lastName}`;
      case 'first':
        return firstName;
      default:
        return `${firstName} ${lastName}`;
    }
  }
}
```

### Array Dependencies and Collection Changes

When depending on arrays, `@computedFrom` observes the array reference, not individual items:

```javascript
import { computedFrom } from 'aurelia-framework';

export class TaskList {
  tasks = [
    { id: 1, title: 'Buy groceries', completed: false },
    { id: 2, title: 'Walk the dog', completed: true },
    { id: 3, title: 'Read a book', completed: false }
  ];

  @computedFrom('tasks')
  get completedTasks() {
    // This updates when tasks array changes (items added/removed)
    // but NOT when individual task properties change
    return this.tasks.filter(task => task.completed);
  }

  @computedFrom('tasks')
  get progress() {
    if (this.tasks.length === 0) return 0;
    const completed = this.tasks.filter(task => task.completed).length;
    return Math.round((completed / this.tasks.length) * 100);
  }

  // Methods that trigger computed property updates
  addTask(title) {
    this.tasks.push({
      id: Date.now(),
      title,
      completed: false
    });
    // This triggers updates because the tasks array reference changes
  }

  removeTask(taskId) {
    const index = this.tasks.findIndex(task => task.id === taskId);
    if (index > -1) {
      this.tasks.splice(index, 1);
      // This triggers updates because the tasks array changes
    }
  }

  // This WON'T trigger computed property updates automatically
  toggleTaskBad(taskId) {
    const task = this.tasks.find(task => task.id === taskId);
    if (task) {
      task.completed = !task.completed; // Modifying object property
      // Computed properties won't update automatically
    }
  }

  // This WILL trigger updates because we modify the array
  toggleTaskGood(taskId) {
    const index = this.tasks.findIndex(task => task.id === taskId);
    if (index > -1) {
      // Create a new object to trigger change detection
      this.tasks.splice(index, 1, {
        ...this.tasks[index],
        completed: !this.tasks[index].completed
      });
    }
  }
}
```

## Complex Computed Property Examples

### Financial Calculator

```javascript
import { computedFrom } from 'aurelia-framework';

export class MortgageCalculator {
  principal = 300000; // Loan amount
  annualRate = 4.5;   // Annual interest rate (%)
  termYears = 30;     // Loan term in years

  @computedFrom('principal', 'annualRate', 'termYears')
  get monthlyPayment() {
    const monthlyRate = (this.annualRate / 100) / 12;
    const numPayments = this.termYears * 12;
    
    if (monthlyRate === 0) {
      return this.principal / numPayments;
    }
    
    const payment = this.principal * 
      (monthlyRate * Math.pow(1 + monthlyRate, numPayments)) /
      (Math.pow(1 + monthlyRate, numPayments) - 1);
    
    return Math.round(payment * 100) / 100;
  }

  @computedFrom('monthlyPayment', 'termYears')
  get totalPayments() {
    return this.monthlyPayment * this.termYears * 12;
  }

  @computedFrom('totalPayments', 'principal')
  get totalInterest() {
    return this.totalPayments - this.principal;
  }

  @computedFrom('totalInterest', 'principal')
  get interestPercentage() {
    return Math.round((this.totalInterest / this.principal) * 100);
  }
}
```

### Data Analysis Dashboard

```javascript
import { computedFrom } from 'aurelia-framework';

export class SalesAnalytics {
  salesData = [
    { month: 'Jan', revenue: 45000, expenses: 32000 },
    { month: 'Feb', revenue: 52000, expenses: 35000 },
    { month: 'Mar', revenue: 48000, expenses: 33000 },
    { month: 'Apr', revenue: 61000, expenses: 38000 },
    { month: 'May', revenue: 58000, expenses: 36000 },
    { month: 'Jun', revenue: 67000, expenses: 41000 }
  ];

  @computedFrom('salesData')
  get totalRevenue() {
    return this.salesData.reduce((sum, month) => sum + month.revenue, 0);
  }

  @computedFrom('salesData')
  get totalExpenses() {
    return this.salesData.reduce((sum, month) => sum + month.expenses, 0);
  }

  @computedFrom('totalRevenue', 'totalExpenses')
  get netProfit() {
    return this.totalRevenue - this.totalExpenses;
  }

  @computedFrom('netProfit', 'totalRevenue')
  get profitMargin() {
    return ((this.netProfit / this.totalRevenue) * 100).toFixed(2);
  }

  @computedFrom('salesData')
  get averageMonthlyRevenue() {
    return Math.round(this.totalRevenue / this.salesData.length);
  }

  @computedFrom('salesData')
  get bestPerformingMonth() {
    return this.salesData.reduce((best, current) => 
      current.revenue > best.revenue ? current : best
    );
  }

  @computedFrom('salesData')
  get monthlyProfits() {
    return this.salesData.map(month => ({
      month: month.month,
      profit: month.revenue - month.expenses,
      margin: ((month.revenue - month.expenses) / month.revenue * 100).toFixed(1)
    }));
  }
}
```

## Working with Observable Properties

Combine `@computedFrom` with `@observable` for reactive computed properties:

```javascript
import { computedFrom, observable } from 'aurelia-framework';

export class ReactiveProfile {
  @observable firstName = 'John';
  @observable lastName = 'Doe';
  @observable title = 'Mr.';

  @computedFrom('title', 'firstName', 'lastName')
  get formalName() {
    return `${this.title} ${this.firstName} ${this.lastName}`;
  }

  // Change handlers for observables
  firstNameChanged(newValue, oldValue) {
    console.log(`First name changed from "${oldValue}" to "${newValue}"`);
  }

  lastNameChanged(newValue, oldValue) {
    console.log(`Last name changed from "${oldValue}" to "${newValue}"`);
  }
}
```

## Best Practices

### 1. Always Use @computedFrom for Non-Trivial Getters

```javascript
// ❌ Bad - Will cause performance issues
get complexCalculation() {
  return this.largeDataset.map(item => 
    this.expensiveTransformation(item)
  ).filter(item => 
    this.complexFilter(item)
  );
}

// ✅ Good - Optimized with explicit dependencies
@computedFrom('largeDataset')
get complexCalculation() {
  return this.largeDataset.map(item => 
    this.expensiveTransformation(item)
  ).filter(item => 
    this.complexFilter(item)
  );
}
```

### 2. Be Explicit About All Dependencies

```javascript
// ❌ Bad - Missing dependency
export class Calculator {
  multiplier = 2;
  
  @computedFrom('numbers') // Missing 'multiplier' dependency
  get result() {
    return this.numbers.reduce((sum, n) => sum + (n * this.multiplier), 0);
  }
}

// ✅ Good - All dependencies listed
export class Calculator {
  multiplier = 2;
  
  @computedFrom('numbers', 'multiplier')
  get result() {
    return this.numbers.reduce((sum, n) => sum + (n * this.multiplier), 0);
  }
}
```

### 3. Use Meaningful Names

```javascript
// ❌ Bad - Unclear purpose
@computedFrom('items')
get calc() {
  return this.items.reduce((s, i) => s + i.p * i.q, 0);
}

// ✅ Good - Clear, descriptive name
@computedFrom('items')
get totalPrice() {
  return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}
```

### 4. Chain Computed Properties When Appropriate

```javascript
export class OrderSummary {
  items = [];
  taxRate = 0.08;
  shippingCost = 15.99;

  @computedFrom('items')
  get subtotal() {
    return this.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }

  @computedFrom('subtotal', 'taxRate')
  get tax() {
    return this.subtotal * this.taxRate;
  }

  @computedFrom('subtotal', 'tax', 'shippingCost')
  get total() {
    return this.subtotal + this.tax + this.shippingCost;
  }
}
```

## Common Pitfalls and Solutions

### 1. Forgetting Dependencies

**Problem**: Computed property doesn't update when it should

```javascript
// ❌ This won't update when discount changes
@computedFrom('price')
get discountedPrice() {
  return this.price * (1 - this.discount); // Missing discount dependency
}

// ✅ Fixed version
@computedFrom('price', 'discount')
get discountedPrice() {
  return this.price * (1 - this.discount);
}
```

### 2. Array Item Property Changes

**Problem**: Computed property doesn't update when array item properties change

```javascript
// ❌ Won't update when individual task completion status changes
@computedFrom('tasks')
get completedCount() {
  return this.tasks.filter(task => task.completed).length;
}

// ✅ Solution: Manually signal updates or restructure data
import { BindingSignaler } from 'aurelia-templating-resources';

@inject(BindingSignaler)
export class TaskManager {
  constructor(signaler) {
    this.signaler = signaler;
  }

  @computedFrom('tasks')
  get completedCount() {
    return this.tasks.filter(task => task.completed).length;
  }

  toggleTask(task) {
    task.completed = !task.completed;
    this.signaler.signal('task-updated'); // Manual signal
  }
}
```

### 3. Overly Complex Dependencies

**Problem**: Too many dependencies make code hard to maintain

```javascript
// ❌ Too complex
@computedFrom('user.profile.address.street', 'user.profile.address.city', 'user.profile.address.state', 'user.profile.address.zip', 'displayFormat', 'includeCountry', 'countryCode')
get formattedAddress() {
  // Complex formatting logic
}

// ✅ Better approach - break into smaller pieces
@computedFrom('user.profile.address')
get baseAddress() {
  const addr = this.user.profile.address;
  return `${addr.street}, ${addr.city}, ${addr.state} ${addr.zip}`;
}

@computedFrom('baseAddress', 'displayFormat', 'includeCountry', 'countryCode')
get formattedAddress() {
  // Simpler formatting logic using baseAddress
}
```

## Performance Monitoring

To verify that `@computedFrom` is working correctly, you can add logging:

```javascript
import { computedFrom } from 'aurelia-framework';

export class PerformanceTest {
  items = [];
  
  // Without @computedFrom - will log frequently
  get unoptimizedCount() {
    console.log('Unoptimized getter called');
    return this.items.length;
  }
  
  // With @computedFrom - will only log when items change
  @computedFrom('items')
  get optimizedCount() {
    console.log('Optimized getter called');
    return this.items.length;
  }
}
```

## Summary

Computed properties are a powerful feature in Aurelia that enable reactive, derived data. Key takeaways:

- Use `@computedFrom` for any non-trivial getter method
- Explicitly declare all dependencies to ensure proper updates
- Break complex calculations into smaller, composable computed properties
- Be aware of array dependency limitations
- Monitor performance to ensure optimizations are working

By following these patterns, you'll create maintainable, performant applications with reactive computed properties that automatically keep your UI in sync with your data.