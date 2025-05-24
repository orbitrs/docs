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

The `orbiton build` command compiles your Orbit application for production deployment. It performs comprehensive optimizations including code splitting, asset bundling, and platform-specific optimizations to create efficient, deployable artifacts.

### Options

- `--target <target>` - Build target (choices: "web", "desktop", "embedded", "all", default: "all")
- `--release` - Build in release mode with full optimizations
- `--optimize <level>` - Optimization level (choices: "size", "speed", "balanced", default: "balanced")
- `--output <dir>` - Output directory for build artifacts (default: "dist")
- `--sourcemap` - Generate source maps for debugging (automatically enabled in debug mode)
- `--minify` - Enable minification (automatically enabled in release mode)
- `--tree-shake` - Enable dead code elimination (automatically enabled in release mode)
- `--compress` - Enable asset compression (gzip/brotli)
- `--watch` - Watch for changes and rebuild automatically
- `--verbose` - Show detailed build information
- `--clean` - Clean output directory before building

### Build Modes

#### Debug Mode (Default)
```bash
orbiton build
```
- Fast compilation for development
- Includes debug symbols and source maps
- No code minification
- Detailed error messages
- Hot reload support when combined with `--watch`

#### Release Mode
```bash
orbiton build --release
```
- Full optimizations enabled
- Code minification and compression
- Tree shaking to remove unused code
- Asset optimization (image compression, CSS/JS bundling)
- Production-ready performance

### Optimization Levels

#### Size Optimization
```bash
orbiton build --optimize size
```
- Prioritizes smallest bundle size
- Aggressive dead code elimination
- Maximum compression
- Ideal for web deployments with bandwidth constraints

#### Speed Optimization
```bash
orbiton build --optimize speed
```
- Prioritizes runtime performance
- Optimized for fast execution
- May result in larger bundle sizes
- Ideal for desktop applications

#### Balanced Optimization (Default)
```bash
orbiton build --optimize balanced
```
- Balances size and speed
- Suitable for most production deployments
- Good compromise between bundle size and performance

### Build Targets

#### Web Target
```bash
orbiton build --target web
```
**Output Structure:**
```
dist/web/
├── index.html          # Entry point
├── assets/             # Static assets
│   ├── app-[hash].js   # Main application bundle
│   ├── vendor-[hash].js # Third-party dependencies
│   └── style-[hash].css # Compiled styles
├── icons/              # Application icons
└── manifest.json       # Web app manifest
```

**Optimizations:**
- WebAssembly compilation for performance-critical components
- Code splitting for lazy loading
- Service worker generation for offline support
- Progressive Web App (PWA) features

#### Desktop Target
```bash
orbiton build --target desktop
```
**Output Structure:**
```
dist/desktop/
├── orbit-app(.exe)     # Executable binary
├── resources/          # Application resources
├── assets/             # Static assets
└── installer/          # Platform-specific installers
    ├── setup.exe       # Windows installer
    ├── app.dmg         # macOS disk image
    └── app.deb         # Linux package
```

**Optimizations:**
- Native compilation with platform-specific optimizations
- Resource bundling for offline operation
- Icon and metadata embedding
- Code signing preparation

#### Embedded Target
```bash
orbiton build --target embedded
```
**Output Structure:**
```
dist/embedded/
├── firmware.bin        # Compiled firmware
├── resources.pak       # Packed resources
└── metadata.json       # Build metadata
```

**Optimizations:**
- Minimal runtime footprint
- Aggressive dead code elimination
- Optimized memory usage
- Hardware-specific optimizations

### Examples

```bash
# Build for all platforms in release mode
orbiton build --release

# Build optimized for web with compression
orbiton build --target web --optimize size --compress

# Build desktop version with speed optimization
orbiton build --target desktop --optimize speed

# Build with custom output directory
orbiton build --output ./build

# Watch mode for development
orbiton build --watch --sourcemap

# Clean build with verbose output
orbiton build --clean --verbose --release

# Build with specific configuration
orbiton build --release --output ./production
```

### Build Output Analysis

After building, you can analyze the output using:

```bash
# Analyze bundle size
orbiton analyze --format json | grep -E 'bundle|size'

# Check for optimization opportunities
orbiton analyze --rules "perf-*" --format text
```

### Integration with CI/CD

Example GitHub Actions workflow:

```yaml
- name: Build Production
  run: |
    orbiton build --release --target web
    orbiton analyze --format json --output build-analysis.json
```

### Troubleshooting Build Issues

- **Out of Memory**: Use `--target` to build specific platforms
- **Slow Builds**: Try `--optimize speed` or disable `--tree-shake` for development
- **Large Bundles**: Analyze with `orbiton analyze` and enable compression
- **Missing Assets**: Check asset paths in your components and ensure they're included in the build

### Performance Tips

- Use `--watch` during development for faster incremental builds
- Combine `--target web --optimize size` for optimal web deployments
- Enable `--compress` for production deployments to reduce transfer size
- Use `--sourcemap` during development for better debugging experience

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

The Orbiton CLI can be configured using a file named `orbiton.toml` located in the project's root directory. This file allows you to set project-specific defaults for various CLI commands and options, streamlining your development workflow and ensuring consistent behavior across team members and environments.

If an `orbiton.toml` file is not present, the CLI will use its default settings. Command-line arguments will always override any settings defined in `orbiton.toml`.

### Benefits of Configuration

- **Consistency**: Ensure all team members use the same build and development settings
- **Efficiency**: Reduce repetitive command-line arguments
- **Project-Specific Defaults**: Tailor behavior to your project's specific needs
- **Environment Management**: Different configurations for development, staging, and production

### Structure of `orbiton.toml`

The configuration file is organized into sections, each corresponding to an Orbiton CLI command (e.g., `[new]`, `[dev]`, `[build]`). Each section contains key-value pairs that mirror the command-line options for that command.

### Global Configuration Options

```toml
# Global project metadata (optional)
[project]
name = "my-orbit-app"
version = "0.1.0"
description = "A modern Orbit application"
authors = ["Your Name <your.email@example.com>"]

# Workspace settings for monorepos
[workspace]
members = ["packages/*", "apps/*"]
shared_assets = "shared/assets"
```

### Command-Specific Configuration

#### `[new]` - Project Creation Defaults

```toml
[new]
# Default project template (choices: "app", "library", "component")
template = "app"

# Default renderer (choices: "skia", "wgpu", "hybrid", "auto")
renderer = "auto"

# Git repository initialization
skip_git = false
git_template = "standard"  # Custom git template if supported

# Dependency installation
skip_install = false
package_manager = "cargo"  # Preferred package manager

# Default project structure
create_examples = true     # Include example components
create_tests = true        # Include test directory structure
create_docs = false        # Include documentation templates
```

#### `[dev]` - Development Server Configuration

```toml
[dev]
# Server binding options
port = 3000
host = "localhost"        # Use "0.0.0.0" for network access

# Browser behavior
open = true               # Automatically open in browser
browser = "default"       # Specific browser: "chrome", "firefox", "safari"

# Development features
hot_reload = true         # Enable hot module replacement
live_reload = true        # Reload page on non-HMR changes
source_maps = true        # Generate source maps for debugging

# Proxy configuration for API calls
[dev.proxy]
"/api" = "http://localhost:8080"
"/auth" = "https://auth.example.com"

# HTTPS configuration (optional)
[dev.https]
enabled = false
cert_path = "certs/dev-cert.pem"
key_path = "certs/dev-key.pem"

# Asset serving
[dev.assets]
static_dir = "static"
cache_control = "no-cache"  # Development cache behavior
compress = false            # Enable gzip compression
```

#### `[build]` - Production Build Configuration

```toml
[build]
# Build targets (choices: "web", "desktop", "embedded", "all")
target = ["web", "desktop"]

# Optimization settings
release = true
optimize = "balanced"     # Choices: "size", "speed", "balanced"
minify = true
tree_shake = true
compress = true

# Output configuration
output_dir = "dist"
clean_before_build = true
generate_manifest = true

# Asset optimization
[build.assets]
optimize_images = true
image_quality = 85        # JPEG quality (1-100)
generate_webp = true      # Generate WebP variants
inline_small_assets = true # Inline assets under 8KB

# Bundle splitting for web builds
[build.web]
code_splitting = true
chunk_size_limit = 500    # KB
vendor_chunk = true       # Separate vendor dependencies

# Desktop-specific options
[build.desktop]
app_name = "My Orbit App"
app_version = "1.0.0"
icon_path = "assets/icon.png"
```

#### `[component]` - Component Generation Defaults

```toml
[component]
# Default component location
path = "src/components"
nested_directories = true  # Create subdirectories for components

# Template and styling
template = "stateful"      # Choices: "basic", "stateful", "layout", "form", "list", "modal", "container"
style = "scoped"          # Choices: "scoped", "global", "none"

# Generated files
test = true               # Generate test files
story = true              # Generate story files for documentation
types = true              # Generate TypeScript definitions

# Naming conventions
case_style = "PascalCase"  # Choices: "PascalCase", "camelCase", "kebab-case"
file_extension = ".orbit"

# Default props for new components
[component.default_props]
className = "string?"      # Optional className prop
children = "node?"         # Optional children prop
```

#### `[analyze]` - Code Analysis Configuration

```toml
[analyze]
# Analysis behavior
fix = false               # Automatically fix issues when possible
verbose = false           # Show detailed analysis information
format = "text"           # Choices: "text", "json", "html"
output = "analysis.json"  # Output file for JSON/HTML formats

# File inclusion/exclusion
include = ["src/**/*.orbit", "components/**/*.orbit"]
exclude = ["**/*.test.orbit", "**/node_modules/**", "dist/**"]

# Rule configuration
config = ".orlint.toml"   # Path to Orlint configuration
rules = ["accessibility", "performance", "best-practices"]
ignore_rules = ["experimental-features"]

# Performance settings
parallel = true           # Run analysis in parallel
incremental = true        # Only analyze changed files
cache_results = true      # Cache analysis results
```

#### `[renderer]` - Rendering Backend Configuration

```toml
[renderer]
# Default renderer (choices: "skia", "wgpu", "auto")
default = "auto"

# Renderer-specific options
[renderer.skia]
antialiasing = true
text_rendering = "subpixel"  # Choices: "pixel", "subpixel", "lcd"
gpu_acceleration = true

[renderer.wgpu]
power_preference = "high-performance"  # Choices: "low-power", "high-performance"
backend = "vulkan"         # Choices: "vulkan", "metal", "dx12", "gl"
vsync = true
multisample = 4            # MSAA sample count

# Fallback configuration
[renderer.fallback]
enable_fallback = true
fallback_order = ["wgpu", "skia"]
```

#### `[test]` - Testing Configuration

```toml
[test]
# Test execution
watch = false             # Run in watch mode
parallel = true           # Run tests in parallel
coverage = false          # Generate coverage reports

# Test types
unit = true               # Run unit tests
integration = true        # Run integration tests
performance = false       # Run performance benchmarks

# Coverage settings
[test.coverage]
threshold = 80            # Minimum coverage percentage
output_dir = "coverage"
format = ["html", "lcov"] # Coverage report formats

# Performance benchmarks
[test.performance]
render_threshold_ms = 16  # Maximum render time
memory_threshold_mb = 50  # Maximum memory usage
```

### Environment-Specific Configuration

You can create environment-specific configuration files:

- `orbiton.dev.toml` - Development overrides
- `orbiton.prod.toml` - Production overrides
- `orbiton.test.toml` - Testing overrides

Use the `ORBITON_ENV` environment variable to specify which configuration to use:

```bash
ORBITON_ENV=dev orbiton dev    # Uses orbiton.dev.toml
ORBITON_ENV=prod orbiton build # Uses orbiton.prod.toml
```

### Configuration Validation and Defaults

The Orbiton CLI validates all configuration options and provides helpful error messages for invalid values. If a required option is missing, sensible defaults are used.

#### Example Validation Messages

```bash
❌ Error in orbiton.toml: Invalid renderer 'invalid-renderer'
   Valid options: skia, wgpu, auto

⚠️  Warning: Port 3000 is already in use, falling back to 3001

✅ Configuration loaded successfully from orbiton.toml
```

### Configuration Inheritance

Configuration follows this precedence order (highest to lowest):

1. **Command-line arguments** - Always take precedence
2. **Environment variables** - Override configuration files
3. **Environment-specific config** - `orbiton.<env>.toml`
4. **Project config** - `orbiton.toml`
5. **Global user config** - `~/.orbiton/config.toml`
6. **Built-in defaults** - Framework defaults

### Common Configuration Patterns

#### Development Team Setup

```toml
# Optimized for team development
[dev]
port = 3000
host = "0.0.0.0"          # Allow network access for device testing
open = false              # Don't auto-open browser (varies by developer preference)
hot_reload = true

[build]
target = ["web"]          # Focus on web deployment
optimize = "speed"        # Faster builds during development

[component]
test = true               # Always generate tests
story = true              # Generate stories for design system
template = "stateful"     # Most components need state

[analyze]
fix = true                # Auto-fix when possible
verbose = true            # Detailed output for CI/CD
format = "json"           # Machine-readable for CI reporting
```

#### Production-Focused Setup

```toml
# Optimized for production deployment
[build]
target = ["web", "desktop"]
release = true
optimize = "size"         # Minimize bundle size
minify = true
tree_shake = true
compress = true

[analyze]
fix = false               # Don't auto-modify in CI
format = "json"
output = "build-analysis.json"
rules = ["security", "performance", "accessibility"]

[test]
coverage = true
threshold = 90            # High coverage requirement
performance = true        # Include performance benchmarks
```

#### Component Library Setup

```toml
# Optimized for component library development
[new]
template = "library"

[component]
path = "src/components"
template = "basic"        # Start simple, add complexity as needed
test = true
story = true              # Essential for component documentation
types = true              # Generate TypeScript definitions

[build]
target = ["web"]
optimize = "balanced"
generate_manifest = true  # For package distribution

[dev]
port = 6006               # Storybook-style port
open = true
```

### Configuration Best Practices

#### 1. Environment Separation

```bash
# Development
# orbiton.dev.toml
[dev]
port = 3000
hot_reload = true
source_maps = true

[build]
optimize = "speed"
minify = false

# Production
# orbiton.prod.toml
[build]
optimize = "size"
minify = true
compress = true
tree_shake = true
```

#### 2. Team Consistency

```toml
# Enforce consistent formatting and analysis
[analyze]
fix = true
format = "text"
rules = [
  "accessibility",
  "performance", 
  "best-practices",
  "security"
]

[component]
template = "stateful"
test = true
style = "scoped"
```

#### 3. Monorepo Configuration

```toml
[workspace]
members = ["packages/*", "apps/*"]
shared_assets = "shared/assets"

# Shared development configuration
[dev]
proxy."/api" = "http://localhost:8080"
proxy."/shared" = "http://localhost:3001"

# Component generation for shared libraries
[component]
path = "packages/ui/src/components"
```

### Troubleshooting Configuration

#### Common Issues

**Configuration Not Loading**
```bash
# Verify configuration file exists and is valid TOML
orbiton config validate

# Check which configuration file is being used
orbiton config show
```

**Port Conflicts**
```bash
# Override port in configuration or command line
orbiton dev --port 3001

# Or set in orbiton.toml:
[dev]
port = 3001
```

**Invalid Configuration Values**
```
❌ Error: Invalid optimization level 'ultra'
   Valid options: size, speed, balanced

   Fix: Update orbiton.toml
   [build]
   optimize = "balanced"  # ✅ Valid option
```

### Advanced Configuration Features

#### Environment Variable Substitution

```toml
[dev]
port = "${PORT:-3000}"           # Use PORT env var, default to 3000
host = "${HOST:-localhost}"      # Use HOST env var, default to localhost

[build]
output_dir = "${BUILD_DIR:-dist}" # Configurable output directory
```

#### Conditional Configuration

```toml
# Different settings based on platform
[build.web]
optimize = "size"
compress = true

[build.desktop]
optimize = "speed"
compress = false

# Different settings based on environment
[dev]
port = 3000

[dev.ci]
port = 8080              # Different port in CI
open = false             # Don't open browser in CI
```

#### Configuration Includes

```toml
# Include shared configuration
include = ["shared/orbiton-base.toml"]

# Override specific settings
[build]
target = ["web"]         # Override shared target setting
```

### Migration and Versioning

#### Configuration Version

```toml
# Specify configuration format version
config_version = "1.0"

# Enable forward compatibility warnings
compatibility_mode = "strict"
```

#### Migration Helpers

```bash
# Migrate from older configuration format
orbiton config migrate --from-version 0.9 --to-version 1.0

# Validate configuration against current version
orbiton config validate --strict

# Show configuration differences
orbiton config diff --compare-version 0.9
```

### Validation

The Orbiton CLI will parse `orbiton.toml` when a command is run. If the file contains invalid TOML syntax or unrecognized options, the CLI may issue a warning or error and fall back to default behavior or halt execution, depending on the severity.

It's recommended to refer to the latest Orbiton CLI documentation or use `orbiton help <command>` for the most up-to-date list of configurable options and their valid values.

### Configuration Schema Reference

For comprehensive validation and IDE support, you can reference the configuration schema:

```bash
# Generate JSON schema for IDE validation
orbiton config schema > orbiton-schema.json

# Validate configuration against schema
orbiton config validate --schema orbiton-schema.json
```

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
