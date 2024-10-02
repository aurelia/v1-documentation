# Gulp tasks

The Aurelia CLI leverages Gulp as its task runner, providing a series of predefined tasks to streamline your development process. These tasks are located in the `aurelia_project/tasks` directory.

## Task Execution

Tasks can be executed using the `au` command. For example:

*   **Build the application:**

    ```bash
    au build
    ```

    Executes the Gulp task exported from `aurelia_project/tasks/build.js`.
*   **Run tests:**

    ```bash
    au test
    ```

    Executes the task from `aurelia_project/tasks/test.js`.

**Note:** The CLI executes the task exported as `default`. You can export multiple tasks from a single file:

```javascript
let task1 = () => {};
let task2 = () => {};

export { task1 as default, task2 };
```

Executing the above using `au task1` will run `task1`.

## Creating a New Task

You can create custom tasks to extend the CLI's functionality:

1. **Create a new file:** Add a `.js` or `.ts` file in the `aurelia_project/tasks` directory.
2.  **Export the task function:**

    ```javascript
    export default function myCustomTask() {
      // Task implementation
    }
    ```
3. **Run the task:** Use `au myCustomTask` to execute it.

## Task Metadata

To integrate custom tasks with the CLI help system, define task metadata in a `.json` file with the same name as the task. For example, for `build.js`, create `build.json`:

```json
{
  "name": "build",
  "description": "Builds and processes all application assets.",
  "flags": [
    {
      "name": "env",
      "description": "Sets the build environment.",
      "type": "string"
    },
    {
      "name": "watch",
      "description": "Watches source files for changes and refreshes the bundles automatically.",
      "type": "boolean"
    }
  ],
  "parameters": [
    {
      "name": "some-parameter",
      "description": "A description of this parameter."
    }
  ]
}
```

{% hint style="info" %}
Only tasks with corresponding metadata JSON files appear in the `au help` command.
{% endhint %}
