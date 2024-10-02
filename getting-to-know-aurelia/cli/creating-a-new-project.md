# Creating a new project

## Create a new project

To create a new Aurelia project, run the following command:

```bash
au new
```

You'll be prompted to choose a project name, select a bundler, a module loader, a transpiler, a CSS processor, and more. If you're unsure about any option, you can accept the defaults for a quick setup.

After you make your selections, the CLI summarizes them and asks for confirmation to create the project structure. You'll then be asked whether you want to install the project dependencies.

Once the dependencies are installed, your project is ready to use.

## Create a new plugin project

To create a new Aurelia plugin project, use the `--plugin` flag:

```bash
au new --plugin
```

This command initiates a scaffold tailored to create reusable Aurelia plugins.

{% hint style="warning" %}
Aurelia CLI commands may not work properly in Git Bash on Windows. Please use Command Prompt (CMD) or PowerShell instead.
{% endhint %}
