# Internationalization (i18n) Testing

Internationalization (i18n) testing ensures that your Aurelia application correctly handles multiple languages, translations, and locale-specific formatting. This section covers comprehensive strategies for testing i18n components and functionalities.

### Setup and Prerequisites

#### Required Dependencies

```shell
npm install aurelia-i18n i18next
```

#### Basic I18n Configuration for Testing

```javascript
import { I18N } from 'aurelia-i18n';
import Backend from 'i18next-xhr-backend';

class I18nTestHelper {
  static configure(aurelia) {
    const i18n = aurelia.container.get(I18N);
    
    return i18n.setup({
      backend: {
        loadPath: 'locales/{{lng}}/{{ns}}.json'
      },
      lng: 'en',
      attributes: ['t', 'i18n'],
      fallbackLng: 'en',
      debug: false
    });
  }
}
```

### Translation Testing Strategies

#### Testing Basic Translations

```javascript
describe('Translation Functionality', () => {
  let i18n;

  beforeEach(() => {
    // Mock translation resources
    const translations = {
      en: {
        greeting: 'Hello',
        welcome: 'Welcome, {{name}}!'
      },
      fr: {
        greeting: 'Bonjour',
        welcome: 'Bienvenue, {{name}}!'
      }
    };

    // Setup mock i18n instance
    i18n = {
      tr: (key, options) => {
        let lang = this.currentLang || 'en';
        let translation = translations[lang][key];
        
        if (options && options.name) {
          translation = translation.replace('{{name}}', options.name);
        }
        
        return translation;
      },
      setLocale: (lang) => {
        this.currentLang = lang;
      }
    };
  });

  it('should translate basic strings', () => {
    expect(i18n.tr('greeting')).toBe('Hello');
  });

  it('should support interpolation', () => {
    expect(i18n.tr('welcome', { name: 'John' })).toBe('Welcome, John!');
  });

  it('should change language', () => {
    i18n.setLocale('fr');
    expect(i18n.tr('greeting')).toBe('Bonjour');
  });
});
```

#### Testing Component with Translations

```javascript
describe('Localized Component', () => {
  let component;
  let i18n;

  beforeEach(() => {
    // Mock I18N setup
    i18n = {
      tr: (key) => {
        const translations = {
          'en': {
            'welcome': 'Welcome',
            'goodbye': 'Goodbye'
          },
          'es': {
            'welcome': 'Bienvenido',
            'goodbye': 'Adiós'
          }
        };
        
        return translations[this.currentLang || 'en'][key];
      },
      setLocale: (lang) => {
        this.currentLang = lang;
      }
    };

    component = StageComponent
      .withResources(PLATFORM.moduleName('localized-component'))
      .inView('<localized-component></localized-component>')
      .bootstrap(aurelia => {
        // Register mock i18n service
        aurelia.container.registerInstance(I18N, i18n);
      });
  });

  it('renders translations correctly', done => {
    component.create(bootstrap).then(() => {
      const welcomeElement = document.querySelector('.welcome-text');
      expect(welcomeElement.textContent).toBe('Welcome');

      // Change language and verify
      i18n.setLocale('es');
      expect(welcomeElement.textContent).toBe('Bienvenido');

      done();
    });
  });
});
```

### Formatting and Locale Testing

#### Number and Date Formatting

```javascript
describe('Locale Formatting', () => {
  let i18n;

  beforeEach(() => {
    i18n = {
      nf: (value, locale = 'en-US', options = {}) => {
        return new Intl.NumberFormat(locale, options).format(value);
      },
      df: (value, locale = 'en-US', options = {}) => {
        return new Intl.DateTimeFormat(locale, options).format(value);
      }
    };
  });

  it('formats numbers by locale', () => {
    // US locale
    expect(i18n.nf(1234.56, 'en-US')).toBe('1,234.56');
    
    // German locale uses different formatting
    expect(i18n.nf(1234.56, 'de-DE')).toBe('1.234,56');
  });

  it('formats dates by locale', () => {
    const testDate = new Date('2023-01-15');
    
    // US format
    expect(i18n.df(testDate, 'en-US')).toBe('1/15/2023');
    
    // UK format
    expect(i18n.df(testDate, 'en-GB')).toBe('15/01/2023');
  });
});
```

### Pluralization Testing

```javascript
describe('Pluralization', () => {
  let i18n;

  beforeEach(() => {
    i18n = {
      p: (key, count, options) => {
        const translations = {
          'en': {
            'car': '{{count}} car',
            'car_plural': '{{count}} cars'
          }
        };

        const translation = count === 1 
          ? translations['en'][key] 
          : translations['en'][key + '_plural'];
        
        return translation.replace('{{count}}', count);
      }
    };
  });

  it('handles singular and plural forms', () => {
    expect(i18n.p('car', 1)).toBe('1 car');
    expect(i18n.p('car', 5)).toBe('5 cars');
  });
});
```
