# Testing Custom Value Converters and Binding Behaviors

Value converters and binding behaviors are powerful tools in Aurelia for transforming and modifying data during binding. Testing these requires specific strategies to ensure their reliability and correct functionality.

### Value Converter Testing

#### What are Value Converters?

Value converters are used to transform data between the view and view-model:

* Convert data formatting
* Transform data representations
* Perform custom data manipulations

#### Basic Value Converter Example and Test

```typescript
// Value Converter
export class UppercaseValueConverter {
  toView(value: string): string {
    return value ? value.toUpperCase() : '';
  }

  fromView(value: string): string {
    return value ? value.toLowerCase() : '';
  }
}

// Testing Value Converter
describe('UppercaseValueConverter', () => {
  let converter: UppercaseValueConverter;

  beforeEach(() => {
    converter = new UppercaseValueConverter();
  });

  it('should convert string to uppercase', () => {
    expect(converter.toView('hello')).toBe('HELLO');
    expect(converter.toView('')).toBe('');
    expect(converter.toView(null)).toBe('');
  });

  it('should convert string to lowercase from view', () => {
    expect(converter.fromView('HELLO')).toBe('hello');
    expect(converter.fromView('')).toBe('');
    expect(converter.fromView(null)).toBe('');
  });
});
```

#### Complex Value Converter Testing

```typescript
// Date Formatting Value Converter
export class DateFormatValueConverter {
  toView(value: Date, format: string = 'YYYY-MM-DD'): string {
    if (!value) return '';
    return moment(value).format(format);
  }

  fromView(value: string, format: string = 'YYYY-MM-DD'): Date {
    return value ? moment(value, format).toDate() : null;
  }
}

describe('DateFormatValueConverter', () => {
  let converter: DateFormatValueConverter;

  beforeEach(() => {
    converter = new DateFormatValueConverter();
  });

  it('should format date to specified format', () => {
    const testDate = new Date('2023-11-25');
    
    // Default format
    expect(converter.toView(testDate))
      .toBe('2023-11-25');
    
    // Custom format
    expect(converter.toView(testDate, 'DD/MM/YYYY'))
      .toBe('25/11/2023');
  });

  it('should parse date from string', () => {
    const dateString = '25/11/2023';
    const parsedDate = converter.fromView(dateString, 'DD/MM/YYYY');
    
    expect(parsedDate).toEqual(new Date('2023-11-25'));
  });

  it('handles null and undefined inputs', () => {
    expect(converter.toView(null)).toBe('');
    expect(converter.fromView(null)).toBeNull();
  });
});
```

### Binding Behavior Testing

#### What are Binding Behaviors?

Binding behaviors modify how binding works:

* Debounce user input
* Trigger specific binding actions
* Modify binding performance

#### Simple Binding Behavior Example

```typescript
// Debounce Binding Behavior
export class DebounceBindingBehavior {
  bind(binding, source, delay = 250) {
    // Implement debounce logic
    let timeout;
    const originalHandler = binding.handleChange;

    binding.handleChange = () => {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        originalHandler.call(binding);
      }, delay);
    };

    return {
      dispose() {
        clearTimeout(timeout);
        binding.handleChange = originalHandler;
      }
    };
  }
}

describe('DebounceBindingBehavior', () => {
  let behavior: DebounceBindingBehavior;
  let mockBinding;
  let jasmineClock;

  beforeEach(() => {
    behavior = new DebounceBindingBehavior();
    
    // Create mock binding
    mockBinding = {
      handleChange: jasmine.createSpy('handleChange')
    };

    // Setup Jasmine clock for timing tests
    jasmineClock = jasmine.clock();
    jasmineClock.install();
  });

  afterEach(() => {
    jasmineClock.uninstall();
  });

  it('should debounce handler calls', () => {
    // Bind behavior
    const result = behavior.bind(mockBinding, null, 100);

    // Trigger multiple changes
    mockBinding.handleChange();
    mockBinding.handleChange();
    mockBinding.handleChange();

    // Initially, no calls should be made
    expect(mockBinding.handleChange.calls.count()).toBe(0);

    // Fast-forward time
    jasmineClock.tick(150);

    // Only one call should be made
    expect(mockBinding.handleChange.calls.count()).toBe(1);
  });

  it('should dispose correctly', () => {
    const disposable = behavior.bind(mockBinding, null, 100);
    
    // Dispose the binding behavior
    disposable.dispose();

    // Trigger changes
    mockBinding.handleChange();
    jasmineClock.tick(150);

    // No calls should be made after disposal
    expect(mockBinding.handleChange.calls.count()).toBe(0);
  });
});
```

### Component Integration Testing

```typescript
describe('Value Converter in Component', () => {
  let component;

  beforeEach(() => {
    component = StageComponent
      .withResources([
        PLATFORM.moduleName('uppercase-value-converter'),
        PLATFORM.moduleName('my-component')
      ])
      .inView(`
        <div>${'hello' | uppercase}</div>
      `);
  });

  it('should apply value converter in view', done => {
    component.create(bootstrap).then(() => {
      const element = document.querySelector('div');
      expect(element.textContent).toBe('HELLO');
      done();
    });
  });
});
```
