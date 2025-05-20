# 🚀 Getting Started with Orbit

This guide will help you get up and running with the Orbit UI Framework quickly.

## 📋 Prerequisites

- Rust (latest stable version)
- Cargo (included with Rust)
- Node.js (for web development targets)
- Git

## 🔧 Installation

Install the Orbit CLI tool (orbiton) which will help you manage your Orbit projects:

```bash
cargo install orbiton
```

## 🏗️ Creating Your First Project

You can create a new Orbit project using the CLI:

```bash
# Create a new project
orbiton new my-first-app

# Navigate to your new project
cd my-first-app
```

This will scaffold a new project with the basic structure and dependencies you need to get started.

## 🚀 Running Your Application

Start the development server which includes hot reloading:

```bash
orbiton dev
```

This will compile your application and start a development server. Your application will be available at `http://localhost:3000` by default.

## 📁 Project Structure

A typical Orbit project structure looks like this:

```
my-first-app/
├── Cargo.toml           # Rust dependencies and project configuration
├── src/
│   ├── main.rs          # Application entry point
│   ├── app.orbit        # Main application component
│   ├── components/      # Your custom components
│   │   └── counter.orbit # Example component
│   └── styles/          # Global styles
│       └── theme.rs     # Theme configuration
├── static/              # Static assets
│   ├── images/          # Image assets
│   └── fonts/           # Font assets
└── orbiton.config.js    # Orbit CLI configuration
```

## 🧩 Creating Components

Components in Orbit use the `.orbit` file format, which combines markup, styling, and logic in a single file:

```rust
// src/components/counter.orbit

<Component>
  <Props>
    initial_count: i32 = 0
  </Props>

  <State>
    count: i32 = self.props.initial_count
  </State>

  <Style>
    .counter {
      display: flex;
      align-items: center;
      gap: 16px;
    }
    
    .button {
      padding: 8px 16px;
      border-radius: 4px;
      background-color: #3498db;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }
    
    .count {
      font-size: 24px;
      font-weight: bold;
    }
  </Style>

  <Template>
    <div class="counter">
      <button class="button" on:click={self.decrement}>-</button>
      <span class="count">{self.state.count}</span>
      <button class="button" on:click={self.increment}>+</button>
    </div>
  </Template>

  <Script>
    fn increment(&mut self) {
        self.state.count += 1;
    }
    
    fn decrement(&mut self) {
        self.state.count -= 1;
    }
  </Script>
</Component>
```

## 🔄 Next Steps

Now that you have your first Orbit application running, you might want to:

1. Explore more [examples](../examples/)
2. Learn about [core concepts](../core-concepts/)
3. Dive into the [API documentation](../api/)
4. Check out the [orbitkit](../../orbitkit/) component library

Happy building with Orbit! 🚀
