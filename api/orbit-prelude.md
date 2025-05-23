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

## Exported Types and Traits

The `orbit::prelude` module re-exports the following items:

### Component System

```rust
pub use orbit::component::{Component, ComponentId};
pub use orbit::props::{Props, Children};
pub use orbit::node::{Node, RenderNode, Renderable};
pub use orbit::lifecycle::{Lifecycle, LifecycleEvent};
```

### State Management

```rust
pub use orbit::state::{State, Store, Reducer, Action, use_state};
pub use orbit::effect::{Effect, EffectCleanup, use_effect};
pub use orbit::memo::{Memo, use_memo};
pub use orbit::ref_cell::{RefCell, use_ref};
```

### Event Handling

```rust
pub use orbit::event::{Event, EventHandler, EventPhase, EventTarget};
pub use orbit::event::dom::{
    ClickEvent, InputEvent, KeyboardEvent, MouseEvent, 
    FocusEvent, FormEvent, DragEvent, TouchEvent
};
```

### Context API

```rust
pub use orbit::context::{Context, Provider, Consumer, use_context, create_context};
```

### Attribute and Style Handling

```rust
pub use orbit::attribute::{Attribute, AttributeValue};
pub use orbit::style::{Style, StyleValue, css};
```

### Macros

```rust
pub use orbit_macros::{component, props, orbit_component, orbit_props};
```

## Component Trait

The `Component` trait is the foundation of Orbit's component system:

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
    
    /// Gets a unique identifier for this component instance
    fn id(&self) -> ComponentId {
        // Implementation provided by the framework
        self.component_id
    }
    
    /// Access the underlying DOM or render node
    fn node(&self) -> Option<&RenderNode> {
        // Implementation provided by the framework
        self.render_node.as_ref()
    }
}
```

## Props Trait

The `Props` trait is implemented for component property structures:

```rust
pub trait Props: Clone + 'static {
    // Marker trait for props types
}
```

Typically used with the `#[derive(Props)]` macro:

```rust
#[derive(Props)]
pub struct ButtonProps {
    pub label: String,
    
    #[prop(default)]
    pub disabled: bool,
    
    #[prop(event)]
    pub on_click: Option<EventHandler<ClickEvent>>,
}
```

## Event System

The event system provides a way to handle user interactions:

```rust
pub struct EventHandler<T: Event> {
    callback: Box<dyn Fn(T) + 'static>,
}

impl<T: Event> EventHandler<T> {
    pub fn new<F>(callback: F) -> Self 
    where
        F: Fn(T) + 'static,
    {
        Self {
            callback: Box::new(callback),
        }
    }
    
    pub fn emit(&self, event: T) {
        (self.callback)(event);
    }
}

pub trait Event: Clone + 'static {
    const NAME: &'static str;
    fn target(&self) -> EventTarget;
    fn prevent_default(&mut self);
    fn stop_propagation(&mut self);
}
```

## State Management

The state management system provides reactive state updates:

```rust
pub struct Store<S, A>
where
    S: State,
    A: Action,
{
    state: S,
    reducer: Box<dyn Fn(&mut S, &A)>,
    subscribers: Vec<Box<dyn Fn(&S)>>,
}

impl<S, A> Store<S, A>
where
    S: State,
    A: Action,
{
    pub fn new<R>(initial_state: S, reducer: R) -> Self
    where
        R: Fn(&mut S, &A) + 'static,
    {
        Self {
            state,
            reducer: Box::new(reducer),
            subscribers: Vec::new(),
        }
    }
    
    pub fn get_state(&self) -> &S {
        &self.state
    }
    
    pub fn dispatch(&mut self, action: A) {
        (self.reducer)(&mut self.state, &action);
        self.notify_subscribers();
    }
    
    pub fn subscribe<F>(&mut self, callback: F)
    where
        F: Fn(&S) + 'static,
    {
        self.subscribers.push(Box::new(callback));
    }
}
```

## Context API

The Context API allows sharing data through the component tree without prop drilling:

```rust
pub fn create_context<T: Context>() -> (Provider<T>, Consumer<T>) {
    // Implementation provided by the framework
}

pub fn use_context<T: Context>() -> Option<T> {
    // Implementation provided by the framework
}
```

## Hooks

The prelude includes various hooks for state and lifecycle management:

```rust
pub fn use_state<T: Clone + 'static>(initial_value: T) -> (T, impl Fn(T)) {
    // Implementation provided by the framework
}

pub fn use_effect<F>(deps: &[Dependency], effect: F)
where
    F: FnOnce() -> Option<EffectCleanup> + 'static,
{
    // Implementation provided by the framework
}

pub fn use_memo<T, F>(deps: &[Dependency], factory: F) -> T
where
    F: FnOnce() -> T,
    T: Clone + 'static,
{
    // Implementation provided by the framework
}
```

## Usage Examples

### Basic Component

```rust
use orbit::prelude::*;

#[derive(Props)]
pub struct GreetingProps {
    pub name: String,
}

pub struct Greeting {
    props: GreetingProps,
}

impl Component for Greeting {
    type Props = GreetingProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
}
```

### Component with State

```rust
use orbit::prelude::*;

#[derive(Props)]
pub struct CounterProps {
    #[prop(default)]
    pub initial: i32,
}

pub struct Counter {
    props: CounterProps,
    count: i32,
}

impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            props,
            count: props.initial,
        }
    }
    
    fn increment(&mut self) {
        self.count += 1;
        self.request_update();
    }
    
    fn decrement(&mut self) {
        self.count -= 1;
        self.request_update();
    }
}
```

### Using the Event System

```rust
use orbit::prelude::*;

#[derive(Props)]
pub struct ButtonProps {
    pub label: String,
    
    #[prop(event)]
    pub on_click: Option<EventHandler<ClickEvent>>,
}

pub struct Button {
    props: ButtonProps,
}

impl Component for Button {
    type Props = ButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
    
    fn handle_click(&mut self, event: ClickEvent) {
        if let Some(handler) = &self.props.on_click {
            handler.emit(event);
        }
    }
}
```

### Using the Store API

```rust
use orbit::prelude::*;

// State
pub struct AppState {
    count: i32,
    user: Option<String>,
}

impl Default for AppState {
    fn default() -> Self {
        Self {
            count: 0,
            user: None,
        }
    }
}

impl State for AppState {}

// Actions
pub enum AppAction {
    Increment,
    Decrement,
    SetUser(String),
    ClearUser,
}

impl Action for AppAction {}

// App component
pub struct App {
    store: Store<AppState, AppAction>,
}

impl Component for App {
    type Props = ();
    
    fn new(_props: Self::Props) -> Self {
        let store = Store::new(
            AppState::default(),
            |state, action| {
                match action {
                    AppAction::Increment => { state.count += 1; },
                    AppAction::Decrement => { state.count -= 1; },
                    AppAction::SetUser(name) => { state.user = Some(name.clone()); },
                    AppAction::ClearUser => { state.user = None; },
                }
            }
        );
        
        Self { store }
    }
}
```

### Using Context

```rust
use orbit::prelude::*;

// Define a context type
pub struct ThemeContext {
    pub primary_color: String,
    pub secondary_color: String,
    pub is_dark_mode: bool,
}

impl Context for ThemeContext {}

impl Default for ThemeContext {
    fn default() -> Self {
        Self {
            primary_color: "#0066cc".to_string(),
            secondary_color: "#4d90fe".to_string(),
            is_dark_mode: false,
        }
    }
}

// Theme provider component
pub struct ThemeProvider {
    context: ThemeContext,
    props: ThemeProviderProps,
}

#[derive(Props)]
pub struct ThemeProviderProps {
    #[prop(default)]
    pub is_dark_mode: bool,
    
    #[prop(default)]
    pub children: Option<Children>,
}

impl Component for ThemeProvider {
    type Props = ThemeProviderProps;
    
    fn new(props: Self::Props) -> Self {
        let mut context = ThemeContext::default();
        context.is_dark_mode = props.is_dark_mode;
        
        if props.is_dark_mode {
            context.primary_color = "#4d90fe".to_string();
            context.secondary_color = "#0066cc".to_string();
        }
        
        Self { context, props }
    }
}

// Consumer component
pub struct ThemedButton {
    props: ThemedButtonProps,
}

#[derive(Props)]
pub struct ThemedButtonProps {
    pub label: String,
}

impl Component for ThemedButton {
    type Props = ThemedButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
    
    fn render(&self) -> Node {
        // Use context to get theme
        if let Some(theme) = use_context::<ThemeContext>() {
            // Use theme values in rendering
            let background_color = &theme.primary_color;
            let text_color = if theme.is_dark_mode { "#ffffff" } else { "#000000" };
            
            // Render with theme values
        } else {
            // Fallback rendering
        }
    }
}
```

## Best Practices

- Import the prelude at the top of your `.orbit` files or Rust modules
- Use type annotations when necessary to avoid ambiguity
- Combine the prelude with specific imports for less common types
- Prefer importing specific types for library code to make dependencies explicit

## See Also

- [orbit::component](orbit-component.md) - Detailed component API
- [orbit::state](orbit-state.md) - State management API
- [orbit::event](orbit-event.md) - Event handling API
- [orbit::render](orbit-render.md) - Rendering API
