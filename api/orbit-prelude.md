# orbit::prelude API Reference

The `orbit::prelude` module provides convenient access to the most commonly used types and traits in the Orbit Framework. Importing this module with `use orbit::prelude::*` brings the core functionality into scope without having to import multiple individual modules.

## Overview

```rust
use orbit::prelude::*;
```

This single import gives you access to the fundamental building blocks for creating Orbit applications:

- Component definition and lifecycle management
- Props system for component configuration
- Event handling for user interactions
- Rendering capabilities
- State management for reactive updates

## Exported Types

### Component Trait

```rust
pub trait Component {
    /// The props type associated with this component
    type Props: Props;
    
    /// Creates a new instance of the component with the given props
    fn new(props: Self::Props) -> Self;
    
    /// Called when the component is added to the DOM
    fn mounted(&mut self) {}
    
    /// Called after the component updates (re-renders)
    fn updated(&mut self) {}
    
    /// Called before the component is removed from the DOM
    fn unmounted(&mut self) {}
    
    /// Determines if the component should update when props change
    /// Default implementation always returns true
    fn should_update(&self, new_props: &Self::Props) -> bool {
        true
    }
    
    /// Request that this component be re-rendered
    fn request_update(&mut self) {
        // Implementation provided by the framework
    }
}
```

**Example usage:**

```rust
use orbit::prelude::*;

pub struct Counter {
    count: i32,
}

impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            count: props.initial_count,
        }
    }
    
    fn mounted(&mut self) {
        println!("Counter component mounted with initial count: {}", self.count);
    }
}
```

### Props Trait

```rust
pub trait Props: Sized + Default {
    /// Create props from attributes in template
    fn from_attrs(attrs: &[Attribute]) -> Result<Self, PropError> {
        // Default implementation provided by the framework
        Ok(Self::default())
    }
}
```

**Example usage:**

```rust
use orbit::prelude::*;

#[derive(Default)]
pub struct ButtonProps {
    pub label: String,
    pub variant: ButtonVariant,
    pub on_click: Option<EventHandler<ClickEvent>>,
}

impl Props for ButtonProps {}
```

### EventEmitter

```rust
pub trait EventEmitter {
    /// Emit an event of the given type with the provided data
    fn emit<T: Event>(&self, event: T);
    
    /// Register an event handler for a specific event type
    fn on<T: Event, F: Fn(T) + 'static>(&mut self, handler: F) -> EventHandlerId;
    
    /// Unregister an event handler
    fn off(&mut self, handler_id: EventHandlerId);
}
```

**Example usage:**

```rust
use orbit::prelude::*;

pub struct CustomButton {
    props: ButtonProps,
    emitter: EventEmitter,
}

impl CustomButton {
    pub fn handle_click(&mut self) {
        // Emit a click event
        self.emitter.emit(ClickEvent {
            target: "custom-button".to_string(),
        });
    }
}
```

### Renderer

```rust
pub trait Renderer {
    /// Initialize the renderer with the given configuration
    fn init(config: RendererConfig) -> Result<Self, Error> where Self: Sized;
    
    /// Render a component to the target
    fn render<T: Component>(&mut self, component: &T) -> Result<(), Error>;
    
    /// Update a previously rendered component
    fn update<T: Component>(&mut self, component: &T) -> Result<(), Error>;
    
    /// Clean up resources used by the renderer
    fn cleanup(&mut self);
}
```

**Example usage:**

```rust
use orbit::prelude::*;

// Create a renderer instance
let mut renderer = SkiaRenderer::init(RendererConfig::default())?;

// Render a component
let app = MyAppComponent::new(MyAppProps::default());
renderer.render(&app)?;

// Cleanup when done
renderer.cleanup();
```

### State

```rust
pub struct State<T> {
    value: T,
    subscribers: Vec<Box<dyn Fn(&T)>>,
}

impl<T: Clone> State<T> {
    /// Create a new state instance with the given initial value
    pub fn new(initial: T) -> Self;
    
    /// Get the current value
    pub fn get(&self) -> &T;
    
    /// Update the value and notify subscribers
    pub fn set(&mut self, new_value: T);
    
    /// Subscribe to state changes
    pub fn subscribe<F: Fn(&T) + 'static>(&mut self, callback: F) -> SubscriptionId;
    
    /// Unsubscribe from state changes
    pub fn unsubscribe(&mut self, id: SubscriptionId);
}
```

**Example usage:**

```rust
use orbit::prelude::*;

// Create a state instance
let mut count_state = State::new(0);

// Subscribe to changes
let subscription = count_state.subscribe(|count| {
    println!("Count changed to: {}", count);
});

// Update the state
count_state.set(1); // This will trigger the subscriber

// Unsubscribe when done
count_state.unsubscribe(subscription);
```

## Common Patterns

### Component with State

```rust
use orbit::prelude::*;

pub struct Counter {
    count: State<i32>,
}

impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            count: State::new(props.initial_count),
        }
    }
    
    fn increment(&mut self) {
        let current = *self.count.get();
        self.count.set(current + 1);
        self.request_update();
    }
}
```

### Component with Event Handling

```rust
use orbit::prelude::*;

pub struct ToggleButton {
    is_active: bool,
    on_toggle: Option<EventHandler<ToggleEvent>>,
}

impl Component for ToggleButton {
    type Props = ToggleButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            is_active: props.initial_state,
            on_toggle: props.on_toggle,
        }
    }
    
    fn toggle(&mut self) {
        self.is_active = !self.is_active;
        
        if let Some(handler) = &self.on_toggle {
            handler(ToggleEvent {
                is_active: self.is_active,
            });
        }
        
        self.request_update();
    }
}
```

## Best Practices

### 1. Always import the prelude in component files

```rust
// At the top of your .orbit file's code section
use orbit::prelude::*;
```

### 2. Avoid name conflicts

The prelude is designed to provide commonly used types without conflicts, but if you encounter name conflicts, you can import specific items instead:

```rust
// Instead of using the entire prelude
use orbit::component::{Component, Props};
use orbit::state::State;
```

### 3. Extend with module-specific imports as needed

The prelude contains only the most essential types. For more specialized functionality, import specific modules:

```rust
use orbit::prelude::*;
// Add specialized imports
use orbit::renderer::skia::SkiaRendererOptions;
use orbit::platform::window::WindowConfig;
```

## Related Documentation

- [Component Model](../core-concepts/component-model.md) - Detailed explanation of Orbit's component system
- [State Management](../core-concepts/state-management.md) - In-depth guide to state management
- [Event Handling](../core-concepts/event-handling.md) - Comprehensive guide to events and handlers
