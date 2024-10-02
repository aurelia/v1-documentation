# Environments

The Aurelia CLI supports multiple environments to manage different development, staging, and production configurations. By default, three environments are set up: `dev`, `stage`, and `prod`.

To run or build your app in a specific environment, use the `--env` flag:

```bash
au run --env prod
```

When you specify an environment, the CLI replaces `src/environment.js` or `src/environment.ts` with the corresponding file (`dev.js`, `stage.js`, or `prod.js`) from the `aurelia_project/environments` directory.
