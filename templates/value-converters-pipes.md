# Value converters (pipes)

Value converters in Aurelia are powerful tools that transform data between your view-model and view. They're especially useful for formatting model data for display, but can also handle input conversion in two-way binding scenarios.

If you're familiar with value converters in other frameworks like Xaml, you'll find Aurelia's approach similar but with some neat improvements:

1. Clear method naming: 'toView' and 'fromView' explicitly indicate data flow direction.
2. Data-bound parameters: Unlike Xaml, Aurelia allows dynamic converter parameters.
3. Multi-parameter support: Converters can accept multiple arguments for complex transformations.
4. Composition: Chain multiple converters using the pipe (|) symbol.
5. External trigger support: Use the 'signals' field to update views based on external changes like language or locale.

## Simple converters

Let's enhance our previous example with some basic value converters. We'll leverage Aurelia's capabilities alongside the popular [Moment](https://momentjs.com/) and [Numeral](https://numeraljs.com/) libraries to streamline our code. Here's how we can efficiently implement these improvements:

{% code title="currency-format.js" %}
```javascript
import numeral from 'numeral';
  
export class CurrencyFormatValueConverter {
  toView(value) {
    return numeral(value).format('($0,0.00)');
  }
}
```
{% endcode %}

{% code title="date-format.js" %}
```javascript
import moment from 'moment';

export class DateFormatValueConverter {
  toView(value) {
    return moment(value).format('M/D/YYYY h:mm:ss a');
  }
}
```
{% endcode %}

{% code title="net-worth.js" %}
```javascript
export class NetWorth {
  constructor() {
    this.update();
    setInterval(() => this.update(), 1000);
  }

  update() {
    this.currentDate = new Date();
    this.netWorth = Math.random() * 1000000000;
  }
}
```
{% endcode %}

{% code title="net-worth.html" %}
```markup
<template>
  <require from="./date-format"></require>
  <require from="./currency-format"></require>

  ${currentDate | dateFormat} <br>
  ${netWorth | currencyFormat}
</template>
```
{% endcode %}

{% hint style="info" %}
A working demo of the above code can be found [here](https://gist.dumber.app/?gist=aa754984f15f97f6a1d19d21a5d5d9ee).
{% endhint %}

So, how did we achieve this improved result?

We created two value converters: `DateFormatValueConverter` and `CurrencyFormatValueConverter`. These converters have a toView method that Aurelia applies to model values before displaying them in the view. We leveraged MomentJS and NumeralJS libraries for date and currency formatting, respectively.

Then, we updated the view to include these converters. To use a resource like a value converter, add it to the require element with the appropriate path in the from attribute.

```markup
<require from="./date-format"></require>
<require from="./currency-format"></require>
```

Aurelia processes resources by examining class metadata to determine their type (e.g., custom element, attribute, or value converter). While metadata isn't always necessary, Aurelia uses conventions to simplify development. For instance, export names ending with "ValueConverter" are automatically recognized as value converters. The registration uses the camel-cased export name, minus the "ValueConverter" suffix.

Examples:

* DateFormatValueConverter becomes dateFormat
* CurrencyFormatValueConverter becomes currencyFormat

Use the pipe `(|)` syntax to apply a converter in bindings.

```html
${currentDate | dateFormat} <br>
${netWorth | currencyFormat}
```

{% hint style="info" %}
The name that a resource is referenced by in a view derives from its export name. For Value Converters and Binding Behaviors, the export name is converted to camel case (think of it as a variable name). For Custom Elements and Custom Attributes, the export name is lower-cased and hyphenated (to comply with HTML element and attribute specifications).
{% endhint %}

## Converter parameters

While the previous converters work well, they're limited to a single format. What if we need to display dates and numbers in various ways? Rather than creating a converter for each format, let's make our converters more flexible by accepting a format parameter. This approach allows us to specify the desired format directly in the binding, maximizing the reusability of our converters.

{% code title="numeral-format.js" %}
```javascript
import numeral from 'numeral';

export class NumberFormatValueConverter {
  toView(value, format) {
    return numeral(value).format(format);
  }
}
```
{% endcode %}

{% code title="date-format.js" %}
```javascript
import moment from 'moment';

export class DateFormatValueConverter {
  toView(value, format) {
    return moment(value).format(format);
  }
}

```
{% endcode %}

{% code title="net-worth.js" %}
```javascript
export class NetWorth {
  constructor() {
    this.update();
    setInterval(() => this.update(), 1000);
  }

  update() {
    this.currentDate = new Date();
    this.netWorth = Math.random() * 1000000000;
  }
}
```
{% endcode %}

```markup
<template>
  <require from="./date-format"></require>
  <require from="./number-format"></require>

  ${currentDate | dateFormat:'M/D/YYYY h:mm:ss a'} <br>
  ${currentDate | dateFormat:'MMMM Mo YYYY'} <br>
  ${currentDate | dateFormat:'h:mm:ss a'} <br>
  ${netWorth | numberFormat:'$0,0.00'} <br>
  ${netWorth | numberFormat:'$0.0a'} <br>
  ${netWorth | numberFormat:'0.00000)'}
</template>
```

With the `format` parameter added to the `toView` methods, we can specify the format in the binding using the `[expression] | [converterName]:[parameterExpression]` syntax:

```markup
${currentDate | dateFormat:'MMMM Mo YYYY'} <br>
${netWorth | numberFormat:'$0.0a'} <br>
```

{% hint style="info" %}
A working demo of the above code can be found [here](https://gist.dumber.app/?gist=51935113f75358e7abf616828fa2e7b6).
{% endhint %}

## Binding converter parameters

While you can use literal values for converter parameters, binding them offers more flexibility. This approach allows for dynamic results that can change based on your application's state or user input.

{% code title="number-format.js" %}
```javascript
import numeral from "numeral";

export class NumberFormatValueConverter {
  toView(value, format) {
    return numeral(value).format(format);
  }
}
```
{% endcode %}

{% code title="net-worth.js" %}
```javascript
export class NetWorth {
  constructor() {
    this.update();
    setInterval(() => this.update(), 1000);
  }

  update() {
    this.netWorth = Math.random() * 1000000000;
  }
}
```
{% endcode %}

{% code title="net-worth.html" %}
```markup
<template>
  <require from="./number-format"></require>

  <label for="formatSelect">Select Format:</label>
  <select id="formatSelect" ref="formatSelect">
    <option value="$0,0.00">$0,0.00</option>
    <option value="$0.0a">$0.0a</option>
    <option value="0.00000">0.00000</option>
  </select>

  ${netWorth | numberFormat:formatSelect.value}
</template>
```
{% endcode %}

{% hint style="info" %}
A working demo of the above code can be found [here](https://gist.dumber.app/?gist=51935113f75358e7abf616828fa2e7b6).
{% endhint %}

## Multiple parameters & chaining

Value converters in Aurelia are versatile, accepting multiple parameters and allowing composition within a single binding expression. This design promotes flexibility and reusability.

Consider this example:

Our view-model contains an array of Aurelia repositories. The view uses a repeat binding to display these repos in a table. We apply two value converters:

1. `SortValueConverter`: Sorts the array based on 'propertyName' and 'direction'.
2. `TakeValueConverter`: Limits the number of displayed repositories.

This demonstrates how multiple converters can work together to transform data efficiently.

{% code title="sort.js" %}
```javascript
export class SortValueConverter {
  toView(array, propertyName, direction) {
    let factor = direction === 'ascending' ? 1 : -1;
    return array.slice(0).sort((a, b) => {
      return (a[propertyName] - b[propertyName]) * factor;
    });
  }
}
```
{% endcode %}

{% code title="take.js" %}
```javascript
export class TakeValueConverter {
  toView(array, count) {
    return array.slice(0, count);
  }
}
```
{% endcode %}

{% code title="app.js" %}
```javascript
import {HttpClient} from 'aurelia-http-client';

export class App {
  repos = [];

  activate() {
    return new HttpClient()
      .get('https://api.github.com/orgs/aurelia/repos')
      .then(response => this.repos = response.content);
  }
}
```
{% endcode %}

{% code title="app.html" %}
```markup
<template>
  <require from="./sort"></require>
  <require from="./take"></require>

  <label for="column">Sort By:</label>
  <select id="column" ref="column">
    <option value="stargazers_count">Stars</option>
    <option value="forks_count">Forks</option>
    <option value="open_issues">Issues</option>
  </select>

  <select ref="direction">
    <option value="descending">Descending</option>
    <option value="ascending">Ascending</option>
  </select>

  <table class="table table-striped">
    <thead>
      <tr>
        <th>Name</th>
        <th>Stars</th>
        <th>Forks</th>
        <th>Issues</th>
      </tr>
    </thead>
    <tbody>
      <tr repeat.for="repo of repos | sort:column.value:direction.value | take:10">
        <td>${repo.name}</td>
        <td>${repo.stargazers_count}</td>
        <td>${repo.forks_count}</td>
        <td>${repo.open_issues}</td>
      </tr>
    </tbody>
  </table>
</template>
```
{% endcode %}

{% hint style="info" %}
A working demo of the above code can be found [here](https://gist.dumber.app/?gist=782a2e0dc5dab02da6799b483b20ded6).
{% endhint %}

## Object parameters

Aurelia supports object converter parameters. An alternate implementation of the `SortValueConverter` using a single `config` parameter would look like this:

{% code title="sort.js" %}
```javascript
export class SortValueConverter {
  toView(array, config) {
    let factor = (config.direction || 'ascending') === 'ascending' ? 1 : -1;
    return array.sort((a, b) => {
      return (a[config.propertyName] - b[config.propertyName]) * factor;
    });
  }
}
```
{% endcode %}

{% code title="app.js" %}
```javascript
import {HttpClient} from 'aurelia-http-client';

export class App {
  repos = [];

  activate() {
    return new HttpClient()
      .get('https://api.github.com/orgs/aurelia/repos')
      .then(response => this.repos = response.content);
  }
}
```
{% endcode %}

{% code title="app.html" %}
```markup
<template>
  <require from="./sort"></require>

  <div class="row">
    <div class="col-sm-3"
          repeat.for="repo of repos | sort: { propertyName: 'open_issues', direction: 'descending' }">
      <a href="${repo.html_url}/issues" target="_blank">
        ${repo.name} (${repo.open_issues})
      </a>
    </div>
  </div>
</template>
```
{% endcode %}

This approach has a couple of advantages: you don't need to remember the order of the converter parameter arguments, and anyone reading the markup can easily tell what each converter parameter represents.

{% hint style="info" %}
A working demo of the above code can be found [here](https://gist.dumber.app/?gist=1ddd605a7ba40ae3052b442801a833c3).
{% endhint %}

## Bi-directional value converters

Converters aren't just for displaying data but also crucial when handling user input. While to-view bindings use a converter's toView method, two-way bindings on input elements need the fromView method as well. This method transforms user input into the format your view-model expects.

Let's look at a practical example: binding a color object to an HTML5 color input. Our view-model stores colors as objects with red, green, and blue properties, but the color input requires a hex format. An RgbToHexValueConverter bridges this gap, ensuring smooth data flow in both directions.

{% code title="rgb-to-hex.js" %}
```javascript
export class RgbToHexValueConverter {
  toView(rgb) {
    return "#" + (
      (1 << 24) + (rgb.r << 16) + (rgb.g << 8) + rgb.b
    ).toString(16).slice(1);
  }

  fromView(hex) {
    let exp = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i,
        result = exp.exec(hex);
    return {
      r: parseInt(result[1], 16),
      g: parseInt(result[2], 16),
      b: parseInt(result[3], 16)
    };
  }
}
```
{% endcode %}

{% code title="app.js" %}
```javascript
export class App {
  rgb = { r: 146, g: 39, b: 143 };
}
```
{% endcode %}

{% code title="app.html" %}
```markup
<template>
  <require from="./rgb-to-hex"></require>

  <label for="color">Select Color:</label>
  <input id="color" type="color" value.bind="rgb | rgbToHex">
  <br> r: ${rgb.r}, g:${rgb.g}, b:${rgb.b}
</template>
```
{% endcode %}

{% hint style="info" %}
A working demo of the above code can be found [here](https://gist.dumber.app/?gist=c99fe051feecf7ea20f57de042c51bc0).
{% endhint %}

## Globally accessible value converters

We've been using the `require` element to import converters in our views, but there's a simpler approach. For frequently used value converters, Aurelia's `globalResources` function allows you to register them globally. This eliminates the need to add `require` elements at the top of each view, streamlining your code.

Let's make the `rgb-to-hex` value converter globally accessible:

{% code title="rgb-to-hex.js" %}
```javascript
export class RgbToHexValueConverter {
  toView(rgb) {
    return "#" + (
      (1 << 24) + (rgb.r << 16) + (rgb.g << 8) + rgb.b
    ).toString(16).slice(1);r
  }

  fromView(hex) {
    let exp = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i,
        result = exp.exec(hex);
    return {
      r: parseInt(result[1], 16),
      g: parseInt(result[2], 16),
      b: parseInt(result[3], 16)
    };
  }
}
```
{% endcode %}

Then in our `app.js` file where we bootstrap Aurelia, we use `globalResources` to register our value converter:

```javascript
import {Aurelia} from 'aurelia-framework';
import {PLATFORM} from 'aurelia-pal';

export function configure(aurelia: Aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  // Register global resources (value converters, custom elements, etc.)
  aurelia.use.globalResources([
    PLATFORM.moduleName('resources/value-converters/rgb-to-hex'),
  ]);

  aurelia.start().then(() => aurelia.setRoot(PLATFORM.moduleName('app')));
}
```

## Signalable value converters

Value converters sometimes rely on global parameters that Aurelia can't directly observe, like timezones or hardware changes. In other cases, you might need to simultaneously update all bindings using a specific converter, such as when changing the application's language.

Aurelia provides a signaling mechanism for value converters to address these scenarios. By declaring a 'signals' property on a converter, you can trigger updates for all bindings using that converter.

Consider a flight information display system. The view-model contains a list of flights, and the view shows each flight's time using a clock format. The display format depends on a global `currentLocale` variable. Using signals, you can update all flight displays when the locale changes.

To implement this, use the framework's `signalBindings` export to trigger the value converter's signal, refreshing all related bindings.

```javascript
import {signalBindings} from 'aurelia-framework';

signalBindings('locale-changed');
```

{% code title="flight-time-value-converter.js" %}
```javascript
export class FlightTimeValueConverter {
  signals = ["locale-changed"];

  toView(val) {
    let newVal =
      val instanceof Date
        ? val.toLocaleString(window.currentLocale)
        : val === null
        ? ""
        : val; // eslint-disable-line
    return newVal;
  }
}
```
{% endcode %}

{% code title="app.js" %}
```javascript
import { signalBindings } from "aurelia-binding";

export class App {
  flights;
  locales = [
    { locale: "en-US", name: "US" },
    { locale: "en-GB", name: "UK" },
    { locale: "ko-KR", name: "Korean" },
    { locale: "ar-EG", name: "Arabic" },
    { locale: "ja-JP-u-ca-japanese", name: "Japan" },
    { locale: "de-DE", name: "Germany" },
    { locale: "pt-BR", name: "Brazil" },
    { locale: "ru-RU", name: "Russia" },
    { locale: "es-ES", name: "Spain" },
    { locale: "it-IT", name: "Italy" },
    { locale: "zh-CN", name: "China" },
    { locale: "zh-HK", name: "Hong Kong" },
    { locale: "zh-TW", name: "Taiwan" }
  ];

  changeLocale(locale) {
    window.currentLocale = locale;
    signalBindings("locale-changed");
  }

  constructor() {
    this.flights = [
      {
        from: "Los Angeles",
        to: "San Fran",
        depart: new Date("2017-10-09"),
        arrive: new Date("2017-10-10")
      },
      {
        from: "Melbourne",
        to: "Sydney",
        depart: new Date("2017-10-11"),
        arrive: new Date("2017-10-12")
      },
      {
        from: "Hawaii",
        to: "Crescent",
        depart: new Date("2017-10-13"),
        arrive: new Date("2017-10-14")
      }
    ];
  }
}
```
{% endcode %}

{% code title="clock.html" %}
```markup
<template
  bindable="time"
  style="display: inline-block;
  width: 200px;
  padding: 4px 6px;
  border-radius: 10px;
  border: 2px solid #1e1e1e;
  font-size: 16px;
  text-align: center;"
>
  <require from="./flight-time-value-converter"></require> ${time | flightTime}
</template>
```
{% endcode %}

{% code title="app.html" %}
```markup
<template>
  <require from="./clock.html"></require>

  <div>
    Change locale to
    <div>
      <button
        repeat.for="locale of locales"
        click.delegate="changeLocale(locale.locale)"
        style="margin-right: 5px"
      >
        ${locale.name}
      </button>
    </div>
  </div>

  <div>
    <h2>Flights</h2>
    <table
      repeat.for="flight of flights"
      style="margin-bottom: 15px; color: #5B5B5B"
    >
      <tr>
        <th>From ${flight.from}</th>
        <th style="width: 10px"></th>
        <th>To ${flight.to}</th>
      </tr>
      <tr>
        <td><clock time.bind="flight.depart"></clock></td>
        <td style="width: 10px"></td>
        <td><clock time.bind="flight.arrive"></clock></td>
      </tr>
    </table>
  </div>
</template>
```
{% endcode %}

{% hint style="info" %}
A working demo of the above code can be found [here](https://gist.dumber.app/?gist=d43822bc180f2dc8602ebf435124a9e4).
{% endhint %}
