# Orbiton CLI Usage Guide

Orbiton is the command-line interface (CLI) tool for the Orbit Framework. It provides everything you need to create, develop, and build Orbit applications.

## Installation

Install Orbiton using Cargo:

```bash
cargo install orbiton
```

## Command Overview

```
orbiton [command] [options]
```

Available commands:

- `new` - Create a new Orbit project
- `dev` - Start the development server
- `build` - Build the project for production
- `component` - Generate a new component
- `analyze` - Analyze your Orbit project
- `renderer` - Manage renderer configuration
- `help` - Display help information

## Creating a New Project

```bash
orbiton new [project-name] [options]
```

Options:
- `--template <template>` - Project template to use (default: "app")
- `--renderer <renderer>` - Initial renderer (choices: "skia", "wgpu", "hybrid", default: "hybrid")
- `--skip-git` - Skip Git repository initialization
- `--skip-install` - Skip dependency installation

Examples:

```bash
# Create a basic application
orbiton new my-orbit-app

# Create a component library with Skia renderer
orbiton new my-components --template library --renderer skia

# Create a project without initializing Git
orbiton new quick-test --skip-git
```

## Development Server

```bash
orbiton dev [options]
```

The `orbiton dev` command starts the development server, enabling features like hot module replacement (HMR), static asset serving, and WebSocket communication for live updates.

Note: When developing a web application, `orbiton dev` will expose an HTTP port (default 3000) and a WebSocket port (default HTTP port +1) to serve the app and enable HMR. CLI-only commands (e.g., `orbiton analyze`, `orbiton component`) run entirely in the terminal and do not open any listening ports. For desktop or embedded GUI applications (when targeting non-web environments), no HTTP or WebSocket ports are exposed unless explicitly configured for maintenance or diagnostic services.

**Standard Options:**

- `--port <port>` - Port to run the HTTP server on (default: 3000). The WebSocket server will run on `<port> + 1`.
- `--host <host>` - Host address to bind to (default: "localhost" for HTTP, "0.0.0.0" for WebSocket allowing broader network access for testing on other devices).
- `--no-open` - Don't automatically open the application in your default browser.
- `--renderer <renderer>` - Override the project's configured renderer for this development session (e.g., "skia", "wgpu").

**Advanced Features & Internals:**

Based on `orbiton/src/dev_server.rs`:

*   **HTTP Server:**
    *   Serves static files from your project directory.
    *   If a requested URL is empty (e.g., `http://localhost:3000/`), it defaults to serving `index.html` from the project root.
    *   Returns a 404 error if a requested file is not found.
    *   The HTTP server (using `tiny_http`) listens on the specified `--port`.
*   **WebSocket Server:**
    *   Runs on `port + 1` (e.g., if HTTP is on 3000, WebSocket is on 3001).
    *   Used for Hot Module Replacement (HMR) and other live updates.
    *   When changes are detected in your project (e.g., saving an `.orbit` file), the server broadcasts an update message to all connected WebSocket clients.
    *   Clients (your application running in the browser) receive these messages and can react accordingly (e.g., reload a component, refresh the page).
*   **Hot Module Replacement (HMR):**
    *   The `broadcast_update` method in `DevServer` is responsible for sending messages that trigger HMR or live reloads in connected clients.
    *   The exact mechanism of HMR (how components are swapped without a full page reload) depends on the Orbit runtime and client-side HMR logic.
*   **Project Directory:**
    *   The dev server serves files relative to the root of your Orbit project.
*   **Concurrency:**
    *   The HTTP and WebSocket servers run in separate Tokio tasks, managed within a dedicated thread for the `DevServer`.

**Configuration via `orbit.config.toml`:**

You can set default development server options in your `orbit.config.toml` file:

```toml
[dev]
port = 3001       # Default port for orbiton dev
open = false      # Prevent auto-opening the browser
hot_reload = true # Ensure HMR is enabled (usually default)
# host = "0.0.0.0"  # Example: Make server accessible on your local network
```

CLI options (e.g., `orbiton dev --port 3002`) will override settings in `orbit.config.toml` for that specific session.

**Debugging the Dev Server:**

*   Logs from the dev server (e.g., request information, WebSocket connections) can be viewed in the terminal where you ran `orbiton dev`.
*   Increase log verbosity using the `ORBIT_LOG_LEVEL` environment variable if needed (e.g., `ORBIT_LOG_LEVEL=debug orbiton dev`).

## Building for Production

```bash
orbiton build [options]
```

Options:
- `--target <target>` - Build target (choices: "web", "desktop", "embedded", "all", default: "all")
- `--release` - Build in release mode
- `--optimize <level>` - Optimization level (choices: "size", "speed", "balanced", default: "balanced")

Examples:

```bash
# Build for all platforms in release mode
orbiton build --release

# Build optimized for web only
orbiton build --target web --optimize size

# Build desktop version with speed optimization
orbiton build --target desktop --optimize speed
```

## Generating Components

```bash
orbiton component [name] [options]
```

The `orbiton component` command helps you create new components following best practices. It generates all the necessary files with boilerplate code, letting you focus on implementation rather than setup.

### Options

- `--path <path>` - Custom path for the component (default: "src/components")
- `--template <template>` - Component template to use (choices: "basic", "stateful", "layout", "form", "list", "modal", "container", default: "basic")
- `--style <style>` - Style approach to use (choices: "scoped", "global", "none", default: "scoped")
- `--test` - Generate accompanying test file
- `--story` - Generate component story for documentation
- `--props <props>` - Comma-separated list of props to include (e.g., "name:string,disabled:bool")

### Templates

Orbiton provides several component templates to help you get started quickly:

- **basic** - Minimal component with props
- **stateful** - Component with internal state management
- **layout** - Layout component with slots for content
- **form** - Form component with validation support
- **list** - Component for rendering lists with virtualization support
- **modal** - Modal/dialog component with accessibility features
- **container** - Container component with state management and child context

### Examples

```bash
# Generate a basic button component
orbiton component button

# Generate a stateful form component in a custom directory
orbiton component user-form --template form --path src/features/user

# Generate a modal component with test and documentation
orbiton component confirmation-modal --template modal --test --story

# Generate a component with predefined props
orbiton component data-table --props "data:Array<Record>,sortable:bool,paginated:bool"

# Generate a layout component with global styles
orbiton component page-layout --template layout --style global
```

### Generated Files

When you generate a component, Orbiton creates the following files (using `button` as an example):

```
src/components/
  button/
    Button.orbit       # Main component file
    Button.test.ts     # Test file (if --test was specified)
    Button.story.ts    # Story file (if --story was specified)
    index.ts           # Re-export file for easier imports
```

### Component Structure

The generated component follows the Orbit component model with sections for template, style, and script:

```
<!-- Basic component template -->
<template>
  <button class="orbit-button" @click="handleClick">
    <slot></slot>
  </button>
</template>

<style>
  .orbit-button {
    /* Default styling */
    padding: 8px 16px;
    border-radius: 4px;
    border: none;
    cursor: pointer;
    background-color: var(--orbit-primary-color, #0066cc);
    color: white;
  }

  .orbit-button:hover {
    background-color: var(--orbit-primary-hover-color, #0052a3);
  }

  .orbit-button:focus {
    outline: 2px solid var(--orbit-focus-color, #4d90fe);
    outline-offset: 2px;
  }
</style>

<script>
use orbit::prelude::*;

#[derive(Props)]
pub struct ButtonProps {
    #[prop(default)]
    pub disabled: bool,
    
    #[prop(default)]
    pub variant: String,
    
    #[prop(event)]
    pub on_click: Option<EventHandler>,
}

pub struct Button {
    props: ButtonProps,
}

impl Component for Button {
    type Props = ButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
    
    fn handleClick(&mut self, event: ClickEvent) {
        if !self.props.disabled {
            if let Some(handler) = &self.props.on_click {
                handler.emit(event);
            }
        }
    }
}
</script>
```

### Best Practices

- **Naming Convention**: Use PascalCase for component names (e.g., `DataTable` instead of `data-table`)
- **Directory Structure**: Place related components in feature-specific directories
- **Reusability**: Design components to be reusable with flexible props
- **Testing**: Always generate test files for components using the `--test` option
- **Documentation**: Generate story files with examples using the `--story` option
- **Props**: Use typed props with defaults where appropriate

## Project Analysis

```bash
orbiton analyze [options]
```

The `orbiton analyze` command is a powerful tool for examining your Orbit project to identify potential issues, enforce best practices, and optimize performance. It leverages the Orlint analysis engine to provide actionable insights for improving your application.

### Options

- `--fix` - Automatically fix issues when possible
- `--verbose` - Show detailed analysis information
- `--format <format>` - Output format (choices: "text", "json", "html", default: "text")
- `--output <file>` - Write output to file instead of stdout
- `--include <patterns>` - Glob patterns for files to include (default: "src/**/*.orbit")
- `--exclude <patterns>` - Glob patterns for files to exclude
- `--config <path>` - Path to custom analysis configuration (default: ".orlint.toml")
- `--rules <rules>` - Comma-separated list of rule IDs to include
- `--ignore-rules <rules>` - Comma-separated list of rule IDs to exclude
- `--renderer <renderer>` - Analyze with a specific renderer context (e.g., "skia", "wgpu")

### Analysis Categories

`orbiton analyze` examines your project in several key areas:

- **Syntax Validation**: Ensures all `.orbit` files follow the correct syntax
- **Component Structure**: Verifies components follow Orbit's component model
- **Performance**: Identifies potential performance bottlenecks
- **Accessibility**: Flags accessibility issues and suggests improvements
- **Best Practices**: Enforces Orbit framework conventions and patterns
- **Dependencies**: Analyzes component dependencies and identifies circular references
- **Renderer Compatibility**: Validates components work with the configured renderers

### Examples

```bash
# Run basic analysis on the project
orbiton analyze

# Automatically fix issues when possible
orbiton analyze --fix

# Generate detailed JSON output for CI/CD integration
orbiton analyze --format json --output analysis-report.json

# Analyze only specific components
orbiton analyze --include "src/features/dashboard/**/*.orbit"

# Exclude test components from analysis
orbiton analyze --exclude "**/*.test.orbit,**/*.spec.orbit"

# Focus on accessibility rules
orbiton analyze --rules "a11y-*"

# Ignore specific performance rules
orbiton analyze --ignore-rules "perf-no-inline-styles,perf-optimize-loops"

# Run analysis in verbose mode for debugging
orbiton analyze --verbose
```

### Analysis Report

The analysis report contains:

- Summary of files analyzed and issues found
- Categorized list of issues by severity (error, warning, info)
- Source locations with line and column numbers
- Descriptions of issues with links to documentation
- Suggested fixes for common problems

Example report output:

```
Analysis Summary:
- Files analyzed: 42
- Errors: 3
- Warnings: 12
- Info: 5

Errors:
[E001] src/components/DataTable.orbit:24:5
  Component missing required accessibility attributes for interactive elements
  Suggestion: Add aria-label or aria-labelledby attribute to the table element

Warnings:
[W023] src/features/dashboard/Chart.orbit:56:10
  Inefficient rendering pattern detected - component may re-render unnecessarily
  Suggestion: Implement shouldUpdate method or use memoization

Info:
[I008] src/layouts/MainLayout.orbit:7:3
  Consider extracting repeated style patterns into shared styles
  Suggestion: Create a common style file for layout components
```

### Configuration File

You can customize the analysis with an `.orlint.toml` file in your project root:

```toml
# .orlint.toml
[analyzer]
include = ["src/**/*.orbit"]
exclude = ["**/test/**", "**/node_modules/**"]

[rules]
disabled = ["perf-no-inline-styles"] # Rules to disable
warning = ["a11y-*"] # Rules to treat as warnings instead of errors

[performance]
rendering_threshold_ms = 16 # Flag components that may render slower than this

[accessibility]
level = "AA" # WCAG compliance level to enforce (A, AA, or AAA)
```

### Integration with CI/CD

`orbiton analyze` is designed to integrate with CI/CD pipelines to ensure code quality. For example, in a GitHub Actions workflow:

```yaml
name: Orbit Analysis

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Install Rust
      uses: actions-rs/toolchain@v1
      with:
        toolchain: stable
    - name: Install Orbiton
      run: cargo install orbiton
    - name: Run Analysis
      run: orbiton analyze --format json --output analysis.json
    - name: Upload Analysis Results
      uses: actions/upload-artifact@v4
      with:
        name: orbit-analysis
        path: analysis.json
```

### Learn More

For more detailed information on analysis rules and configuration options, see the [Orlint Documentation](../../orlint/docs/README.md).

## Orbiton CLI Configuration (`orbiton.toml`)

The Orbiton CLI can be configured using a file named `orbiton.toml` located in the project's root directory. This file allows you to set project-specific defaults for various CLI commands and options, streamlining your development workflow.

If an `orbiton.toml` file is not present, the CLI will use its default settings. Command-line arguments will always override any settings defined in `orbiton.toml`.

### Structure of `orbiton.toml`

The configuration file is organized into sections, each corresponding to an Orbiton CLI command (e.g., `[new]`, `[dev]`, `[build]`).

**Example `orbiton.toml`:**

```toml
# Global settings (if any, usually command-specific)
# project_name = "MyDefaultProjectName" # Example, not a standard global option

[new]
# Default options for `orbiton new`
template = "library" # Default project template
renderer = "skia"     # Default renderer for new projects
skip_git = false      # Initialize a Git repository by default
skip_install = false  # Install dependencies by default

[dev]
# Default options for `orbiton dev`
port = 3001
host = "0.0.0.0"
open = true # Automatically open in browser
# renderer = "wgpu" # Optionally override project's default renderer for dev

[build]
# Default options for `orbiton build`
target = "web"    # Default build target
release = true    # Build in release mode by default
optimize = "size" # Optimize for size by default

[component]
# Default options for `orbiton component`
path = "src/components" # Default path for new components
template = "basic"
style = "scoped"
test = true             # Generate test files by default
story = false           # Do not generate story files by default

[analyze]
# Default options for `orbiton analyze`
fix = false
verbose = false
format = "text"
# config = ".orlint.custom.toml" # Path to a custom Orlint config
# include = ["src/**/*.orbit", "ui/**/*.orbit"]
# exclude = ["**/__tests__/**"]

[renderer]
# Default renderer for the project (can be overridden by `dev.renderer` or `build.renderer` for specific commands)
# default_renderer = "auto" # Example: 'skia', 'wgpu', 'auto'

# Configuration for specific renderers, if applicable
# [renderer.skia_options]
# antialiasing = true

# [renderer.wgpu_options]
# power_preference = "high-performance"
```

### Supported Options

Each section in `orbiton.toml` can specify default values for the corresponding command's flags. For example, in the `[dev]` section, you can set `port = 3030` to make `orbiton dev` use port 3030 by default.

*   **`[new]`**: Corresponds to `orbiton new` options.
    *   `template`: e.g., "app", "library"
    *   `renderer`: e.g., "skia", "wgpu", "hybrid"
    *   `skip_git`: true or false
    *   `skip_install`: true or false
*   **`[dev]`**: Corresponds to `orbiton dev` options.
    *   `port`: e.g., 3000
    *   `host`: e.g., "localhost", "0.0.0.0"
    *   `open`: true or false
    *   `renderer`: e.g., "skia", "wgpu" (overrides project default for dev server)
*   **`[build]`**: Corresponds to `orbiton build` options.
    *   `target`: e.g., "web", "desktop", "embedded", "all"
    *   `release`: true or false
    *   `optimize`: e.g., "size", "speed", "balanced"
*   **`[component]`**: Corresponds to `orbiton component` options.
    *   `path`: e.g., "src/components", "app/ui"
    *   `template`: e.g., "basic", "stateful"
    *   `style`: e.g., "scoped", "global", "none"
    *   `test`: true or false
    *   `story`: true or false
*   **`[analyze]`**: Corresponds to `orbiton analyze` options.
    *   `fix`: true or false
    *   `verbose`: true or false
    *   `format`: e.g., "text", "json", "html"
    *   `output`: e.g., "analysis_report.json"
    *   `include`: e.g., `["src/**/*.orbit"]`
    *   `exclude`: e.g., `["**/node_modules/**"]`
    *   `config`: e.g., ".orlint.toml"
    *   `rules`: e.g., `["a11y-*"]`
    *   `ignore_rules`: e.g., `["perf-no-inline-styles"]`
    *   `renderer`: e.g., "skia"
*   **`[renderer]`**: General renderer configuration for the project.
    *   `default_renderer`: Sets the primary renderer (e.g., "skia", "wgpu", "auto"). This can be overridden by specific command configurations like `dev.renderer`.
    *   Specific renderer options can be nested, e.g., `[renderer.skia_options]`. (Actual option names would depend on Orbiton's implementation).

### Validation

The Orbiton CLI will parse `orbiton.toml` when a command is run. If the file contains invalid TOML syntax or unrecognized options, the CLI may issue a warning or error and fall back to default behavior or halt execution, depending on the severity.

It's recommended to refer to the latest Orbiton CLI documentation or use `orbiton help <command>` for the most up-to-date list of configurable options and their valid values.

## Renderer Management

The Orbit Framework supports multiple rendering backends to optimize performance across different platforms and use cases. The `renderer` command helps you manage and configure these renderers for your project.

```bash
orbiton renderer [command] [options]
```

### Available Commands

- `list` - List all available renderers and their current status
- `set <renderer>` - Set the default renderer for your project
- `info <renderer>` - Show detailed information about a specific renderer
- `config <renderer>` - Configure the renderer with additional options

### Renderer Options

Orbit supports the following rendering backends:

- **skia** - Skia-based renderer, optimized for 2D graphics and UI components with broad platform support
- **wgpu** - WebGPU-based renderer for high-performance graphics with hardware acceleration
- **auto** (default) - Automatically selects the best renderer based on the target platform and available hardware

### Command Options

```bash
orbiton renderer config --dir <project-directory>
```

Options:
- `--dir, -d <path>` - Specify the project directory (defaults to current directory)

### Configuration File

The renderer settings are stored in the `orbit.config.json` file in your project root. This is a JSON file that can be manually edited if needed.

Example configuration file:
```json
{
  "renderer": "wgpu",
  "rendererOptions": {
    "antialiasing": true,
    "hardware-acceleration": "prefer"
  }
}
```

### Renderer Comparison

| Feature | Skia | WebGPU | Notes |
|---------|------|--------|-------|
| 2D Graphics | ✅ Excellent | ✅ Good | Skia has more optimized 2D primitives |
| 3D Support | ⚠️ Limited | ✅ Excellent | WebGPU provides better 3D capabilities |
| Text Rendering | ✅ Excellent | ✅ Good | Skia has more advanced text features |
| Platform Support | ✅ Broad | ⚠️ Growing | Skia works on more platforms currently |
| Performance | ✅ Good | ✅ Excellent | WebGPU has better hardware acceleration |
| Memory Usage | ✅ Low | ⚠️ Higher | Skia is more memory-efficient for simple UIs |

### Usage Examples

```bash
# List all available renderers and their status
orbiton renderer list

# Set the default renderer for a project
orbiton renderer set wgpu

# Get detailed information about a renderer
orbiton renderer info skia

# Configure the renderer for a specific project
orbiton renderer config skia --dir ./my-orbit-project
```

### Best Practices

- Use **skia** for:
  - Applications targeting broad platform support
  - UI-heavy applications with limited graphics needs
  - Mobile applications where memory efficiency is critical

- Use **wgpu** for:
  - Applications with complex visualizations or 3D elements
  - Games and interactive experiences
  - Desktop applications where performance is critical

- Use **auto** for:
  - General-purpose applications
  - When you want to ensure the best experience across different devices

## Testing

The Orbit framework includes comprehensive testing tools to help ensure your applications are reliable and robust.

```bash
orbiton test [options]
```

> **Note**: This feature is currently in development and planned for a future release. The documentation below reflects the planned functionality.

### Options

- `--watch` - Run tests in watch mode, automatically rerunning when files change
- `--unit` - Run only unit tests
- `--integration` - Run only integration tests
- `--performance` - Run performance tests to measure rendering speed and memory usage
- `--coverage` - Generate test coverage information
- `--report` - Generate and display a detailed coverage report
- `--update-snapshots` - Update test snapshots instead of failing on mismatch
- `--verbose` - Show detailed test output

### Examples

```bash
# Run all tests
orbiton test

# Run tests and watch for changes
orbiton test --watch

# Run only unit tests
orbiton test --unit

# Run tests with coverage report
orbiton test --coverage --report

# Update test snapshots
orbiton test --update-snapshots
```

### Test Configuration

Tests can be configured in your project's `orbit.config.json` file:

```json
{
  "testing": {
    "coverage": {
      "threshold": 80,
      "excludes": ["**/*.test.js", "**/test/**"]
    },
    "performance": {
      "benchmarks": {
        "renderTime": 16,
        "memoryUsage": 10
      }
    }
  }
}
```

For more detailed information on testing strategies and best practices, refer to the [Testing Guide](/docs/guides/testing.md).

## Configuration

Orbiton uses a configuration file for project settings. By default, it looks for `orbit.config.toml` in the project root.

Example configuration:

```toml
[project]
name = "my-orbit-app"
version = "0.1.0"

[build]
target = ["web", "desktop"]
optimization = "balanced"

[renderer]
default = "hybrid"
web = "skia"
desktop = "wgpu"

[dev]
port = 3000
open = true
hot_reload = true
```

## Linting Your Project with Orlint

Orbiton integrates with Orlint, the Orbit Analyzer, to help you maintain code quality, enforce best practices, and catch potential issues in your `.orbit` files. While Orlint is a separate tool with its own comprehensive CLI, you can typically invoke it through Orbiton or run it directly in your project.

**Typical Usage (directly using Orlint):**

```bash
# Analyze all .orbit files in the src directory
orlint analyze src/

# Analyze a specific component
orlint analyze src/components/MyButton.orbit

# Generate an initial Orlint configuration file (.orlint.toml)
orlint init
```

**Key Orlint Features:**

*   **Comprehensive Analysis:** Checks for syntax errors, adherence to Orbit best practices, potential performance issues, and more.
*   **Configurable Rules:** Customize which rules to apply or exclude via an `.orlint.toml` configuration file.
*   **Multiple Output Formats:** Get reports in text, JSON, or HTML.
*   **Incremental Analysis:** Speed up linting by only analyzing changed files.

**Integration with Orbiton CLI (Conceptual):**

While `orbiton-cli.md` previously listed an `orbiton lint` command, direct invocation of `orlint` is the standard approach. If an `orbiton lint` command is (re)introduced, it would likely serve as a wrapper around `orlint analyze`.

**Detailed Orlint Documentation:**

For a complete guide to Orlint, including all commands, options, configuration details, and available rules, please refer to the dedicated Orlint documentation:

*   [Orlint CLI Usage Guide](../../orlint/docs/cli-usage.md)
*   [Orlint Configuration Guide](../../orlint/docs/configuration.md)

## Environment Variables

Orbiton respects the following environment variables:

- `ORBIT_CONFIG_PATH` - Custom path to the configuration file
- `ORBIT_RENDERER` - Override the default renderer
- `ORBIT_DEV_PORT` - Override the development server port
- `ORBIT_LOG_LEVEL` - Set the log level (error, warn, info, debug, trace)

## Common Workflows

### Complete Project Development Workflow

1. Create a new project:
   ```bash
   orbiton new my-app
   cd my-app
   ```

2. Generate components:
   ```bash
   orbiton component header
   orbiton component footer
   orbiton component user-card
   ```

3. Start development server:
   ```bash
   orbiton dev
   ```

4. Analyze your project:
   ```bash
   orbiton analyze
   ```

5. Build for production:
   ```bash
   orbiton build --release
   ```

### Switching Renderers During Development

Test your application with different renderers:

```bash
# Start with Skia renderer
orbiton dev --renderer skia

# Then test with WGPU renderer
orbiton dev --renderer wgpu
```

## Troubleshooting

### Common Issues

**Error: Could not find orbit.config.toml**
- Make sure you're in the correct project directory
- Create a config file or specify a custom path with `ORBIT_CONFIG_PATH`

**Error: Failed to start development server**
- Check if the port is already in use
- Try specifying a different port with `--port`

**Error: Component generation failed**
- Verify the component name uses valid characters
- Check write permissions in the target directory

**Build fails with renderer errors**
- Ensure the selected renderer is properly configured
- Check compatibility with your target platform

### Getting Help

- Run `orbiton help` for general information
- Run `orbiton help [command]` for help with a specific command
- Check the [Orbit documentation](https://docs.orbitrs.dev)
- Join the [community Discord](https://discord.gg/orbitrs)

## Advanced Usage

### Custom Build Configurations

Create different build configurations for various environments:

```bash
# Development build
orbiton build --config dev.config.toml

# Production build
orbiton build --config prod.config.toml
```

### CI/CD Integration

Example GitHub Actions workflow:

```yaml
name: Orbit CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Install Rust
      uses: actions-rs/toolchain@v1
      with:
        toolchain: stable
    - name: Install Orbiton
      run: cargo install orbiton
    - name: Analyze project
      run: orbiton analyze
    - name: Build project
      run: orbiton build --release
```

### Plugin System

Orbiton supports plugins to extend functionality:

```bash
# Install a plugin
orbiton plugin install my-plugin

# Use plugin command
orbiton my-plugin [options]
```

## Next Steps

- Explore the [Orbit component library](../api/orbitkit-components.md)
- Learn about [advanced rendering options](../core-concepts/rendering-architecture.md)
- Check out [example applications](../examples/README.md)
