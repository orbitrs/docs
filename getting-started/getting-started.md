# Getting Started with Orbit UI Framework

This guide will help you set up your development environment and create your first Orbit application.

## Prerequisites

- Rust toolchain (1.70.0 or newer) - [rustup.rs](https://rustup.rs/)
- Cargo package manager
- Git

## Installation

First, install the Orbit CLI tool:

```bash
cargo install orbiton
```

This will install the `orbiton` command-line tool, which you'll use to create and manage Orbit projects.

## Creating Your First Project

Create a new Orbit project:

```bash
orbiton new my-first-app
cd my-first-app
```

This creates a new directory with a basic Orbit application structure:

```
my-first-app/
├── Cargo.toml
├── src/
│   ├── main.rs
│   └── components/
│       ├── app.orbit
│       └── counter.orbit
├── static/
│   ├── index.html
│   └── styles.css
└── .gitignore
```

## Running the Application

Start the development server:

```bash
orbiton dev
```

This will compile your application and start a development server at `http://localhost:3000`. The server includes hot reloading, so changes to your code will be reflected immediately in the browser.

## Creating Components

Orbit uses a single-file component format with `.orbit` files. Each file contains three sections:

- `<template>` - HTML-like markup for the component
- `<style>` - CSS styles for the component
- `<script>` - Rust code for the component logic

Let's create a simple "Hello, World" component:

```bash
# Create a new component
orbiton component hello-world
```

This creates a new file `src/components/hello_world.orbit`:

```
<template>
  <div class="hello-world">
    <h1>{{ greeting }}</h1>
    <p>Welcome to Orbit UI!</p>
  </div>
</template>

<style>
.hello-world {
  text-align: center;
  padding: 20px;
}

h1 {
  color: #333;
  font-size: 24px;
}

p {
  color: #666;
}
</style>

<script>
use orbit::prelude::*;

pub struct HelloWorld {
    greeting: String,
}

pub struct HelloWorldProps {
    pub greeting: String,
}

impl Props for HelloWorldProps {}

impl Component for HelloWorld {
    type Props = HelloWorldProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            greeting: props.greeting,
        }
    }
    
    // render() method is auto-generated from the template
}
</script>
```

## Using the Component

Now let's use our new component in the application. Open `src/components/app.orbit` and replace its contents:

```
<template>
  <div class="app">
    <HelloWorld greeting="Hello, Orbit!" />
  </div>
</template>

<style>
.app {
  max-width: 800px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}
</style>

<script>
use orbit::prelude::*;
use crate::components::hello_world::HelloWorld;

pub struct App {}

pub struct AppProps {}

impl Props for AppProps {}

impl Component for App {
    type Props = AppProps;
    
    fn new(_props: Self::Props) -> Self {
        Self {}
    }
}
</script>
```

## Building for Production

When you're ready to deploy your application, build it for production:

```bash
orbiton build
```

This creates optimized builds for your target platforms in the `dist/` directory.

## Next Steps

- Learn more about [State Management](../core-concepts/state-management.md)
- Explore [Event Handling](../core-concepts/events.md)
- Check out the [Component API](../api/component-api.md)
- Browse the [OrbitKit Component Library](../api/orbitkit-components.md)

## Troubleshooting

If you encounter issues:

1. Make sure your Rust toolchain is up to date: `rustup update`
2. Check the FAQ in the documentation
3. Search for similar issues on GitHub
4. Ask for help in the community Discord server
