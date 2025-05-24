# `orbiton new` Project Templates

This document provides visual examples of projects generated with the `orbiton new` command, showing the directory structure and key files for each template type.

## Basic Application Template

The basic application template creates a minimal but complete Orbit application structure. Here's what a freshly generated project looks like:

### Directory Structure

```
my-orbit-app/
├── Cargo.toml
├── README.md
├── index.html
└── src/
    ├── main.rs
    └── components/
        └── app.orbit
```

### Key Files Overview

#### `index.html`

The main HTML entry point that loads your Orbit application and establishes WebSocket connections for hot module replacement:

```html
<!DOCTYPE html>
<html>
<head>
    <title>my-orbit-app - Dev Server</title>
    <script>
        // WebSocket setup for hot module replacement
        const port = window.location.port;
        const wsPort = parseInt(port) + 1;
        const ws = new WebSocket(`ws://localhost:${wsPort}`);
        
        // Connection handling and HMR logic
        // ...
    </script>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```

#### `src/components/app.orbit`

The main application component, showing the Orbit single-file component format:

```orbit
<template>
  <div class="app">
    <h1>Welcome to my-orbit-app</h1>
    <p>Created with Orbit UI Framework</p>
  </div>
</template>

<style>
.app {
  text-align: center;
  padding: 2rem;
}

h1 {
  color: #2c3e50;
}
</style>

<code lang="rust">
use orbit::prelude::*;

pub struct App;

impl Component for App {
    type Props = ();

    fn new(_: Self::Props) -> Self {
        Self
    }
}
</code>
```

#### `src/main.rs`

The application entry point:

```rust
use orbit::prelude::*;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("Hello from my-orbit-app!");
    Ok(())
}
```

### Application Running in Browser

When running with `orbiton dev`, the application displays in the browser with a centered welcome message:

![Basic Application Screenshot](../images/basic-app-screenshot.png)

*Note: The welcome page displays the project name and a simple centered layout.*

## Component Library Template

The component library template sets up a project structure focused on creating reusable components.

### Directory Structure

```
my-components/
├── Cargo.toml
├── README.md
└── src/
    ├── lib.rs
    ├── components/
    │   ├── mod.rs
    │   ├── button.orbit
    │   └── card.orbit
    └── examples/
        └── showcase.orbit
```

### Key Files Overview

#### `src/lib.rs`

Exports the component library:

```rust
//! Component library for my-components
pub mod components;

// Re-export commonly used components
pub use components::button::Button;
pub use components::card::Card;
```

#### `src/components/button.orbit`

Example of a component in the library:

```orbit
<template>
  <button 
    class="orbit-button {{ variant }} {{ size }} {{ disabled ? 'disabled' : '' }}" 
    :disabled="disabled"
    @click="handleClick"
  >
    <slot></slot>
  </button>
</template>

<style>
.orbit-button {
  border-radius: 4px;
  cursor: pointer;
  font-family: sans-serif;
  transition: background-color 0.2s;
}

.primary {
  background-color: #3498db;
  color: white;
  border: none;
}

/* Additional styles... */
</style>

<code lang="rust">
use orbit::prelude::*;

pub struct Button {
    variant: String,
    size: String,
    disabled: bool,
    on_click: Option<EventHandler<ClickEvent>>,
}

impl Component for Button {
    type Props = ButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            variant: props.variant.unwrap_or_else(|| "primary".to_string()),
            size: props.size.unwrap_or_else(|| "medium".to_string()),
            disabled: props.disabled.unwrap_or(false),
            on_click: props.on_click,
        }
    }
    
    fn handle_click(&mut self, event: ClickEvent) {
        if !self.disabled {
            if let Some(handler) = &self.on_click {
                handler.emit(event);
            }
        }
    }
}

#[derive(Clone)]
pub struct ButtonProps {
    pub variant: Option<String>,
    pub size: Option<String>,
    pub disabled: Option<bool>,
    pub on_click: Option<EventHandler<ClickEvent>>,
}
</code>
```

### Component Library in Use

The component library can be used in other projects or demonstrated in the showcase:

![Component Library Screenshot](../images/component-library-screenshot.png)

*Note: The showcase demonstrates various components with different props and states.*

## Cross-Platform Application

When using the `--renderer` option, projects are set up with different rendering backends.

### Skia Renderer

The Skia renderer targets desktop applications with a consistent rendering experience across platforms:

![Skia Renderer Application](../images/skia-renderer-screenshot.png)

*Note: The Skia renderer provides hardware-accelerated 2D graphics for desktop applications.*

### WGPU Renderer

The WGPU renderer targets high-performance graphics applications and games:

![WGPU Renderer Application](../images/wgpu-renderer-screenshot.png)

*Note: The WGPU renderer provides 3D capability and modern GPU features.*

## Using Custom Templates

You can also create projects with custom templates:

```bash
orbiton new my-project --template path/to/custom-template
```

## Next Steps After Generation

After generating a project:

1. Navigate to the project directory: `cd my-project-name`
2. Run the development server: `orbiton dev`
3. Start editing component files in the `src/components` directory
4. Build for production with `orbiton build` when ready

For more information on working with generated projects, see the [Getting Started guide](../getting-started/getting-started.md).
