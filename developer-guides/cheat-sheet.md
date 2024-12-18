---
description: >-
  Forgot the syntax for bindings? Need to know how to create a custom attribute?
  This article contains answers to questions like those as well as a bunch of
  quick snippets for common tasks..
---

# Cheat sheet

### Configuration and Startup <a href="#configuration-and-startup" id="configuration-and-startup"></a>

#### Standard Startup Configuration

{% code overflow="wrap" %}
```javascript
export function configure(aurelia) {
    aurelia.use
      .standardConfiguration()
      .developmentLogging();
  
    aurelia.start().then(() => aurelia.setRoot());
}
```
{% endcode %}

**Explicit Startup Configuration**

```javascript
 
  import {LogManager} from 'aurelia-framework';
  import {ConsoleAppender} from 'aurelia-logging-console';
  
  LogManager.addAppender(new ConsoleAppender());
  LogManager.setLevel(LogManager.logLevel.debug);
  
  export function configure(aurelia) {
    aurelia.use
      .defaultBindingLanguage()
      .defaultResources()
      .history()
      .router()
      .eventAggregator();
  
    aurelia.start().then(() => aurelia.setRoot('app', document.body));
  }
```

#### Configuring a Feature

```javascript
  export function configure(aurelia) {
    aurelia.use
      .standardConfiguration()
      .developmentLogging()
      .feature('feature-name', featureConfiguration);
  
    aurelia.start().then(() => aurelia.setRoot());
  }
```

#### Installing a Plugin

```javascript
  export function configure(aurelia) {
    aurelia.use
      .standardConfiguration()
      .developmentLogging()
      .plugin('plugin-name', pluginConfiguration);
  
    aurelia.start().then(() => aurelia.setRoot());
  }
```

### Creating Components <a href="#creating-components" id="creating-components"></a>

UI components consist of two parts: a view-model and a view. Simply create each part in its own file. Use the same file name but different file extensions for the two parts. For example: _hello.js_ and _hello.html_.

**Explicit Configuration**

```javascript
  import {useView} from 'aurelia-framework';
  
  @useView('./hello.html')
  export class Hello {
    ...
  }
  
```

#### **The Component Lifecycle**

Components have a well-defined lifecycle:

1. `constructor()` - The view-model's constructor is called first.
2. `created(owningView: View, myView: View)` - If the view-model implements the `created` callback it is invoked next. At this point in time, the view has also been created and both the view-model and the view are connected to their controller. The created callback will receive the instance of the "owningView". This is the view that the component is declared inside of. If the component itself has a view, this will be passed second.
3. `bind(bindingContext: Object, overrideContext: Object)` - Databinding is then activated on the view and view-model. If the view-model has a `bind` callback, it will be invoked at this time. The "binding context" to which the component is being bound will be passed first. An "override context" will be passed second. The override context contains information used to traverse the parent hierarchy and can also be used to add any contextual properties the component wants. It should be noted that when the view-model has implemented the `bind` callback, the data-binding framework will not invoke the changed handlers for the view-model's bindable properties until the "next" time those properties are updated. If you need to perform specific post-processing on your bindable properties, when implementing the `bind` callback, you should do so manually within the callback itself.
4. `attached()` - Next, the component is attached to the DOM (in document). If the view-model has an `attached` callback, it will be invoked at this time.
5. `detached()` - At some point in the future, the component may be removed from the DOM. If/When this happens, and if the view-model has a `detached` callback, this is when it will be invoked.
6. `unbind()` - After a component is detached, it's usually unbound. If your view-model has the `unbind` callback, it will be invoked during this process.

### Dependency Injection <a href="#dependency-injection" id="dependency-injection"></a>

**Declaring Dependencies**

```javascript
  import {inject} from 'aurelia-framework';
  import {Dep1} from 'dep1';
  import {Dep2} from 'dep2';
  
  @inject(Dep1, Dep2)
  export class CustomerDetail {
    constructor(dep1, dep2) {
      this.dep1 = dep1;
      this.dep2 = dep2;
    }
  }
```

#### Using Resolvers

```javascript
  import {Lazy, inject} from 'aurelia-framework';
  import {HttpClient} from 'aurelia-fetch-client';
  
  @inject(Lazy.of(HttpClient))
  export class CustomerDetail {
    constructor(getHTTP){
      this.getHTTP = getHTTP;
    }
  }
```

#### **Available Resolvers**

* `Lazy` - Injects a function for lazily evaluating the dependency.
  * ex. `Lazy.of(HttpClient)`
* `All` - Injects an array of all services registered with the provided key.
  * ex. `All.of(Plugin)`
* `Optional` - Injects an instance of a class only if it already exists in the container; null otherwise.
  * ex. `Optional.of(LoggedInUser)`
* `Parent` - Skips starting dependency resolution from the current container and instead begins the lookup process on the parent container.
  * ex. `Parent.of(MyCustomElement)`
* `Factory` - Used to allow injecting dependencies, but also passing data to the constructor.
  * ex. `Factory.of(CustomClass)`
* `NewInstance` - Used to inject a new instance of a dependency, without regard for existing instances in the container.
  * ex. `NewInstance.of(CustomClass).as(Another)`

#### Explicit Registration

```javascript
  import {transient, inject} from 'aurelia-framework';
  import {HttpClient} from 'aurelia-fetch-client';
  
  @transient()
  @inject(HttpClient)
  export class CustomerDetail {
    constructor(http) {
      this.http = http;
    }
  }
```

### Templating Basics <a href="#templating-basics" id="templating-basics"></a>

#### A Simple Template

```markup
  <template>
    <div>Hello World!</div>
  </template>
```

#### Requiring Resources

```markup
  <template>
    <require from='nav-bar'></require>
  
    <nav-bar router.bind="router"></nav-bar>
  
    <div class="page-host">
      <router-view></router-view>
    </div>
  </template>
```

{% hint style="info" %}
**Invalid Table Structure When Dynamically Creating Tables**

When the browser loads in the template it very helpfully validates the structure of the HTML, notices that you have an invalid tag inside your table definition, and very unhelpfully removes it for you before Aurelia even has a chance to examine your template.
{% endhint %}

{% hint style="warning" %}
**For webpack users**\
Dynamic compose as used below does not work when using webpack. It is suggested to bind to a property on the viewmodel `template: PLATFORM.moduleName("./template_.html")` and then use it like so `<tr repeat.for="r of ['A','B','A','B']" as-element="compose" view.bind='template'>`
{% endhint %}

Use of the `as-element` attribute ensures we have a valid HTML table structure at load time, yet we tell Aurelia to treat its contents as a different tag.

**Compose an existing object instance with a view.**

```markup
  <template>
    <table>
      <tr repeat.for="r of ['A','B','A','B']" as-element="compose" view='./template_${r}.html'>
    </table>
  <template>
```

For the above example we can then programmatically choose the embedded template based on an element of our data:

#### template\_A.html

```markup
  <template>
    <td>I'm an A Row</td><td>Col 2A</td><td>Col 3A</td>
  </template>
```

#### template\_B.html

```markup
  <template>
    <td>I'm an B Row</td><td>Col 2B</td><td>Col 3B</td>
  </template>
```

Note that when a `containerless` attribute is used, the container is stripped _after_ the browser has loaded the DOM elements, and as such this method cannot be used to transform non-HTML compliant structures into compliant ones!

**Illegal Table Code**

```markup
  <template>
    <table>
      <template repeat.for="customer of customers">
        <tr>
          <td>${customer.fullName}</td>
        </tr>
      </template>
    </table>
  </template>
```

#### Correct Table Code

```markup
  <template>
    <table>
      <tr repeat.for="customer of customers">
        <td>${customer.fullName}</td>
      </tr>
    </table>
  </template>
```

#### Illegal Select Code

```markup
  <template>
    <select>
      <template repeat.for="customer of customers">
        <option>...</option>
      </template>
    </select>
  </template>
```

#### Correct Select Code

```markup
  <template>
    <select>
      <option repeat.for="customer of customers">...</option>
    </select>
  </template>
```

### Databinding <a href="#databinding" id="databinding"></a>

#### bind, one-way, two-way & one-time

Use on any HTML attribute.

* `.bind` - Uses the default binding. One-way binding for everything but form controls, which use two-way binding.
* `.one-way` - Flows data one direction: from the view-model to the view.
* `.two-way` - Flows data both ways: from view-model to view and from view to view-model.
* `.one-time` - Renders data once, but does not synchronize changes after the initial render.

#### Data Binding Examples

```markup
  <template>
    <input type="text" value.bind="firstName">
    <input type="text" value.two-way="lastName">
  
    <a href.one-way="profileUrl">View Profile</a>
  </template>
```

{% hint style="info" %}
At the moment inheritance of bindables is not supported. For use cases where `class B extends A` and `B` is used as custom Element/Attribute `@bindable` properties cannot be defined only on `class A`. If inheritance is used, `@bindable` properties should be defined on the instantiated class.
{% endhint %}

#### delegate, trigger

Use on any native or custom DOM event. (Do not include the "on" prefix in the event name.)

* `.trigger` - Attaches an event handler directly to the element. When the event fires, the expression will be invoked.
* `.delegate` - Attaches a single event handler to the document (or nearest shadow DOM boundary) which handles all events of the specified type, properly dispatching them back to their original targets for invocation of the associated expression.

{% hint style="info" %}
The `$event` value can be passed as an argument to a `delegate` or `trigger` function call if you need to access the event object.
{% endhint %}

#### Event Binding Examples

```markup
  <template>
    <button click.trigger="save()">Save</button>
    <button click.delegate="save($event)">Save</button>
  </template>
```

#### call

Passes a function reference.

```markup
  <template>
    <button my-attribute.call="sayHello()">Say Hello</button>
  </template>
```

#### ref

Creates a reference to an HTML element, a component or a component's parts.

* `ref="someIdentifier"` or `element.ref="someIdentifier"` - Create a reference to the HTMLElement in the DOM.
* `attribute-name.ref="someIdentifier"`- Create a reference to a custom attribute's view-model.
* `view-model.ref="someIdentifier"`- Create a reference to a custom element's view-model.
* `view.ref="someIdentifier"`- Create a reference to a custom element's view instance (not an HTML Element).
* `controller.ref="someIdentifier"`- Create a reference to a custom element's controller instance.

```markup
  <template>
    <input type="text" ref="name"> ${name.value}
  </template>
```

#### String Interpolation

Used in an element's content. Can be used inside attributes, particularly useful in the `class` and `css` attributes.

```markup
  <template>
    <span>${fullName}</span>
    <div class="dot ${color} ${isHappy ? 'green' : 'red'}"></div>
  </template>
```

#### Binding to Select Elements

A typical select element is rendered using a combination of `value.bind` and `repeat`. You can also bind to arrays of objects and synchronize based on an id (or similar) property.

#### Basic Select

```markup
  <template>
    <select value.bind="favoriteColor">
      <option>Select A Color</option>
      <option repeat.for="color of colors" value.bind="color">${color}</option>
    </select>
  </template>
```

#### Select With Object Array

```markup
  <template>
    <select value.bind="employeeOfTheMonth">
      <option>Select An Employee</option>
      <option repeat.for="employee of employees" model.bind="employee">${employee.fullName}</option>
    </select>
  </template>
```

**Select with Object Id Sync**

```markup
  <template>
    <select value.bind="employeeOfTheMonthId">
      <option>Select An Employee</option>
      <option repeat.for="employee of employees" model.bind="employee.id">${employee.fullName}</option>
    </select>
  </template>
```

**Basic Multi-Select**

```markup
  <template>
    <select value.bind="favoriteColors" multiple>
      <option repeat.for="color of colors" value.bind="color">${color}</option>
    </select>
  </template>
```

**Multi-Select with Object Array**

```markup
  <template>
    <select value.bind="favoriteEmployees" multiple>
      <option repeat.for="employee of employees" model.bind="employee">${employee.fullName}</option>
    </select>
  </template>
```

#### Binding Radios

```html
  <template>
    <label repeat.for="color of colors">
      <input type="radio" name="clrs" value.bind="color" checked.bind="$parent.favoriteColor">
      ${color}
    </label>
  </template>
```

```markup
  <template>
    <label repeat.for="employee of employees">
      <input type="radio" name="emps" model.bind="employee" checked.bind="$parent.employeeOfTheMonth">
      ${employee.fullName}
    </label>
  </template>
```

```markup
  <template>
    <label><input type="radio" name="tacos" model.bind="null" checked.bind="likesTacos">Unanswered</label>
    <label><input type="radio" name="tacos" model.bind="true" checked.bind="likesTacos">Yes</label>
    <label><input type="radio" name="tacos" model.bind="false" checked.bind="likesTacos">No</label>
  </template>
```

#### Binding Checkboxes

{% hint style="warning" %}
You cannot use a `click.delegate` on checkboxes if you want to attach a method to it. You need to use `change.delegate`.
{% endhint %}

```markup
  <template>
    <label repeat.for="color of colors">
      <input type="checkbox" value.bind="color" checked.bind="$parent.favoriteColors" >
      ${color}
    </label>
  </template>
```

```markup
  <template>
    <label repeat.for="employee of employees">
      <input type="checkbox" model.bind="employee" checked.bind="$parent.favoriteEmployees">
      ${employee.fullName}
    </label>
  </template>
```

```markup
  <template>
    <li><label><input type="checkbox" checked.bind="wantsFudge">Fudge</label></li>
    <li><label><input type="checkbox" checked.bind="wantsSprinkles">Sprinkles</label></li>
    <li><label><input type="checkbox" checked.bind="wantsCherry">Cherry</label></li>
  </template>
```

#### Binding innerHTML and textContent

```markup
  <template>
    <div innerhtml.bind="htmlProperty | sanitizeHTML"></div>
    <div innerhtml="${htmlProperty | sanitizeHTML}"></div>
  </template>
```

{% hint style="danger" %}
Always use HTML sanitization. We provide a simple converter that can be used.
{% endhint %}

{% hint style="warning" %}
Binding using the `innerhtml` attribute simply sets the element's `innerHTML` property. The markup does not pass through Aurelia's templating system. Binding expressions and require elements will not be evaluated.
{% endhint %}

```markup
  <template>
    <div textcontent.bind="stringProperty"></div>
    <div textcontent="${stringProperty}"></div>
  </template>
```

```markup
  <template>
    <div textcontent.bind="stringProperty" contenteditable="true"></div>
  </template>
```

#### Binding Style

You can bind a css string or object to an element's `style` attribute. Use the `style` attribute's alias, `css` when doing string interpolation to ensure your application is compatible with Internet Explorer.

```javascript
  export class StyleData {
    constructor() {
      this.styleString = 'color: red; background-color: blue';
  
      this.styleObject = {
        color: 'red',
        'background-color': 'blue'
      };
    }
  }
```

```markup
  <template>
    <div style.bind="styleString"></div>
    <div style.bind="styleObject"></div>
  </template>
```

#### Illegal Style Interpolation

```markup
  <template>
    <div style="width: ${width}px; height: ${height}px;"></div>
  </template>
```

#### Legal Style Interpolation

```markup
  <template>
    <div css="width: ${width}px; height: ${height}px;"></div>
  </template>
```

#### Declaring Computed Property Dependencies

```javascript
  import {computedFrom} from 'aurelia-framework';
  
  export class Person {
    firstName = 'John';
    lastName = 'Doe';
  
    @computedFrom('firstName', 'lastName')
    get fullName(){
      return `${this.firstName} ${this.lastName}`;
    }
  }
```

### Templating View Resources <a href="#templating-view-resources" id="templating-view-resources"></a>

**Conditionally displays an HTML element.**

```markup
  <template>
    <div show.bind="isSaving" class="spinner"></div>
  </template>
```

**Conditionally add/remove an HTML element**

```markup
  <template>
    <div if.bind="isSaving" class="spinner"></div>
  </template>
```

**Conditionally add/remove a group of elements**

```markup
  <template>
    <input value.bind="firstName">
  
    <template if.bind="hasErrors">
        <i class="icon error"></i>
        ${errorMessage}
    </template>
  </template>
```

**Creating new component instance every time condition changes**

```markup
  <template>
    <my-custom-element if="condition.bind: showMessage; cache: false"></my-custom-element>
  </template>
```

**Render an array with a template**

```markup
  <template>
    <ul>
      <li repeat.for="customer of customers">${customer.fullName}</li>
    </ul>
  </template>
```

**Render a map with a template**

```markup
  <template>
    <ul>
      <li repeat.for="[id, customer] of customers">${id} ${customer.fullName}</li>
    </ul>
  </template>
```

**Render a template N times**

```markup
  <template>
    <ul>
      <li repeat.for="i of rating">*</li>
    </ul>
  </template>
```

Contextual items available inside a repeat template:

* `$index` - The index of the item in the array.
* `$first` - True if the item is the first item in the array.
* `$last` - True if the item is the last item in the array.
* `$even` - True if the item has an even numbered index.
* `$odd` - True if the item has an odd numbered index.

{% hint style="info" %}
**Containerless Template Controllers**

The `if` and `repeat` attributes are usually placed on the HTML elements that they affect. However, you can also use a `template` tag to group a collection of elements that don't have a parent element and place the `if` or `repeat` on the `template` element instead.
{% endhint %}

**Dynamically render UI into the DOM based on data**

```markup
  <template repeat.for="item of items">
    <compose model.bind="item" view-model="widgets/${item.type}"></compose>
  </template>
```

**Composing a view only, inheriting the parent binding context**

```markup
  <template repeat.for="item of items">
    <compose view="my-view.html"></compose>
  </template>
```

**Compose an existing object instance with a view**

```markup
  <template>
    <div repeat.for="item of items">
      <compose view="my-view.html" view-model.bind="item">
    </div>
  </template>
```

### Routing <a href="#routing" id="routing"></a>

#### Basic Route Configuration

```javascript
  export class App {
    configureRouter(config, router) {
      this.router = router;
      config.title = 'Aurelia';
      config.map([
        { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
        { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
        { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
        { route: 'files*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
      ]);
    }
  }
```

#### Route Pattern Options

* static routes
  * ie 'home' - Matches the string exactly.
* parameterized routes
  * ie 'users/:id/detail' - Matches the string and then parses an `id` parameter. Your view-model's `activate` callback will be called with an object that has an `id` property set to the value that was extracted from the url.
* wildcard routes
  * ie 'files\*path' - Matches the string and then anything that follows it. Your view-model's `activate` callback will be called with an object that has a `path` property set to the wildcard's value.

#### The Route Screen Activation Lifecycle

* `canActivate(params, routeConfig, navigationInstruction)` - Implement this hook if you want to control whether or not your view-model _can be navigated to_. Return a boolean value, a promise for a boolean value, or a navigation command.
* `activate(params, routeConfig, navigationInstruction)` - Implement this hook if you want to perform custom logic just before your view-model is displayed. You can optionally return a promise to tell the router to wait to bind and attach the view until after you finish your work.
* `canDeactivate()` - Implement this hook if you want to control whether or not the router _can navigate away_ from your view-model when moving to a new route. Return a boolean value, a promise for a boolean value, or a navigation command.
* `deactivate()` - Implement this hook if you want to perform custom logic when your view-model is being navigated away from. You can optionally return a promise to tell the router to wait until after you finish your work.

{% hint style="warning" %}
**Root Screen Activation**

Unlike the mapped routes, the root's view-model only has access to the `activate()` hook. However this can also be used to implement logic for _attaching the component_ by returning a promise for a boolean value.
{% endhint %}

The `params` object will have a property for each parameter of the route that was parsed, as well as a property for each query string value. `routeConfig` will be the original route configuration object that you set up. `routeConfig` will also have a new `navModel` property, which can be used to change the document title for data loaded by your view-model. For example:

**Route Params and NavModel**

```javascript
  import {autoinject} from 'aurelia-framework';
  import {UserService} from './user-service';
  
  @inject(UserService)
  export class UserEditScreen {
    constructor(userService) {
      this.userService = userService;
    }
  
    activate(params, routeConfig) {
      return this.userService.getUser(params.id)
        .then(user => {
          routeConfig.navModel.setTitle(user.name);
        });
    }
  }
```

#### Conventional Routing

```javascript
  export class App {
    configureRouter(config){
      config.mapUnknownRoutes(instruction => {
        //check instruction.fragment
        //return moduleId
      });
    }
  }
```

#### Customizing The Navigation Pipeline

```javascript
 import {Redirect} from 'aurelia-router';
  
  export class App {
    configureRouter(config) {
      config.title = 'Aurelia';
      config.addPipelineStep('authorize', AuthorizeStep);
      config.map([
        { route: ['welcome'],    name: 'welcome',       moduleId: 'welcome',      nav: true, title:'Welcome' },
        { route: 'flickr',       name: 'flickr',        moduleId: 'flickr',       nav: true, auth: true },
        { route: 'child-router', name: 'childRouter',   moduleId: 'child-router', nav: true, title:'Child Router' },
        { route: '', redirect: 'welcome' }
      ]);
    }
  }
  
  class AuthorizeStep {
    run(navigationInstruction, next) {
      if (navigationInstruction.getAllInstructions().some(i => i.config.auth)) {
        var isLoggedIn = /* insert magic here */false;
        if (!isLoggedIn) {
          return next.cancel(new Redirect('login'));
        }
      }
  
      return next();
    }
  }
```

{% hint style="warning" %}
PushState requires server-side support. Don't forget to configure your server appropriately.
{% endhint %}

#### Reusing an Existing View Model

Since the view model's navigation lifecycle is called only once, you may have problems recognizing that the user switched the route from `Product A` to `Product B` (see below). To work around this issue implement the method `determineActivationStrategy` in your view model and return hints for the router about what you'd like to happen. Available return values are `replace` and `invoke-lifecycle`. Remember, "lifecycle" refers to the navigation lifecycle.

**Router View Model Activation Control**

```javascript
  //app.js
  
  export class App {
    configureRouter(config) {
      config.title = 'Aurelia';
      config.map([
        { route: 'product/a',    moduleId: 'product',     nav: true },
        { route: 'product/b',    moduleId: 'product',     nav: true },
      ]);
    }
  }
  
  //product.js
  
  import {activationStrategy} from 'aurelia-router';
  
  export class Product {
    determineActivationStrategy() {
      return activationStrategy.replace;
    }
  }
```

{% hint style="info" %}
Alternatively, if the strategy is always the same and you don't want that to be in your view model code, you can add the `activationStrategy` property to your route config instead.
{% endhint %}

#### Rendering multiple ViewPorts

{% hint style="info" %}
If you don't name a `router-view`, it will be available under the name `'default'`.
{% endhint %}

**Multi-ViewPort View**

```markup
  <template>
    <div class="page-host">
      <router-view name="left"></router-view>
    </div>
    <div class="page-host">
      <router-view name="right"></router-view>
    </div>
  </template>
```

#### Multi-ViewPort View-Model

```javascript
  export class App {
    configureRouter(config){
      config.map([{
        route: 'edit',
          viewPorts: {
            left: {
              moduleId: 'editor'
            },
            right: {
              moduleId: 'preview'
            }
          }
        }]);
    }
  }
```

#### Generating Route URLs

```javascript
router.generate('routeName', { id: 123 });

router.navigateToRoute('routeName', { id: 123 })
```

```markup
  <template>
    <a route-href="route: routeName; params.bind: { id: user.id }">${user.name}</a>
  </template>
```

```markup
  <template>
    <a href="/my-page" router-ignore>Click to load my-page from server</a>
  </template>
```

### Custom Attributes <a href="#custom-attributes" id="custom-attributes"></a>

**Simple Attribute Declaration**

```javascript
  import {inject, customAttribute, DOM} from 'aurelia-framework';
  
  @customAttribute('my-attribute')
  @inject(DOM.Element)
  export class Show {
    constructor(element) {
      this.element = element;
    }
  
    valueChanged(newValue, oldValue) {
      ...
    }
  }
```

**Simple Attribute Use**

```markup
  <template>
    <div my-attribute="literal value"></div>
    <div my-attribute.bind="an.expression"></div>
  </template>
```

**Options Attribute Declaration**

```javascript
  import {customAttribute, bindable} from 'aurelia-framework';
  
  @customAttribute('my-attribute')
  export class MyAttribute {
    @bindable foo;
    @bindable bar;
  
    fooChanged(newValue, oldValue) { ... }
  
    barChanged(newValue, oldValue) { ... }
  }
  
```

**Options Attribute Use**

```markup
  <template>
    <div my-attribute="foo: literal value; bar.bind: an.expression"></div>
  </template>
```

**Dynamic Option Attribute Declaration**

```javascript
  import {customAttribute, dynamicOptions} from 'aurelia-framework';
  
  @customAttribute('my-attribute')
  @dynamicOptions
  export class MyAttribute {
    propertyChanged(propertyName, newValue, oldValue) { ... }
  }
```

**Dynamic Option Attribute Use**

```markup
  <template>
    <div my-attribute="foo: literal value; bar.bind: an.expression"></div>
  </template>
```

**Bindable Signature (Showing Defaults)**

```javascript
  bindable({
    name:'myProperty',
    attribute:'my-property',
    changeHandler:'myPropertyChanged',
    defaultBindingMode: bindingMode.oneWay,
    defaultValue: undefined
  })
```

**Template Controller Attribute Declaration**

```javascript
  import {BoundViewFactory, ViewSlot, customAttribute, templateController, inject} from 'aurelia-framework';
  
  @customAttribute('naive-if')
  @templateController
  @inject(BoundViewFactory, ViewSlot)
  export class NaiveIf {
    constructor(viewFactory, viewSlot) {
      this.viewFactory = viewFactory;
      this.viewSlot = viewSlot;
    }
  
    valueChanged(newValue) {
      if (newValue) {
        let view = this.viewFactory.create();
        this.viewSlot.add(view);
      } else {
        this.viewSlot.removeAll();
      }
    }
  }
```

**Template Controller Attribute Use**

```markup
  <template>
    <div naive-if.bind="an.expression"></div>
  
    <template naive-if.bind="an.expression">
      <div></div>
      <div></div>
    </template>
  </template>
```

### Custom Elements <a href="#custom-elements" id="custom-elements"></a>

**Custom Element View-Model Declaration**

```javascript
  import {customElement, bindable} from 'aurelia-framework';
  
  @customElement('say-hello')
  export class SayHello {
    @bindable to;
  
    speak(){
      alert(`Hello ${this.to}!`);
    }
  }
```

**Custom Element View Declaration**

```markup
  <template>
      <button click.trigger="speak()">Say Hello To ${to}</button>
  </template>
```

**Custom Element Use**

```markup
  <template>
    <require from="say-hello"></require>
  
    <input type="text" ref="name">
    <say-hello to.bind="name.value"></say-hello>
  </template>
```

#### Custom Element Without View-Model Declaration

Aurelia will not search for a JavaScript file if you reference a component with an .html extension.

```markup
  <template bindable="greeting,name">
      Say ${greeting} To ${name}
  </template>
```

```javascript
aurelia.use.globalResources('./js-less-component.html');
```

```markup
  <require from="./js-less-component.html"></require>
  
  <js-less-component greeting="Hello" name.bind="someProperty"></js-less-component>
```

#### Custom Element Variable Binding

It's worth noting that when binding variables to custom elements, **use camelCase inside the custom element's View-Model, and dash-case on the html element**. See the following example:

```javascript
  import {bindable} from 'aurelia-framework';
  
  export class SayHello {
    @bindable to;
    @bindable greetingCallback;
  
    speak(){
      this.greetingCallback(`Hello ${this.to}!`);
    }
  }
```

```markup
  <template>
    <require from="./say-hello"></require>
  
    <input type="text" ref="name">
    <say-hello to.bind="name.value" greeting-callback.call="doSomething($event)"></say-hello>
  </template>
```

#### Custom Element Options

* `@children(selector)` - Decorates a property to create an array on your class that has its items automatically synchronized based on a query selector against the element's immediate child content. Does not work with `@containerless()`, see below.
* `@child(selector)` - Decorates a property to create a reference to a single immediate child content element. Does not work with `@containerless()`, see below.
* `@processContent(false|Function)` - Tells the compiler that the element's content requires special processing. If you provide `false` to the decorator, the compiler will not process the content of your custom element. It is expected that you will do custom processing yourself. But, you can also supply a custom function that lets you process the content during the view's compilation. That function can then return true/false to indicate whether or not the compiler should also process the content. The function takes the following form `function(compiler, resources, node, instruction):boolean`
* `@useView(path)` - Specifies a different view to use.
* `@noView()` - Indicates that this custom element does not have a view and that the author intends for the element to handle its own rendering internally.
* `@inlineView(markup, dependencies?)` - Allows the developer to provide a string that will be compiled into the view.
* `@useShadowDOM(options?: { mode: 'open' | 'closed' })` - Causes the view to be rendered in the ShadowDOM. When an element is rendered to ShadowDOM, a special `DOMBoundary` instance can optionally be injected into the constructor. This represents the shadow root. Does not work with `@containerless()`, see below.
* `@containerless()` - Causes the element's view to be rendered without the custom element container wrapping it. This cannot be used in conjunction with `@child`, `@children` or `@useShadowDOM` decorators. It also cannot be uses with surrogate behaviors. Use sparingly.

#### SVG Elements

SVG (scalable vector graphic) tags can support Aurelia's custom element `<template>` tags by nesting the templated code inside a second `<svg>` tag. For example if you had a base `<svg>` element and wanted to add a templated `<rect>` inside it, you would first put your custom tag inside the main `<svg>` tag. Also, make sure the custom element class uses the `@containerless()` decorator.

```javascript
  import {containerless} from 'aurelia-framework';
  
  @containerless()
  export class MyCustomRect {
    ...
  }
```

```markup
  <template>
    <svg>
      <rect width="10" height="10" fill="red" x="50" y="50"/>
    </svg>
  </template>
```

```markup
  <template>
    <require from="my-custom-rect"></require>
  
    <svg width="100" height="100" >
      <my-custom-rect></my-custom-rect>
    </svg>
  </template>
```

#### Template Parts

```markup
  <template>
    <ul>
      <li class="foo" repeat.for="item of items">
        <template replaceable part="item-template">
          Original: ${item}
        </template>
      </li>
    <ul>
  </template>
```

```markup
  <template>
    <require from="my-element"></require>
  
    <my-element>
      <template replace-part="item-template">
        Replacement: ${item}
      </template>
    </my-element>
  </template>
```

```markup
  <template role="progress-bar" aria-valuenow.bind="progress" aria-valuemin="0" aria-valuemax="100">
    <div class="bar">
      <div class="progress" css="width:${progress}%"></div>
    </div>
  </template>
```

#### Observable decorator

Aurelia exposes a decorator named observable to allow watching for changes to a property and reacting to them. By convention it will look for a matching method name `Changed`

```javascript
  import {observable} from 'aurelia-framework';
  
  export class MyCustomViewModel {
    @observable name = 'Hello world';
    nameChanged(newValue, oldValue) {
      // react to change
    }
  }
```

The developer can also specify a different method name to use.

```javascript
  import {observable} from 'aurelia-framework';
  
  export class MyCustomViewModel {
    @observable({changeHandler: 'nameChanged'}) name = 'Hello world';
    nameChanged(newValue, oldValue) {
      // react to change
    }
  }
```

### The Event Aggregator <a href="#the-event-aggregator" id="the-event-aggregator"></a>

If you include the `aurelia-event-aggregator` plugin using "basicConfiguration" or "standardConfiguration" then the singleton EventAggregator's API will be also present on the `Aurelia` object. You can also create additional instances of the EventAggregator, if needed, and "merge" them into any object. To do this, import `includeEventsIn` and invoke it with the object you wish to turn into an event aggregator. For example `includeEventsIn(myObject)`. Now my object has `publish` and `subscribe` methods and can be used in the same way as the global event aggregator, detailed below.

#### Publishing on a Channel

```javascript
  import {inject} from 'aurelia-framework';
  import {EventAggregator} from 'aurelia-event-aggregator';
  
  @inject(EventAggregator)
  export class APublisher {
    constructor(eventAggregator) {
      this.eventAggregator = eventAggregator;
    }
  
    publish(){
      var payload = {};
      this.eventAggregator.publish('channel name here', payload);
    }
  }
```

#### Subscribing to a Channel

```javascript
  import {inject} from 'aurelia-framework';
  import {EventAggregator} from 'aurelia-event-aggregator';
  
  @inject(EventAggregator)
  export class ASubscriber {
    constructor(eventAggregator) {
      this.eventAggregator = eventAggregator;
    }
  
    subscribe() {
      this.eventAggregator.subscribe('channel name here', payload => {
          ...
      });
    }
  }
```

#### Publishing a Message

```javascript
  //some-message.js
  export class SomeMessage{ }
  
  //a-publisher.js
  import {inject} from 'aurelia-framework';
  import {EventAggregator} from 'aurelia-event-aggregator';
  import {SomeMessage} from './some-message';
  
  @inject(EventAggregator)
  export class APublisher {
    constructor(eventAggregator) {
      this.eventAggregator = eventAggregator;
    }
  
    publish() {
      this.eventAggregator.publish(new SomeMessage());
    }
  }
```

#### Subscribing to a Message Type

```javascript
  import {inject} from 'aurelia-framework';
  import {EventAggregator} from 'aurelia-event-aggregator';
  import {SomeMessage} from './some-message';
  
  @inject(EventAggregator)
  export class ASubscriber {
    constructor(eventAggregator) {
      this.eventAggregator = eventAggregator;
    }
  
    subscribe(){
      this.eventAggregator.subscribe(SomeMessage, message => {
          ...
      });
    }
  }
  
```
