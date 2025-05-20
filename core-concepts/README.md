# 🧠 Core Concepts

This section explains the fundamental concepts and architecture of the Orbit UI Framework.

## 🧩 Component Model

Orbit's component model is built on a unified file format (`.orbit`) that combines markup, styling, and logic in a single cohesive unit.

### Component Lifecycle

Components in Orbit follow a predictable lifecycle:

1. **Creation**: Component is instantiated with initial props
2. **Mounting**: Component is added to the rendering tree
3. **Updating**: Component rerenders in response to state or prop changes
4. **Unmounting**: Component is removed from the rendering tree

### Component Composition

Components can be composed and nested to create complex UIs:

```rust
// parent.orbit
<Template>
  <ChildComponent prop1="value" />
</Template>
```

## 🎨 Rendering Architecture

Orbit uses a hybrid rendering approach that combines different rendering engines:

### Skia Rendering Engine

The Skia-based renderer provides optimized 2D UI rendering with:
- Hardware acceleration
- Vector graphics support
- Text rendering with advanced typography
- Image processing capabilities

### WGPU Rendering Engine

The WGPU-based renderer enables advanced 3D capabilities:
- GPU-accelerated rendering
- Custom shaders support
- 3D model rendering
- WebGPU compatibility

### Unified Rendering Pipeline

Both rendering engines are integrated into a unified pipeline that allows seamless mixing of 2D and 3D elements within the same application.

## 📊 State Management

Orbit provides several approaches to state management:

### Component State

Local component state defined within the component itself:

```rust
<State>
  count: i32 = 0
  name: String = "Orbit".to_string()
</State>
```

### Context API

Shared state that can be accessed by child components:

```rust
// context provider
<CreateContext id="theme">
  { dark_mode: true, primary_color: "#3498db" }
</CreateContext>

// context consumer
<UseContext id="theme" as="theme">
  <div style="color: {theme.primary_color}">
    Themed content
  </div>
</UseContext>
```

### Reactive Stores

Global state management for larger applications:

```rust
// store.rs
pub struct AppStore {
    counter: i32,
    user: Option<User>,
}

// component.orbit
<Script>
  use crate::store;
  
  fn increment(&mut self) {
    store.update(|s| s.counter += 1);
  }
</Script>
```

## 🔄 Event Handling

Orbit provides a unified event system across platforms:

### DOM-like Events

```rust
<button on:click={self.handle_click}>Click me</button>
```

### Custom Events

```rust
<Component>
  <Events>
    value_change: i32
  </Events>
  
  <Script>
    fn update_value(&mut self, new_value: i32) {
      self.emit("value_change", new_value);
    }
  </Script>
</Component>
```

### Effect System

```rust
<Effects>
  effect!(self.state.count) {
    println!("Count changed to: {}", self.state.count);
    // Perform side effects when count changes
  }
</Effects>
```

## 🌐 Cross-Platform Architecture

Orbit achieves cross-platform compatibility through:

1. **Platform Adapters**: Translate platform-specific inputs/outputs
2. **Unified Component Model**: Same components work across platforms
3. **Adaptive Styling**: Automatically adjust to platform conventions
4. **Capability Detection**: Graceful feature detection and fallbacks

## 📏 Styling System

Orbit provides a powerful styling system:

### Component-Scoped CSS

```rust
<Style>
  .container {
    display: flex;
    gap: 16px;
  }
  
  .button {
    background-color: #3498db;
    color: white;
    padding: 8px 16px;
  }
</Style>
```

### Theme Integration

```rust
<Style>
  .button {
    background-color: $theme.primary;
    color: $theme.onPrimary;
  }
</Style>
```

### Responsive Design

```rust
<Style>
  .container {
    width: 100%;
    
    @media (min-width: 768px) {
      width: 720px;
      margin: 0 auto;
    }
  }
</Style>
```

## 🚀 Performance Optimization

Orbit is designed for high performance through:

1. **Virtual DOM**: Efficient diffing and batched updates
2. **Incremental Rendering**: Only update what changed
3. **Lazy Loading**: Components loaded only when needed
4. **Optimized Compilation**: Rust's zero-cost abstractions

These core concepts form the foundation of the Orbit Framework and understanding them will help you build efficient, maintainable applications.
