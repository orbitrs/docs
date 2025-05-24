# 🦀 Rust Examples

The OrbitRS SDK provides several Rust examples to help you understand how to use the framework with native Rust code. These examples demonstrate core concepts like props, events, component lifecycle, and different rendering approaches.

## Component System Examples

### Props Minimal
A minimal example demonstrating the props system:

```rust
// src/props_minimal.rs
use orbit::component::props::*;
use orbit::define_props;

// Example using the basic props macro
define_props! {
    pub struct ButtonProps {
        pub label: String,
        pub disabled: bool,
        pub size: String
    }
}

fn main() {
    println!("Props system minimal example");

    // Using basic props
    let props = ButtonProps {
        label: "Click me".to_string(),
        disabled: false,
        size: "large".to_string(),
    };

    println!(
        "Basic props: label={}, disabled={}, size={}",
        props.label, props.disabled, props.size
    );

    // Using builder pattern
    let props_builder = ButtonProps::builder();
    let props = props_builder.build().unwrap();

    println!(
        "Builder props: label={}, disabled={}, size={}",
        props.label, props.disabled, props.size
    );
}
```

### Props Example
A more advanced props example with validation:

```rust
use orbit::component::props::*;
use orbit::define_props;

// Example using the basic props macro
define_props! {
    pub struct ButtonProps {
        pub label: String,
        pub disabled: bool,
        pub size: String
    }
}

// Create a validator for ButtonProps that ensures label is not empty
struct ButtonPropsValidator;

impl PropValidator<ButtonProps> for ButtonPropsValidator {
    fn validate(&self, props: &ButtonProps) -> Result<(), PropValidationError> {
        if props.label.is_empty() {
            return Err(PropValidationError::InvalidValue {
                name: "label".to_string(),
                reason: "Button label cannot be empty".to_string(),
            });
        }
        Ok(())
    }
}
```

### Component Lifecycle
Example demonstrating component lifecycle methods:

```rust
struct Counter {
    context: Context,
    props: CounterProps,
    // Use Arc<RwLock<T>> for thread-safety
    count: Arc<RwLock<i32>>,
    double_count: i32,
}

impl Component for Counter {
    type Props = CounterProps;

    fn create(props: Self::Props, context: Context) -> Self {
        // Create a thread-safe state for the count
        let count = Arc::new(RwLock::new(props.initial));

        // Pre-compute the double count
        let double_count = props.initial * 2;

        Self {
            context,
            props,
            count,
            double_count,
        }
    }

    fn initialize(&mut self) -> Result<(), ComponentError> {
        let count = match self.count.read() {
            Ok(guard) => *guard,
            Err(_) => {
                return Err(ComponentError::MountError(
                    "Failed to read count".to_string(),
                ))
            }
        };

        println!("Counter component initialized with count {}", count);

        // Register lifecycle hooks
        self.context.on_mount(|_| {
            println!("Counter mounted");
        });

        self.context.on_unmount(|_| {
            println!("Counter unmounted");
        });

        Ok(())
    }
}
```

## Rendering Examples

### Advanced Skia
Example showing advanced Skia rendering:

```rust
// src/advanced_skia.rs
// Advanced Skia test with custom rendering
use orbit::platform::PlatformType;

fn main() -> Result<(), orbit::Error> {
    // Create desktop adapter
    let mut adapter = orbit::platform::create_adapter(PlatformType::Desktop);

    // This example shows how we would use the platform.rs API in practice
    adapter.init()?;

    // Run the app
    adapter.run()?;

    Ok(())
}
```

### WGPU Renderer
Example demonstrating 3D rendering with WGPU:

```rust
// src/wgpu_renderer.rs
//! Example demonstrating the WGPU renderer with 3D content

use std::time::{Duration, Instant};

use orbit::{
    component::{Component, ComponentError, Context, Node},
    renderer::{create_renderer, RendererType},
};

/// A simple 3D scene component
pub struct Scene3D {
    #[allow(dead_code)]
    context: Context,
    rotation: f32,
    last_update: Instant,
}

impl Component for Scene3D {
    type Props = ();

    fn create(_props: Self::Props, context: Context) -> Self {
        Self {
            context,
            rotation: 0.0,
            last_update: Instant::now(),
        }
    }

    fn update(&mut self, _props: Self::Props) -> Result<(), ComponentError> {
        // Update rotation
        let now = Instant::now();
        let dt = now.duration_since(self.last_update).as_secs_f32();
        self.last_update = now;

        self.rotation += dt * 0.5; // Rotate 0.5 radians per second

        Ok(())
    }
}

/// Main function
fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("WGPU Renderer Example");

    // Create a WGPU renderer
    let _renderer = create_renderer(RendererType::Wgpu)?;
    
    // Main loop implementation...
    
    Ok(())
}
```

## Running Examples

You can run any of these examples using Cargo:

```bash
# Navigate to the examples directory
cd /path/to/orbitrs/sdk/examples

# Run a specific example
cargo run --example props_minimal
cargo run --example component_lifecycle
cargo run --example advanced_skia
cargo run --example wgpu_renderer
```

## Project Structure

The examples project is now organized with all examples in the `src/` directory and properly configured in `Cargo.toml` to prevent files from being in multiple build targets.

This structure ensures clean builds without warnings and provides a comprehensive set of examples covering the core functionality of the OrbitRS framework.
