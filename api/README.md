# 📚 API Reference

This section provides detailed API documentation for the Orbit Framework and its related tools.

## 📋 Table of Contents

- [orbit](#orbit-core-framework)
- [orbitkit](#orbitkit-component-library)
- [orbiton](#orbiton-cli-tools)
- [orbit-analyzer](#orbit-analyzer-static-analysis)

---

## orbit (Core Framework)

### Component API

The core component API that powers the Orbit framework.

```rust
// Component structure
<Component>
  <Props>
    // Component properties definition
  </Props>

  <State>
    // Internal state definition
  </State>

  <Style>
    // Component-scoped styles
  </Style>

  <Template>
    // UI template
  </Template>

  <Script>
    // Component logic
  </Script>
</Component>
```

#### Props

Props define the public API of your component:

```rust
<Props>
  name: String
  count: i32 = 0  // Default value
  on_change: Option<Callback<i32>> = None  // Optional callback
</Props>
```

#### State

State defines the internal data of your component:

```rust
<State>
  internal_count: i32 = self.props.count
  is_active: bool = false
</State>
```

#### Methods

Common component methods:

| Method | Description |
|--------|-------------|
| `mount()` | Called when component is added to the DOM |
| `update()` | Called when props or state changes |
| `unmount()` | Called when component is removed from the DOM |
| `should_update(next_props: &Props, next_state: &State) -> bool` | Controls if component should re-render |

### Rendering API

Core rendering APIs for Orbit.

#### Renderer

```rust
pub struct Renderer {
    // ...
}

impl Renderer {
    pub fn new(config: RendererConfig) -> Self;
    pub fn render(&mut self, component: Component);
    pub fn update(&mut self);
    pub fn resize(&mut self, width: u32, height: u32);
    // ...
}
```

#### Virtual DOM

```rust
pub struct VNode {
    // ...
}

impl VNode {
    pub fn element(tag: &str, props: Props, children: Vec<VNode>) -> Self;
    pub fn text(content: &str) -> Self;
    pub fn component(component: Box<dyn Component>, props: Props) -> Self;
    // ...
}
```

### State Management API

APIs for managing component and application state.

#### Store

```rust
pub struct Store<T> {
    // ...
}

impl<T: Clone + 'static> Store<T> {
    pub fn new(initial_state: T) -> Self;
    pub fn get(&self) -> T;
    pub fn update<F>(&self, updater: F) where F: FnOnce(&mut T);
    pub fn subscribe<F>(&self, subscriber: F) where F: Fn(&T) + 'static;
    // ...
}
```

---

## orbitkit (Component Library)

### Layout Components

Building blocks for UI structure.

#### Box

```rust
<Box
  direction="row|column"
  align="start|center|end|stretch"
  justify="start|center|end|space-between|space-around"
  wrap="wrap|nowrap"
  gap={8}
>
  {children}
</Box>
```

#### Grid

```rust
<Grid
  columns={3}
  gap={16}
  template="1fr 2fr 1fr"
>
  {children}
</Grid>
```

### UI Components

Common user interface elements.

#### Button

```rust
<Button
  variant="primary|secondary|text"
  size="small|medium|large"
  disabled={false}
  on:click={self.handle_click}
>
  Click Me
</Button>
```

#### Input

```rust
<Input
  type="text|email|password|number"
  value={self.state.value}
  placeholder="Enter text..."
  on:change={self.handle_change}
  disabled={false}
/>
```

---

## orbiton (CLI Tools)

### Command Line Interface

```bash
# Create a new project
orbiton new [project-name] [--template basic|advanced]

# Start development server
orbiton dev [--port 3000] [--open]

# Build for production
orbiton build [--target web|desktop|mobile] [--optimize]

# Generate component
orbiton generate component [name]

# Run tests
orbiton test [--watch]
```

### Configuration

Example `orbiton.config.js`:

```js
module.exports = {
  // Project configuration
  project: {
    name: 'my-orbit-app',
    version: '0.1.0',
  },
  
  // Build configuration
  build: {
    target: ['web', 'desktop'],
    outDir: 'dist',
    optimizations: {
      minify: true,
      treeShaking: true,
    },
  },
  
  // Development server
  devServer: {
    port: 3000,
    open: true,
    proxy: {
      '/api': 'http://localhost:8080',
    },
  },
  
  // Plugins
  plugins: [
    'orbit-plugin-pwa',
  ],
};
```

---

## orbit-analyzer (Static Analysis)

### Lint Rules

```rust
// Available lint categories
pub enum LintCategory {
    Performance,
    Accessibility,
    BestPractices,
    Style,
    Error,
}

// Example lint rule
pub struct UnusedProps {
    // ...
}

impl LintRule for UnusedProps {
    fn check(&self, component: &Component) -> Vec<Diagnostic>;
    fn fix(&self, component: &mut Component) -> Result<(), FixError>;
}
```

### Command Line Interface

```bash
# Analyze a component or directory
orbit-analyzer [path] [--fix] [--config <config-file>]

# Generate a report
orbit-analyzer --report [html|json|text] --output report.html
```

---

> Note: This API documentation is a preview and subject to change as development progresses. Refer to the in-code documentation for the most up-to-date information.
