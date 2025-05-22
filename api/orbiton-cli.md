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

Options:
- `--port <port>` - Port to run the development server on (default: 3000)
- `--host <host>` - Host address to bind to (default: "localhost")
- `--no-open` - Don't automatically open in browser
- `--renderer <renderer>` - Override the renderer for development

Examples:

```bash
# Start dev server with default settings
orbiton dev

# Start on a custom port and host
orbiton dev --port 8080 --host 0.0.0.0

# Start with a specific renderer
orbiton dev --renderer wgpu
```

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

Options:
- `--path <path>` - Custom path for the component (default: "src/components")
- `--template <template>` - Component template to use (default: "basic")

Examples:

```bash
# Generate a basic component
orbiton component button

# Generate a component with a custom template in a specific path
orbiton component user-profile --template form --path src/features/user
```

## Project Analysis

```bash
orbiton analyze [options]
```

Options:
- `--fix` - Automatically fix issues when possible
- `--verbose` - Show detailed analysis information

Examples:

```bash
# Run analysis
orbiton analyze

# Run analysis and fix issues
orbiton analyze --fix
```

## Renderer Management

```bash
orbiton renderer [command] [options]
```

Commands:
- `list` - List available renderers
- `set <renderer>` - Set the default renderer
- `info <renderer>` - Show information about a renderer

Examples:

```bash
# List all available renderers
orbiton renderer list

# Set the default renderer
orbiton renderer set wgpu

# Get information about a renderer
orbiton renderer info skia
```

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
    - uses: actions/checkout@v3
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
