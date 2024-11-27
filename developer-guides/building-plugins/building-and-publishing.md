# Building and Publishing

### Building the Plugin

Use the Aurelia CLI to build your plugin:

```bash
au build-plugin
```

This command generates two module formats:

* `dist/native-modules/`: ES2015 module format
* `dist/commonjs/`: CommonJS module format

#### Build Output Structure

```
dist/
├── native-modules/
│   ├── index.js
│   ├── elements/
│   ├── attributes/
│   └── ...
└── commonjs/
    ├── index.js
    ├── elements/
    ├── attributes/
    └── ...
```

### Consuming the Plugin

#### Direct Git Reference

Install directly from a Git repository:

```bash
# GitHub
npm install github:username/repo-name

# Bitbucket
npm install bitbucket:username/repo-name

# GitLab
npm install gitlab:username/repo-name
```

#### Loading the Plugin in an Aurelia App

```javascript
// main.js
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin(PLATFORM.moduleName('your-plugin-name'));
}
```

### Publishing to npm

#### Prepare for Publication

1. Remove `"private": true` from `package.json`
2. Update version and publish

```bash
# Run tests
npm test

# Bump version (patch/minor/major)
npm version patch

# Publish to npm
npm publish
```

#### Automated Changelog and Publishing

Install changelog utilities:

```bash
npm install -D standard-changelog standard-version
```

Update `package.json` scripts:

```json
{
  "scripts": {
    "version": "standard-changelog && git add CHANGELOG.md",
    "postversion": "git push && git push --tags && npm publish"
  }
}
```
