---
description: >-
  Now that you've got the basics down, you need to learn how to use the CLI,
  build a more complex app and get a solid knowledge foundation for real-world
  work.
---

# Contact Manager tutorial

Ready to dive deeper? Let's build a contact manager app to showcase Aurelia's power and versatility. This tutorial will take you beyond the basics, introducing key features and practical techniques you'll use in real-world projects. We'll also get you comfortable with the CLI, setting you up for more complex development work.

## Setting up your machine

Before diving into Aurelia, you'll need to set up your development environment. This process is straightforward:

1. Install Node.js if you haven't already.
2. Use npm (or your preferred package manager) to install the Aurelia CLI.

If you've already installed the Aurelia CLI, feel free to skip ahead to the next section.

For detailed installation instructions, check out our [Quick Start](quick-start.md) guide.

## Creating a new Aurelia project

Let's create our contact manager app. From the command line, run:

```
au new
```

You'll see several options. Name your project "contact-manager" and choose either "Default ESNext" or "Default TypeScript" based on your preference. (Skip the "Custom" option for this tutorial.)

When prompted to install dependencies, hit enter to accept the default "yes".

Your project is ready after installation (which may take a few minutes). Navigate to the project folder in your terminal and launch it with:

```
au run --open
```

This command runs the app, opens a browser tab, and watches for source changes. You'll see "Hello World!" in your browser if everything's set up correctly.

### Adding required assets

In this tutorial, we'll use a mock in-memory backend. To save time, we've prepared the CSS and some utility functions in advance. Before we start coding, download these essential assets and add them to your project.

[Download the Contact Manager Assets](https://aurelia.io/downloads/contact-manager-assets.zip)

Once you've downloaded the zip file, extract it, and you'll find three files:

* `web-api.js` - The fake, in-memory backend.
* `utility.js` - Some helper functions used by the app.
* `styles.css` - The styles for this app.

Copy all of these files to your project `src` folder. TypeScript users should also rename the file extensions from `.js` to `.ts`.

## Building the application shell

{% hint style="warning" %}
Before proceeding, please ensure you are familiar with the concepts introduced in [Aurelia for new developers](aurelia-for-new-developers.md) or have some basic experience with Aurelia.
{% endhint %}

Let's start by looking at a picture of the final product of this tutorial. It will help us to see the application's structure and the pieces we need to build.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Our app layout consists of a header, a contact list on the left, and a detail pane. This structure forms our app's shell. Let's set it up.

First, we'll configure the `App` class with a router. This allows browser history to reflect the selected contact, enabling smooth navigation. Update your `app.ts/js` with this code:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import {PLATFORM} from 'aurelia-pal';
  
export class App {
  configureRouter(config, router){
    config.title = 'Contacts';
    config.options.pushState = true;
    config.options.root = '/';
    config.map([
      { route: '',              moduleId: PLATFORM.moduleName('no-selection'),   title: 'Select' },
      { route: 'contacts/:id',  moduleId: PLATFORM.moduleName('contact-detail'), name:'contacts' }
    ]);

    this.router = router;
  }
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import {Router, RouterConfiguration} from 'aurelia-router';
import {PLATFORM} from 'aurelia-pal';

export class App {
  router: Router;

  configureRouter(config: RouterConfiguration, router: Router){
    config.title = 'Contacts';
    config.options.pushState = true;
    config.options.root = '/';
    config.map([
      { route: '',              moduleId: PLATFORM.moduleName('no-selection'),   title: 'Select' },
      { route: 'contacts/:id',  moduleId: PLATFORM.moduleName('contact-detail'), name:'contacts' }
    ]);

    this.router = router;
  }
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
After changing the code above, if you immediately try to compile, you may receive a compile error on your unit tests because the sample test references the `App` class, which we just changed. To address this, remove the dummy unit test.
{% endhint %}

Adding routing to your Aurelia app is straightforward. Include a `configureRouter` method in your `App` class. This method receives two parameters: `RouterConfiguration` and `Router`.

Use the `map` method on the configuration object to set up your routes. Each route requires at least two properties:

1. `route`: The URL pattern to match
2. `moduleId`: The module to load when the route is matched

In this setup, we've defined two routes:

1. An empty route (`''`) that loads the `no-selection` module when no specific route is matched.
2. A `contacts/:id` route that loads the `contact-detail` module when a specific contact is selected. The `:id` part is a parameter that can be used to identify the selected contact.

This configuration allows for a default view and a detailed contact view in your application.

{% hint style="info" %}
Did you notice the calls to `PLATFORM.moduleName(....)`? This special API is used in Aurelia Webpack projects to allow Webpack to identify strings that represent modules. This enables Webpack to include the referenced module in the built package.
{% endhint %}

Two key points to note in this configuration:

1. The `config.title` property sets a base title for the browser's document. You can also set titles on individual routes. When you do, the router's title and the matched route's title combine to form the final document title.
2. The lines `config.options.pushState = true;` and `config.options.root = '/';` are optional. They enable pushState-based routing. Without these, Aurelia defaults to hash-based routing, which works even on older browsers like IE9.

Routing examples:

* PushState: `http://localhost:8080/contacts/1`
* Hash-based: `http://localhost:8080/#/contacts/1`

The second route introduces a name property. This allows us to reference the route by name later on, eliminating the need to repeat the route pattern throughout our code.

With our navigation structure in place, let's set up the visual layout. Update your `app.html` file with this markup:

```html
<template>
  <require from="./styles.css"></require>

  <nav class="navbar navbar-light bg-light border-bottom fixed-top" role="navigation">
    <a class="navbar-brand" href="#">
      <i class="fa fa-user"></i>
      <span>Contacts</span>
    </a>
  </nav>

  <div class="container-md">
    <div class="row">
      <div class="col-sm-5 col-md-4">Contact List Placeholder</div>
      <router-view class="col-sm-7 col-md-8"></router-view>
    </div>
  </div>
</template>
```

The structure of this view reveals some key Aurelia concepts. At the top, you'll notice `require` elements. These function similarly to ES2015's `import` syntax, allowing you to bring in resources like custom styles into your view.

The main body of the view consists of a navbar and a two-column layout. The first column contains a placeholder for our contact list, while the second houses the `router-view` custom element. This Aurelia-provided element indicates where the current route should be rendered, offering flexibility in your application's layout.

Note that when you have a `configureRouter` method in your view-model, your view must include a `router-view`.

To complete our application shell setup, we need to install and import Bootstrap. While we're using Bootstrap in this tutorial for styling, feel free to use any CSS framework you prefer in your own projects.

To install Bootstrap, run the following command:

```sh
npm install bootstrap
npm install font-awesome
```

Next, we need to import it into our `main.ts/js` by adding the following line to the top of the file:

```javascript
import 'bootstrap/dist/css/bootstrap.css';
import 'font-awesome/css/font-awesome.css';
```

{% hint style="info" %}
Whenever you install new dependencies used in your app, make sure to restart the `au run` command, in order to have the CLI re-bundle your freshly added dependencies.
{% endhint %}

## Creating the default route

If you run the application now, using `au run`, you'll see a compile error similar to:

```sh
 ERROR in ./src/app.js
  Module not found: Error: Can't resolve 'contact-detail' in '/contact-manager/src'
```

This behavior is normal. We've used `PLATFORM.moduleName(....)` in our route configuration, but we haven't created the actual modules yet. Let's fix that now. Create a new file called no-selection.js in the src folder and add this code:

{% tabs %}
{% tab title="Javascript" %}
{% code title="no-selection.js" %}
```javascript
export class NoSelection {
  constructor() {
    this.message = "Please Select a Contact.";
  }
}
```
{% endcode %}
{% endtab %}

{% tab title="TypeScript" %}
```typoscript
export class NoSelection {
    message = "Please Select a Contact.";
}
```
{% endtab %}
{% endtabs %}

Now, let's create a simple "no selection" screen. This will prompt users to choose a contact when none is selected.

Create a new file named `no-selection.html` in your `src` folder with this content:

```html
<template>
  <div class="no-selection text-center">
    <h4>${message}</h4>
  </div>
</template>
```

This container applies basic styling to display our user message.

Now, let's create a placeholder for our contact detail screen. This will keep our build happy. In the src folder, add a new file called `contact-detail.ts/js` with this code:

```javascript
export class ContactDetail {

}
```

With this in place, you can now run your application. If you haven't stopped/restarted it after editing the bundles, then you must do that now. When you run the application, you should see something similar to this:

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## Creating the contact list

Now that we've set up our application's basic structure and routing, it's time to make things more interesting. Let's replace that placeholder div with a proper contact list by creating a custom element.

In Aurelia, building custom elements follows the same pattern as creating your `App` component or routed components. This consistency is a hallmark of the framework. To get started, create a new file called `contact-list.ts/js` and add the following code:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import {WebAPI} from './web-api';
import {inject} from 'aurelia-framework';

@inject(WebAPI)
export class ContactList {
  constructor(api) {
    this.api = api;
    this.contacts = [];
  }

  created() {
    this.api.getContactList().then(contacts => this.contacts = contacts);
  }

  select(contact) {
    this.selectedId = contact.id;
    return true;
  }
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import {WebAPI} from './web-api';
import {inject} from 'aurelia-framework';

@inject(WebAPI)
export class ContactList {
  contacts;
  selectedId = 0;

  constructor(private api: WebAPI) { }

  created() {
    this.api.getContactList().then(contacts => this.contacts = contacts);
  }

  select(contact) {
    this.selectedId = contact.id;
    return true;
  }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
We use a dashed naming convention to separate the words _contact-list_ as our custom element name. The name of the class instead should be defined by using the UpperCamelCase version _ContactList_.
{% endhint %}

The `ContactList` view-model showcases a few key Aurelia features. It uses dependency injection, a core Aurelia concept, to manage class dependencies. Our `ContactList` class depends on the WebAPI class, which Aurelia will automatically provide when instantiating ContactList.

The `created` method demonstrates Aurelia's component lifecycle. By implementing this method, we hook into the stage where both view-model and view are ready. Here, we fetch the contact list from our API and store it for binding in the view.

Lastly, we have a `select` method for contact selection, which we'll explore further when we look at the view. Speaking of which, let's create a `contact-list.html` file for our view:

```html
  <template>
    <div class="contact-list">
      <div class="list-group">
        <a
          repeat.for="contact of contacts"
          class="list-group-item list-group-item-action ${contact.id === $parent.selectedId ? 'active' : ''}"
          route-href="route: contacts; params.bind: {id:contact.id}"
          click.delegate="$parent.select(contact)"
        >
          <strong>${contact.firstName} ${contact.lastName}</strong><br>
          <small>${contact.email}</small>
        </a>
      </div>
    </div>
  </template>
```

The markup repeats an `<a>` element for each contact in our contacts array.

Notice the class attribute on the `<a>`. We've used a clever technique to add an 'active' class when the contact's `id` matches the `selectedId` in our ContactList view-model. The `$parent` special value allows us to access the parent view-model's property. Throughout the list, we've used simple string interpolation to display each contact's `firstName`, `lastName`, and `email`.

The `route-href` attribute on the `<a>` is particularly interesting. This Aurelia-provided attribute generates a href for a route based on its name and parameters. We're using the "contacts" route name (remember our configuration?) and binding the contact's id as the route parameter. This allows the router to generate the correct href for each contact.

We've also included a click event. Why? For instant user feedback. The select method immediately updates the selected contact's id, applying the selection style instantly. Returning `true` from this method allows the default action (following the href) to proceed, triggering the router navigation.

To implement this contact list, update your `app.html` with the following markup:

```html
<template>
  <require from="./styles.css"></require>
  <require from="./contact-list"></require>

  <nav class="navbar navbar-light bg-light border-bottom fixed-top" role="navigation">
    <a class="navbar-brand" href="#">
      <i class="fa fa-user"></i>
      <span>Contacts</span>
    </a>
  </nav>

  <div class="container-md">
    <div class="row">
      <contact-list class="col-sm-5 col-md-4"></contact-list>
      <router-view class="col-sm-7 col-md-8"></router-view>
    </div>
  </div>
</template>
```

We've made two key changes here. First, we've imported our new `contact-list` at the top using a `require` element. This makes the `contact-list` available within this view, as views are encapsulated like modules. Second, we've placed the custom element just above our `router-view`.

Fire up the application, and you should see something like this:

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## Building the contact detail screen

Ok, things are starting to come together, but we still can't view an individual contact. If you try selecting something from the list, you'll see an error like the following in the console:

```sh
ERROR [app-router] Error: Unable to find module with ID: contact-detail.html
```

Again, this is because the router is trying to route to the detail screen, but we only have a stub component with no view. So, let's build out the real detail component. Replace the contents of `contact-detail.js/ts` with the following code:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import {inject} from 'aurelia-framework';
import {WebAPI} from './web-api';
import {areEqual} from './utility';

@inject(WebAPI)
export class ContactDetail {
  constructor(api){
    this.api = api;
  }

  activate(params, routeConfig) {
    this.routeConfig = routeConfig;

    return this.api.getContactDetails(params.id).then(contact => {
      this.contact = contact;
      this.routeConfig.navModel.setTitle(contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(contact));
    });
  }

  get canSave() {
    return this.contact.firstName && this.contact.lastName && !this.api.isRequesting;
  }

  save() {
    this.api.saveContact(this.contact).then(contact => {
      this.contact = contact;
      this.routeConfig.navModel.setTitle(contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(contact));
    });
  }

  canDeactivate() {
    if (!areEqual(this.originalContact, this.contact)){
      return confirm('You have unsaved changes. Are you sure you wish to leave?');
    }

    return true;
  }
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import {inject} from 'aurelia-framework';
import {WebAPI} from './web-api';
import {areEqual} from './utility';

interface Contact {
  firstName: string;
  lastName: string;
  email: string;
}

@inject(WebAPI)
export class ContactDetail {
  routeConfig;
  contact: Contact;
  originalContact: Contact;

  constructor(private api: WebAPI) { }

  activate(params, routeConfig) {
    this.routeConfig = routeConfig;

    return this.api.getContactDetails(params.id).then(contact => {
      this.contact = <Contact>contact;
      this.routeConfig.navModel.setTitle(this.contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(this.contact));
    });
  }

  get canSave() {
    return this.contact.firstName && this.contact.lastName && !this.api.isRequesting;
  }

  save() {
    this.api.saveContact(this.contact).then(contact => {
      this.contact = <Contact>contact;
      this.routeConfig.navModel.setTitle(this.contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(this.contact));
    });
  }

  canDeactivate() {
    if (!areEqual(this.originalContact, this.contact)) {
      return confirm('You have unsaved changes. Are you sure you wish to leave?');
    }

    return true;
  }
}
```
{% endtab %}
{% endtabs %}

We use dependency injection to obtain a `WebAPI` instance for loading contact details. The `activate` method, a lifecycle hook for routed components, is called just before the router activates the component. This method receives route parameters and is where we'll fetch our contact data.

The activate method's first argument, params, contains route and query string parameters. Our route pattern "contacts/:id" means params will have an id property for the requested contact. We use this to fetch contact data via the WebAPI, storing the result in the contact property for easy binding. We also keep a copy in `originalContact` for later comparison.

The second argument, `routeConfig`, is the router's configuration object. We use its `navModel` to dynamically set the document title with the loaded contact's name.

The `canDeactivate` hook, another part of the navigation lifecycle, is called before navigating away from the component. It allows us to prevent navigation if needed. Here, we compare `originalContact` with the current contact to check for unsaved changes, prompting the user if necessary.

The save method calls the WebAPI's `saveContact` method, updates `originalContact`, and refreshes the document title.

Lastly, we have a `canSave` computed property that we'll use in the view to provide user feedback on whether saving is currently possible.

Now, let's create the view for this component in a new file named `contact-detail.html`.

```html
<template>
  <div class="card">
    <div class="card-header text-white bg-primary">
      Profile
    </div>
    <div class="card-body">
      <form role="form">
        <div class="form-group row">
          <label class="col-md-3 col-form-label">First Name</label>
          <div class="col-md-9">
            <input type="text" placeholder="first name" class="form-control" value.bind="contact.firstName">
          </div>
        </div>

        <div class="form-group row">
          <label class="col-md-3 col-form-label">Last Name</label>
          <div class="col-md-9">
            <input type="text" placeholder="last name" class="form-control" value.bind="contact.lastName">
          </div>
        </div>

        <div class="form-group row">
          <label class="col-md-3 col-form-label">Email</label>
          <div class="col-md-9">
            <input type="text" placeholder="email" class="form-control" value.bind="contact.email">
          </div>
        </div>

        <div class="form-group row">
          <label class="col-md-3 col-form-label">Phone Number</label>
          <div class="col-md-9">
            <input type="text" placeholder="phone number" class="form-control" value.bind="contact.phoneNumber">
          </div>
        </div>
      </form>
      <div>
        <button class="btn btn-success float-right" click.delegate="save()" disabled.bind="!canSave">Save</button>
      </div>
    </div>
  </div>
</template>
```

The HTML may look extensive, but it's primarily composed of standard form controls and Bootstrap structures. Each input element is two-way bound to the corresponding contact property. The save button at the bottom is key: it triggers the save function on click and its disabled state is bound to the canSave property. This prevents users from saving during API requests or when contact information is incomplete.

With this in place, you can now select contacts from the list, view and edit their details, save changes, and see confirmation dialogs for unsaved data navigation. Your interface should resemble the following:

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## Adding pub/sub messaging

During testing, you may notice a few issues with the application:

1. Browser refreshes don't sync the selected contact with the highlighted list item.
2. Cancelling navigation after editing data causes the contact list selection to become misaligned.
3. Saving edits to a contact's name doesn't update the list view.

These problems stem from the `contact-list` and `contact-detail` components operating independently, despite their interconnected nature. The router, which controls the contact detail screen, should be the source of truth for both components.

To resolve these issues, we'll implement a pub/sub-system. This will allow the `contact-detail` component to publish updates, which the `contact-list` can then subscribe to and respond accordingly. Let's create the necessary messages to facilitate this communication.

{% tabs %}
{% tab title="Javascript" %}
{% code title="messages.js" %}
```javascript
export class ContactUpdated {
  constructor(contact) {
    this.contact = contact;
  }
}

export class ContactViewed {
  constructor(contact) {
    this.contact = contact;
  }
}
```
{% endcode %}
{% endtab %}

{% tab title="TypeScript" %}
{% code title="messages.ts" %}
```typescript
export class ContactUpdated {
  constructor(public contact) { }
}

export class ContactViewed {
  constructor(public contact) { }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

When the contact detail screen successfully saves a contact, it publishes a `ContactUpdated` message. Similarly, when a user starts viewing a new contact, it publishes a `ContactViewed` message. Both messages include the contact data, providing context for subscribers.

Let's modify our `contact-detail` code to use Aurelia's EventAggregator and publish these messages at the right moments:

{% tabs %}
{% tab title="Javascript" %}
{% code title="contact-detail.js" %}
```javascript
import {inject} from 'aurelia-framework';
import {EventAggregator} from 'aurelia-event-aggregator';
import {WebAPI} from './web-api';
import {ContactUpdated,ContactViewed} from './messages';
import {areEqual} from './utility';

@inject(WebAPI, EventAggregator)
export class ContactDetail {
  constructor(api, ea){
    this.api = api;
    this.ea = ea;
  }

  activate(params, routeConfig) {
    this.routeConfig = routeConfig;

    return this.api.getContactDetails(params.id).then(contact => {
      this.contact = contact;
      this.routeConfig.navModel.setTitle(contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(contact));
      this.ea.publish(new ContactViewed(this.contact));
    });
  }

  get canSave() {
    return this.contact.firstName && this.contact.lastName && !this.api.isRequesting;
  }

  save() {
    this.api.saveContact(this.contact).then(contact => {
      this.contact = contact;
      this.routeConfig.navModel.setTitle(contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(contact));
      this.ea.publish(new ContactUpdated(this.contact));
    });
  }

  canDeactivate() {
    if(!areEqual(this.originalContact, this.contact)){
      let result = confirm('You have unsaved changes. Are you sure you wish to leave?');

      if(!result) {
        this.ea.publish(new ContactViewed(this.contact));
      }

      return result;
    }

    return true;
  }
}
```
{% endcode %}
{% endtab %}

{% tab title="TypeScript" %}
{% code title="contact-detail.ts" %}
```typescript
import {inject} from 'aurelia-framework';
import {EventAggregator} from 'aurelia-event-aggregator';
import {WebAPI} from './web-api';
import {ContactUpdated,ContactViewed} from './messages';
import {areEqual} from './utility';

interface Contact {
  firstName: string;
  lastName: string;
  email: string;
}

@inject(WebAPI, EventAggregator)
export class ContactDetail {
  routeConfig;
  contact: Contact;
  originalContact: Contact;

  constructor(private api: WebAPI, private ea: EventAggregator) { }

  activate(params, routeConfig) {
    this.routeConfig = routeConfig;

    return this.api.getContactDetails(params.id).then(contact => {
      this.contact = <Contact>contact;
      this.routeConfig.navModel.setTitle(this.contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(this.contact));
      this.ea.publish(new ContactViewed(this.contact));
    });
  }

  get canSave() {
    return this.contact.firstName && this.contact.lastName && !this.api.isRequesting;
  }

  save() {
    this.api.saveContact(this.contact).then(contact => {
      this.contact = <Contact>contact;
      this.routeConfig.navModel.setTitle(this.contact.firstName);
      this.originalContact = JSON.parse(JSON.stringify(this.contact));
      this.ea.publish(new ContactUpdated(this.contact));
    });
  }

  canDeactivate() {
    if(!areEqual(this.originalContact, this.contact)){
      let result = confirm('You have unsaved changes. Are you sure you wish to leave?');

      if(!result) {
        this.ea.publish(new ContactViewed(this.contact));
      }

      return result;
    }

    return true;
  }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

We've imported Aurelia's EventAggregator and set it up for injection into the ContactDetail class constructor. We've also imported our custom messages. The component now publishes:

1. ContactViewed when loading a contact
2. ContactUpdated after saving a contact
3. ContactViewed again if the user cancels navigation away from the current contact

This setup allows other components to subscribe to these events and react accordingly. Let's update the `contact-list` component to stay in sync with these changes:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import {EventAggregator} from 'aurelia-event-aggregator';
import {WebAPI} from './web-api';
import {ContactUpdated, ContactViewed} from './messages';
import {inject} from 'aurelia-framework';

@inject(WebAPI, EventAggregator)
export class ContactList {
  constructor(api, ea) {
    this.api = api;
    this.contacts = [];

    ea.subscribe(ContactViewed, msg => this.select(msg.contact));
    ea.subscribe(ContactUpdated, msg => {
      let id = msg.contact.id;
      let found = this.contacts.find(x => x.id == id);
      Object.assign(found, msg.contact);
    });
  }

  created() {
    this.api.getContactList().then(contacts => this.contacts = contacts);
  }

  select(contact) {
    this.selectedId = contact.id;
    return true;
  }
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import {EventAggregator} from 'aurelia-event-aggregator';
import {WebAPI} from './web-api';
import {ContactUpdated, ContactViewed} from './messages';
import {inject} from 'aurelia-framework';

@inject(WebAPI, EventAggregator)
export class ContactList {
  contacts;
  selectedId = 0;

  constructor(private api: WebAPI, ea: EventAggregator) {
    ea.subscribe(ContactViewed, msg => this.select(msg.contact));
    ea.subscribe(ContactUpdated, msg => {
      let id = msg.contact.id;
      let found = this.contacts.find(x => x.id == id);
      Object.assign(found, msg.contact);
    });
  }

  created() {
    this.api.getContactList().then(contacts => this.contacts = contacts);
  }

  select(contact) {
    this.selectedId = contact.id;
    return true;
  }
}
```
{% endtab %}
{% endtabs %}

We've imported and injected the EventAggregator, then used its subscribe method with a message type and callback. When the message is published, the callback fires, receiving the message instance. In our case, these messages update our list's selection and relevant contact details.

Give it a try now – the application should work as intended.

## Adding a loading indicator

Let's put the finishing touches on our app by adding a loading indicator. This will appear at the top of the screen during navigation and WebAPI requests. We'll use a third-party library called nprogress and wrap it in a custom Aurelia element.

First, install nprogress:

```sh
npm install nprogress
```

{% hint style="info" %}
Typescript users should note that when using 3rd party libraries, in order to make them work in a TypeScript project, you may need to acquire (or create) d.ts files. See TypeScript's official documentation, if you encounter issues.
{% endhint %}

With that in place, let's create our `loading-indicator` custom element. In the `src/resources/elements` folder create a file named `loading-indicator.js/ts` and use the code below for its implementation:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import * as nprogress from 'nprogress';
import {bindable, noView} from 'aurelia-framework';
import 'nprogress/nprogress.css';

@noView
export class LoadingIndicator {
  @bindable loading = false;

  loadingChanged(newValue) {
    if (newValue) {
      nprogress.start();
    } else {
      nprogress.done();
    }
  }
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import * as nprogress from 'nprogress';
import { bindable, noView, PLATFORM } from 'aurelia-framework';
import 'nprogress/nprogress.css';

@noView
export class LoadingIndicator {
  @bindable loading = false;

  loadingChanged(newValue) {
    if (newValue) {
      nprogress.start();
    } else {
      nprogress.done();
    }
  }
}
```
{% endtab %}
{% endtabs %}

In this custom element, we're taking a slightly different approach. Since NProgress handles all the rendering, we use the `noView()` decorator to skip Aurelia's templating process. We've also imported the necessary CSS for NProgress to function correctly.

We've added a bindable 'loading' property, which allows us to control the element through an HTML attribute. Following Aurelia's conventions, we've implemented a `loadingChanged` method. This method toggles the NProgress indicator based on the 'loading' property's value.

Unlike our previous `contact-list` component, we're going to register this as a global resource. This approach allows us to use the element across multiple views without repeatedly importing it. To achieve this, update your `resources/index.js/ts` file with the following registration:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import {PLATFORM} from 'aurelia-framework';

export function configure(config) {
  config.globalResources([PLATFORM.moduleName('./elements/loading-indicator')]);
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import {FrameworkConfiguration, PLATFORM} from 'aurelia-framework';

export function configure(config: FrameworkConfiguration) {
  config.globalResources([PLATFORM.moduleName('./elements/loading-indicator')]);
}
```
{% endtab %}
{% endtabs %}

Now that we've registered our indicator, let's integrate it with our app. First, we'll modify `app.js/ts` to expose the API request state. This allows us to bind the indicator to our API's status. Update your `app.js/ts` like this:

{% tabs %}
{% tab title="Javascript" %}
```javascript
import {inject, PLATFORM} from 'aurelia-framework';
import {WebAPI} from './web-api';

@inject(WebAPI)
export class App {
  constructor(api) {
    this.api = api;
  }

  configureRouter(config, router) {
    config.title = 'Contacts';
    config.options.pushState = true;
    config.options.root = '/';
    config.map([
      { route: '',              moduleId: PLATFORM.moduleName('no-selection'),   title: 'Select'},
      { route: 'contacts/:id',  moduleId: PLATFORM.moduleName('contact-detail'), name:'contacts' }
    ]);

    this.router = router;
  }
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import {Router, RouterConfiguration} from 'aurelia-router';
import {inject, PLATFORM} from 'aurelia-framework';
import {WebAPI} from './web-api';

@inject(WebAPI)
export class App {
  router: Router;

  constructor(public api: WebAPI) {}

  configureRouter(config: RouterConfiguration, router: Router) {
    config.title = 'Contacts';
    config.options.pushState = true;
    config.options.root = '/';
    config.map([
      { route: '',              moduleId: PLATFORM.moduleName('no-selection'),   title: 'Select'},
      { route: 'contacts/:id',  moduleId: PLATFORM.moduleName('contact-detail'), name:'contacts' }
    ]);

    this.router = router;
  }
}
```
{% endtab %}
{% endtabs %}

Ok, now that we've got an `api` property we can bind to, update your `app.html` to the final version that adds the `loading-indicator` and binds its `loading` property:

```html
<template>
  <require from="./styles.css"></require>
  <require from="./contact-list"></require>

  <nav class="navbar navbar-light bg-light border-bottom fixed-top" role="navigation">
    <a class="navbar-brand" href="#">
      <i class="fa fa-user"></i>
      <span>Contacts</span>
    </a>
  </nav>

  <loading-indicator loading.bind="router.isNavigating || api.isRequesting"></loading-indicator>

  <div class="container-md">
    <div class="row">
      <contact-list class="col-sm-5 col-md-4"></contact-list>
      <router-view class="col-sm-7 col-md-8"></router-view>
    </div>
  </div>
</template>
```

And with that, we've finished our app.

## Next steps

Congratulations on completing the tutorial! To further develop your Aurelia skills, consider these next steps:

1. Implement a real backend and use http-client or fetch-client for data retrieval.
2. Add functionality to create new contacts.
3. Enhance the contact detail form with data validation.
4. Dive deeper into the component lifecycle.
5. Explore the navigation lifecycle and advanced routing techniques.
6. Master binding and templating concepts.

These challenges will help you build more complex, real-world applications with Aurelia. Happy coding!
