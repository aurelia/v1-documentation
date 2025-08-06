# CLI command reference

This comprehensive reference covers all Aurelia CLI commands, their options, and usage patterns.

## Global Commands

### au new

Creates a new Aurelia project or plugin.

**Syntax:**
```bash
au new [project-name] [options]
```

**Options:**
- `--plugin` - Creates a plugin project instead of an application
- `--here` - Creates the project in the current directory
- `--unattended` - Uses default options without prompting
- `--typescript` - Uses TypeScript template
- `--webpack` - Uses Webpack bundler
- `--require-js` - Uses RequireJS module loader
- `--system-js` - Uses SystemJS module loader

**Examples:**
```bash
# Interactive project creation
au new my-app

# Create plugin project
au new my-plugin --plugin

# Create TypeScript app with Webpack
au new my-app --typescript --webpack

# Create project in current directory
au new --here
```

**Prerequisites:**
- Node.js v10.0.0 or higher
- Git client
- Available disk space (project templates ~50MB)

### au help

Displays help information for CLI commands.

**Syntax:**
```bash
au help [command]
```

**Examples:**
```bash
# General help
au help

# Help for specific command
au help new
au help generate
```

### au version / au -v

Displays the current CLI version.

**Syntax:**
```bash
au version
au -v
```

## Project Commands

These commands work within an Aurelia project directory.

### au run

Builds and serves the application for development.

**Syntax:**
```bash
au run [options]
```

**Options:**
- `--env <environment>` - Specifies environment (dev, stage, prod)
- `--port <port>` - Sets server port (default: 8080)
- `--host <host>` - Sets server host (default: localhost)
- `--open` - Opens browser automatically
- `--hmr` - Enables Hot Module Reload (Webpack only)
- `--analyze` - Runs webpack bundle analyzer (Webpack only)
- `--extractCss` - Extracts CSS to separate file (Webpack only)
- `--watch` - Watches for file changes and rebuilds
- `--auto-install` - Automatically installs missing packages (ESNext only)

**Examples:**
```bash
# Basic development server
au run

# Production environment
au run --env prod

# Custom port and auto-open browser
au run --port 3000 --open

# Hot module reload for Webpack projects
au run --hmr

# Auto-install missing dependencies
au run --auto-install
```

**Environment Variables:**
- `AURELIA_CLI_SILENCE_WARNINGS` - Suppresses CLI warnings
- `NODE_ENV` - Sets Node.js environment

### au build

Builds the application for production deployment.

**Syntax:**
```bash
au build [options]
```

**Options:**
- `--env <environment>` - Specifies build environment
- `--watch` - Watches for changes and rebuilds
- `--analyze` - Analyzes bundle composition (Webpack only)
- `--extractCss` - Extracts CSS to separate file (Webpack only)

**Examples:**
```bash
# Production build
au build --env prod

# Development build with watching
au build --env dev --watch

# Analyze bundle size
au build --analyze
```

**Output:**
- Built files go to `dist/` directory
- Source maps included based on environment configuration

### au test

Runs the project's test suite.

**Syntax:**
```bash
au test [options]
```

**Options:**
- `--watch` - Runs tests in watch mode
- `--debug` - Enables debug mode
- `--coverage` - Generates code coverage report
- `--env <environment>` - Specifies test environment

**Examples:**
```bash
# Run tests once
au test

# Watch mode for continuous testing
au test --watch

# Generate coverage report
au test --coverage
```

**Supported Test Frameworks:**
- Jest (default for new projects)
- Karma + Jasmine
- Protractor (e2e testing)

### au generate

Scaffolds Aurelia resources using built-in generators.

**Syntax:**
```bash
au generate <resource> <name> [options]
```

**Available Resources:**
- `element` - Custom element
- `attribute` - Custom attribute  
- `value-converter` - Value converter
- `binding-behavior` - Binding behavior
- `task` - Gulp task
- `generator` - Custom generator

**Options:**
- `--force` - Overwrites existing files

**Examples:**
```bash
# Generate custom element
au generate element user-profile

# Generate custom attribute
au generate attribute highlight

# Generate value converter
au generate value-converter date-format

# Generate binding behavior  
au generate binding-behavior throttle

# Generate gulp task
au generate task deploy

# Force overwrite existing files
au generate element my-element --force
```

**File Conventions:**
- Uses kebab-case for file names
- Generates both view-model and template files for elements
- Creates corresponding test files when test framework is configured

## Advanced Commands

### au install

Installs and configures Aurelia plugins and third-party packages.

**Syntax:**
```bash
au install <package-name> [options]
```

**Options:**
- `--save` - Adds to dependencies in package.json
- `--save-dev` - Adds to devDependencies
- `--force` - Forces reinstallation

**Examples:**
```bash
# Install Aurelia plugin
au install aurelia-validation

# Install with save to dependencies
au install lodash --save
```

**Auto-configuration:**
- Updates `aurelia.json` dependencies automatically
- Configures known Aurelia plugins
- Handles font and asset copying for UI libraries

### au clear-cache

Clears build caches to resolve build issues.

**Syntax:**
```bash
au clear-cache
```

**What it clears:**
- Webpack module cache
- TypeScript compilation cache
- CLI bundler cache
- Node modules cache

**When to use:**
- After major dependency updates
- When experiencing unexplained build errors
- After switching between branches with different dependencies

## Flag Reference

### Environment Flags

**Built-in Environments:**
- `dev` - Development (default)
- `stage` - Staging  
- `prod` - Production

**Custom Environments:**
Create additional environment files in `aurelia_project/environments/`

### Debug Flags

**Webpack Projects:**
- `--analyze` - Bundle analyzer
- `--extractCss` - CSS extraction
- `--hmr` - Hot module reload

**CLI Bundler Projects:**
- `--watch` - File watching
- `--auto-install` - Package auto-install

### Compatibility Flags

**Legacy Support:**
- `--require-js` - RequireJS loader
- `--system-js` - SystemJS loader

**Platform Specific:**
- Windows: Use CMD or PowerShell (not Git Bash)
- Docker: May need polling for file watching

## Error Handling

### Common Error Patterns

**Installation Errors:**
```bash
# Permission denied (Mac/Linux)
sudo npm install aurelia-cli -g

# Clear npm cache
npm cache clean --force
```

**Build Errors:**
```bash
# Clear all caches
au clear-cache

# Reinstall dependencies
rm -rf node_modules
npm install
```

**Module Resolution Errors:**
```bash
# For Webpack projects
# Ensure PLATFORM.moduleName() usage

# For CLI bundler projects  
# Check aurelia.json dependencies
```

### Troubleshooting Commands

```bash
# Check CLI version
au -v

# Validate project structure
au help

# Clear caches and rebuild
au clear-cache && au build

# Debug mode for detailed output
au run --env dev --debug
```

## Integration Commands

### Git Integration

```bash
# Create project with git
au new my-app
cd my-app
git init
git add .
git commit -m "Initial commit"
```

### Docker Integration

```bash
# Build for Docker
au build --env prod

# Dockerfile commands
FROM node:14
WORKDIR /app
COPY . .
RUN npm install && au build --env prod
```

### CI/CD Integration

```bash
# Install CLI in CI environment
npm install aurelia-cli -g

# Run tests in CI
au test --coverage

# Build for deployment
au build --env prod
```

## Performance Optimization

### Build Performance

```bash
# Enable caching in aurelia.json
"options": {
  "cache": true
}

# Use webpack analyzer
au build --analyze

# Extract CSS for better caching
au build --extractCss
```

### Development Performance

```bash
# Use auto-install for faster setup
au run --auto-install

# Enable HMR for instant updates
au run --hmr

# Use watch mode for continuous builds
au build --watch
```

## Best Practices

### Command Usage

1. **Always use specific environments:**
   ```bash
   au build --env prod  # Not just 'au build'
   ```

2. **Clear cache when in doubt:**
   ```bash
   au clear-cache
   ```

3. **Use generators for consistency:**
   ```bash
   au generate element user-card  # Not manual file creation
   ```

4. **Test before building:**
   ```bash
   au test && au build --env prod
   ```

### Project Organization

1. **Consistent naming:**
   - Use kebab-case for generated resources
   - Follow CLI naming conventions

2. **Environment configuration:**
   - Keep environment-specific settings in environment files
   - Use `aurelia.json` for shared configuration

3. **Dependency management:**
   - Use `au install` for Aurelia packages
   - Update `aurelia.json` for complex dependencies

The Aurelia CLI provides a comprehensive toolset for managing Aurelia applications throughout their development lifecycle. Understanding these commands and their options enables efficient development workflows and troubleshooting capabilities.