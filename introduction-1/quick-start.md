---
description: >-
  Welcome to Aurelia. This quick start guide will take you through creating a
  todo app using Aurelia and briefly explain its main concepts. We assume you
  are familiar with JavaScript, HTML, and CSS.
---

# Quick Start

## Setup

Dive into the world of Aurelia, a framework that strikes the perfect balance between simplicity and power. In this tutorial, we'll walk you through building a "Todo" application, showcasing how Aurelia empowers you to write clean, concise code without compromising on functionality.

For this tutorial, we're going to use the Aurelia CLI. You can skip to the next section if you've already set up your machine with the CLI. If not, then please install the following CLI prerequisites:

* Install Node.js (the LTS version is fine) from [here](https://nodejs.org/en/).
* Install a GIT client such as the standard GIT client from [here](https://git-scm.com/) or a nicer GUI client such as [GitHub desktop](https://github.com/apps/desktop)&#x20;

Once the prerequisites are installed, you can install the Aurelia CLI itself. From the command line, use npm to install the CLI globally:

```shell
npm install -g aurelia-cli
```

{% hint style="info" %}
Always run commands from a Bash/Terminal prompt. Depending on your environment, you may need to use `sudo` when executing npm global installs.
{% endhint %}

## Create an app

Ready to dive in? Great! Let's get started on our todo app.

First, open your command line. We'll use the `au new` command to kickstart our project.

When you run the command, a few options pop up. Here's what you need to do:

1. Name your project "todo" (keep it simple, right?)
2. Choose your preferred language:
   * If you're comfortable with the latest JavaScript features, go for "Default ESNext"
   * More of a TypeScript fan? Select "Default TypeScript"

After that, you'll be asked if you want to install your new project's dependencies. Press enter to select the default "yes" for this as well.

{% hint style="info" %}
We highly recommend choosing TypeScript for any new Aurelia applications. With TypeScript, you get the benefits of intellisense and type safety.
{% endhint %}

Now that you've installed the dependencies (which might take a few minutes), your project is ready to roll. Head over to your project folder and fire it up by running `au run --open`. This command will launch the app, open a new browser tab, and monitor your project's source files for any changes. If everything's set up correctly, you should see a friendly "Hello World!" message greeting you in the browser.

That's it! Now, you're all set to dive deeper into Aurelia development.

