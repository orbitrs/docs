# Monorepo Setup with Orbiton

This guide shows how to set up and manage a monorepo workspace containing multiple Orbit applications, shared libraries, and component packages using Orbiton CLI.

## Overview

A monorepo structure allows you to:
- Share code between multiple applications
- Maintain consistent tooling and dependencies
- Simplify cross-project collaboration
- Implement centralized testing and CI/CD

## Workspace Structure

Here's a typical Orbit monorepo structure:

```
my-orbit-workspace/
├── Cargo.toml                 # Workspace configuration
├── orbiton.toml              # Global Orbiton configuration
├── .gitignore
├── README.md
├── apps/                     # Applications
│   ├── web-app/
│   │   ├── Cargo.toml
│   │   ├── orbiton.toml
│   │   └── src/
│   ├── desktop-app/
│   │   ├── Cargo.toml
│   │   ├── orbiton.toml
│   │   └── src/
│   └── mobile-app/
│       ├── Cargo.toml
│       ├── orbiton.toml
│       └── src/
├── packages/                 # Shared packages
│   ├── shared-components/
│   │   ├── Cargo.toml
│   │   └── src/
│   ├── shared-utils/
│   │   ├── Cargo.toml
│   │   └── src/
│   └── design-system/
│       ├── Cargo.toml
│       └── src/
├── tools/                    # Development tools
│   ├── build-scripts/
│   └── dev-server/
└── docs/                     # Documentation
    ├── apps/
    ├── packages/
    └── guides/
```

## Setting Up the Workspace

### 1. Initialize the Workspace

Create the root workspace configuration:

**Cargo.toml**
```toml
[workspace]
members = [
    "apps/*",
    "packages/*",
    "tools/*"
]

[workspace.dependencies]
# Shared dependencies
orbit = { version = "0.1.0", path = "../../orbit" }
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }

[workspace.metadata.orbiton]
version = "0.1.0"
```

**orbiton.toml**
```toml
[workspace]
name = "my-orbit-workspace"
version = "0.1.0"

# Global configuration
[dev]
port = 3000
host = "localhost"
open = false

[build]
target = "all"
optimize = "balanced"

# Shared configuration for all apps
[apps]
shared_config = true

# Package-specific settings
[packages]
components_dir = "packages/shared-components/src"
utils_dir = "packages/shared-utils/src"

[analyzer]
rules = [
    "orbit/recommended",
    "accessibility/strict",
    "performance/recommended"
]
```

### 2. Create Applications

Generate applications within the `apps/` directory:

```bash
# Create web application
cd apps
orbiton new web-app --template web --workspace

# Create desktop application  
orbiton new desktop-app --template desktop --workspace

# Create mobile application
orbiton new mobile-app --template mobile --workspace
```

Each app gets its own configuration:

**apps/web-app/orbiton.toml**
```toml
[app]
name = "web-app"
type = "web"

[dev]
port = 3001
proxy = [
    { path = "/api", target = "http://localhost:8080" }
]

[build]
target = "web"
optimize = "size"
output = "../../dist/web"

[dependencies]
packages = [
    "shared-components",
    "shared-utils", 
    "design-system"
]
```

**apps/desktop-app/orbiton.toml**
```toml
[app]
name = "desktop-app"
type = "desktop"

[dev]
hot_reload = true

[build]
target = "desktop"
optimize = "speed"
output = "../../dist/desktop"

[dependencies]
packages = [
    "shared-components",
    "shared-utils",
    "design-system"
]
```

### 3. Create Shared Packages

Generate shared packages:

```bash
# Create shared component library
cd packages
orbiton component-library shared-components --workspace

# Create shared utilities
orbiton package shared-utils --workspace

# Create design system
orbiton design-system design-system --workspace
```

**packages/shared-components/Cargo.toml**
```toml
[package]
name = "shared-components"
version.workspace = true
edition = "2021"

[dependencies]
orbit.workspace = true
serde.workspace = true

[lib]
crate-type = ["cdylib", "rlib"]
```

## Development Workflow

### Running Applications

Use workspace-aware commands to run applications:

```bash
# Run specific app
orbiton dev --app web-app

# Run all apps (different ports)
orbiton dev --all

# Run with specific configuration
orbiton dev --app web-app --config development

# Run with package watching
orbiton dev --app web-app --watch-packages
```

### Building Applications

Build individual or multiple applications:

```bash
# Build specific app
orbiton build --app web-app --release

# Build all apps
orbiton build --all

# Build with shared package optimization
orbiton build --app web-app --optimize-shared

# Build for specific environment
orbiton build --app web-app --env production
```

### Package Development

Work with shared packages:

```bash
# Generate component in shared library
orbiton component Button --package shared-components

# Test shared package
orbiton test --package shared-components

# Build package documentation
orbiton docs --package shared-components

# Analyze package
orbiton analyze --package shared-components
```

## Package Management

### Inter-Package Dependencies

Define dependencies between packages:

**packages/shared-components/Cargo.toml**
```toml
[dependencies]
shared-utils = { path = "../shared-utils" }
design-system = { path = "../design-system" }
```

**packages/design-system/orbiton.toml**
```toml
[package]
name = "design-system"
type = "library"

[exports]
tokens = "src/tokens.rs"
components = "src/components/mod.rs"
themes = "src/themes/mod.rs"

[build]
generate_types = true
generate_docs = true
```

### Version Management

Use workspace versioning for consistency:

```bash
# Update version across workspace
orbiton version patch --workspace

# Update specific package version
orbiton version minor --package shared-components

# Check version consistency
orbiton analyze --check-versions
```

## Shared Configuration

### Global Orbiton Configuration

**orbiton.toml** (workspace root)
```toml
[workspace]
name = "my-orbit-workspace"

# Shared development server configuration
[dev]
shared_port_range = "3000-3099"
proxy_config = "dev-proxy.toml"

# Shared build configuration
[build]
shared_assets = "assets/"
shared_output = "dist/"

# Shared analysis rules
[analyzer]
rules_config = "analysis-rules.toml"
ignore_patterns = [
    "dist/",
    "target/",
    "node_modules/"
]

# Package discovery
[packages]
auto_discover = true
include_patterns = ["packages/*/", "libs/*/"]
exclude_patterns = ["packages/private-*/"]
```

### Environment-Specific Configuration

**configs/development.toml**
```toml
[dev]
debug = true
hot_reload = true
source_maps = true

[build]
minify = false
optimize = "speed"

[analyzer]
strict_mode = false
```

**configs/production.toml**
```toml
[dev]
debug = false

[build]
minify = true
optimize = "size"
tree_shake = true

[analyzer]
strict_mode = true
```

## Cross-Package Development

### Shared Component Development

Create components that work across applications:

**packages/shared-components/src/button.orbit**
```orbit
<template>
  <button 
    class="btn"
    :class="[variant, size]"
    :disabled="disabled"
    @click="handleClick"
  >
    <slot name="icon" v-if="hasIcon" />
    <span class="btn-text">
      <slot />
    </span>
  </button>
</template>

<script>
use orbit::prelude::*;
use design_system::tokens::*;

#[derive(Props)]
pub struct ButtonProps {
    #[prop(default = "ButtonVariant::Primary")]
    pub variant: ButtonVariant,
    
    #[prop(default = "ButtonSize::Medium")]
    pub size: ButtonSize,
    
    #[prop(default = false)]
    pub disabled: bool,
    
    pub on_click: Option<EventHandler<ClickEvent>>,
}

#[component]
pub fn Button(props: ButtonProps) -> impl View {
    let handle_click = use_callback(move |event: ClickEvent| {
        if !props.disabled {
            if let Some(handler) = &props.on_click {
                handler.call(event);
            }
        }
    });
    
    // Component implementation
}
</script>

<style>
.btn {
  /* Styles using design system tokens */
  background: var(--color-primary);
  border: 1px solid var(--color-primary-dark);
  border-radius: var(--radius-md);
  padding: var(--spacing-sm) var(--spacing-md);
}
</style>
```

### Using Shared Components

In applications:

**apps/web-app/src/components/LoginForm.orbit**
```orbit
<template>
  <form @submit="handleSubmit">
    <SharedInput v-model="username" placeholder="Username" />
    <SharedInput v-model="password" type="password" placeholder="Password" />
    <SharedButton type="submit" variant="primary">
      Login
    </SharedButton>
  </form>
</template>

<script>
use orbit::prelude::*;
use shared_components::{Button as SharedButton, Input as SharedInput};

// Component implementation
</script>
```

## Testing Strategy

### Workspace Testing

Run tests across the entire workspace:

```bash
# Test all packages and apps
orbiton test --workspace

# Test specific package
orbiton test --package shared-components

# Test app with dependencies
orbiton test --app web-app --include-deps

# Integration tests
orbiton test --integration --workspace
```

### Test Configuration

**tests/workspace.toml**
```toml
[test]
parallel = true
timeout = "30s"

# Package-specific test configuration
[test.packages.shared-components]
features = ["test-utils"]
extra_args = ["--lib"]

[test.apps.web-app]
features = ["testing"]
browser = "chrome"
```

## CI/CD Integration

### GitHub Actions Workflow

**.github/workflows/workspace.yml**
```yaml
name: Workspace CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Orbiton
        run: cargo install orbiton-cli
        
      - name: Test workspace
        run: orbiton test --workspace
        
      - name: Analyze workspace
        run: orbiton analyze --workspace --format json --output analysis.json
        
      - name: Upload analysis
        uses: actions/upload-artifact@v3
        with:
          name: analysis-report
          path: analysis.json

  build:
    needs: test
    strategy:
      matrix:
        app: [web-app, desktop-app]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build app
        run: orbiton build --app ${{ matrix.app }} --release
        
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: ${{ matrix.app }}-build
          path: dist/${{ matrix.app }}/
```

## Advanced Features

### Package Linking

Link packages for development:

```bash
# Link package for development
orbiton link shared-components --to apps/web-app

# Unlink package
orbiton unlink shared-components --from apps/web-app

# Link all packages
orbiton link --all --to apps/web-app
```

### Workspace Scripts

Define common scripts in workspace configuration:

**orbiton.toml**
```toml
[scripts]
start-all = "orbiton dev --all"
build-all = "orbiton build --all --release"
test-all = "orbiton test --workspace"
lint-all = "orbiton analyze --workspace --fix"
clean-all = "cargo clean && rm -rf dist/"

# App-specific scripts
start-web = "orbiton dev --app web-app"
build-web = "orbiton build --app web-app --release"
deploy-web = "orbiton deploy --app web-app --env production"
```

Run scripts:

```bash
orbiton run start-all
orbiton run build-web
orbiton run deploy-web
```

### Dependency Management

Manage shared dependencies:

```bash
# Add dependency to workspace
orbiton add serde --workspace

# Add dependency to specific package
orbiton add reqwest --package shared-utils

# Update dependencies
orbiton update --workspace

# Check for security vulnerabilities
orbiton audit --workspace
```

## Best Practices

### Organization

1. **Clear Separation**: Keep apps and packages clearly separated
2. **Consistent Naming**: Use consistent naming conventions across packages
3. **Documentation**: Document inter-package dependencies and APIs
4. **Version Management**: Use workspace versioning for related packages

### Development

1. **Hot Reload**: Enable hot reload for efficient development
2. **Incremental Builds**: Use incremental builds for faster iteration
3. **Package Watching**: Watch shared packages during app development
4. **Type Safety**: Maintain strong typing across package boundaries

### Deployment

1. **Independent Deployment**: Allow apps to be deployed independently
2. **Shared Assets**: Optimize shared assets and dependencies
3. **Environment Parity**: Maintain consistent environments across apps
4. **Rollback Strategy**: Plan for independent app rollbacks

## Troubleshooting

### Common Issues

**Package Not Found**
```bash
# Refresh package discovery
orbiton refresh --packages

# Check package paths
orbiton info --package shared-components
```

**Version Conflicts**
```bash
# Check version consistency
orbiton analyze --check-versions

# Resolve conflicts
orbiton resolve --versions
```

**Build Issues**
```bash
# Clean workspace
orbiton clean --workspace

# Rebuild packages
orbiton build --packages --clean
```

### Debugging

Enable debug mode for detailed logging:

```bash
ORBITON_DEBUG=1 orbiton dev --app web-app
```

## Migration Guide

### From Single App to Monorepo

1. **Create Workspace Structure**:
   ```bash
   mkdir my-workspace
   cd my-workspace
   orbiton workspace init
   ```

2. **Move Existing App**:
   ```bash
   mv ../my-app apps/
   cd apps/my-app
   orbiton convert --to-workspace
   ```

3. **Extract Shared Code**:
   ```bash
   orbiton extract --components --to packages/shared-components
   orbiton extract --utils --to packages/shared-utils
   ```

4. **Update Dependencies**:
   ```bash
   orbiton update-deps --workspace
   ```

## See Also

- [Orbiton CLI Reference](../api/orbiton-cli.md) - Complete CLI documentation
- [Project Structure](../guides/project-structure.md) - Organizing Orbit projects
- [Deployment Guide](../guides/deployment-guide.md) - Deploying monorepo applications
- [Testing Strategies](../guides/testing-strategies.md) - Testing in monorepos
