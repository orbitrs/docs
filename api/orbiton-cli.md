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

```bash
# Start the development server
orbiton dev

# Start on a custom port and host
orbiton dev --port 8080 --host 0.0.0.0

# Start with a specific renderer
orbiton dev --renderer wgpu

# Start and automatically open in the browser
orbiton dev --open
```

### Development Server Advanced Features

The Orbit development server provides a rich set of features to enhance your development experience:

#### Hot Module Replacement (HMR)

The development server includes built-in Hot Module Replacement that automatically updates your application when files change, without requiring a full page reload.

```bash
# Enable detailed HMR logging
orbiton dev --hmr-verbose
```

#### Proxy Configuration

You can configure proxy settings to forward API requests to backend servers:

```bash
# With command line options
orbiton dev --proxy-target http://localhost:3000 --proxy-path /api

# Using orbit.config.json
```

```json
{
  "dev": {
    "proxy": [
      {
        "path": "/api",
        "target": "http://localhost:3000"
      },
      {
        "path": "/auth",
        "target": "http://auth-service:8080"
      }
    ]
  }
}
```

#### HTTPS Support

For secure development, the dev server supports HTTPS:

```bash
# Enable HTTPS with auto-generated certificates
orbiton dev --https

# Use custom certificates
orbiton dev --https --cert-file ./certs/cert.pem --key-file ./certs/key.pem
```

#### WebSocket Server

The development server includes a WebSocket server that enables real-time communication:

- Automatically runs on port+1 (e.g., if HTTP is on 8000, WebSocket is on 8001)
- Used for HMR and live-reload features
- Can be used by your application for development-time features

```js
// Connect to the development WebSocket server from your application
const socket = new WebSocket(`ws://localhost:8001`);

socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received message:', data);
};
```

#### Static File Serving

The dev server automatically serves static files from your project directory:

- Public assets from `/public` directory
- Built files from `/dist` directory
- The main `index.html` file

#### Custom Middleware

Advanced users can add custom middleware to the development server by creating a `dev-server.js` file in the project root:

```js
// dev-server.js
module.exports = {
  before(app, server) {
    // Add custom middleware
    app.get('/custom', (req, res) => {
      res.send('Custom middleware response');
    });
  },
  after(app, server) {
    // Add middleware after built-in middleware
  }
};
```

#### Environment Variables

The development server automatically loads environment variables from `.env` files:

- `.env` - Default environment variables for all environments
- `.env.development` - Development-specific variables (takes precedence)
- `.env.local` - Local overrides (highest precedence, not committed to version control)

#### Performance Monitoring

Monitor development server performance:

```bash
# Enable performance monitoring
orbiton dev --perf-monitor

# Access the performance dashboard at http://localhost:8000/__performance
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
