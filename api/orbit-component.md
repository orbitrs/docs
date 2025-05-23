# orbit::component API Reference

The `orbit::component` module defines the core abstractions for components in the Orbit Framework. Components are the building blocks of Orbit applications, encapsulating structure, appearance, and behavior.

## Overview

Components in Orbit are Rust structs that implement the `Component` trait. Each component has associated props that configure its behavior and appearance.

## Core Types

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

### ComponentId

```rust
/// A unique identifier for a component instance
#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
pub struct ComponentId(pub(crate) u64);

impl ComponentId {
    /// Creates a new unique component ID
    pub fn new() -> Self {
        // Implementation generates a unique ID
        Self(generate_unique_id())
    }
    
    /// Gets the raw ID value
    pub fn value(&self) -> u64 {
        self.0
    }
}
```

### ComponentBuilder

```rust
/// Helper for building components with the builder pattern
pub struct ComponentBuilder<T: Component> {
    props_builder: T::Props::Builder,
}

impl<T: Component> ComponentBuilder<T> {
    /// Creates a new component builder
    pub fn new() -> Self {
        Self {
            props_builder: T::Props::builder(),
        }
    }
    
    /// Sets a prop value
    pub fn prop<P: Into<PropValue<T::Props>>>(mut self, prop: P) -> Self {
        self.props_builder = self.props_builder.prop(prop);
        self
    }
    
    /// Builds the component
    pub fn build(self) -> T {
        let props = self.props_builder.build();
        T::new(props)
    }
}
```

### ComponentList

```rust
/// A type-erased list of components
pub struct ComponentList {
    components: Vec<Box<dyn AnyComponent>>,
}

impl ComponentList {
    /// Creates a new empty component list
    pub fn new() -> Self {
        Self {
            components: Vec::new(),
        }
    }
    
    /// Adds a component to the list
    pub fn push<T: Component + 'static>(&mut self, component: T) {
        self.components.push(Box::new(component));
    }
    
    /// Gets the number of components in the list
    pub fn len(&self) -> usize {
        self.components.len()
    }
    
    /// Checks if the list is empty
    pub fn is_empty(&self) -> bool {
        self.components.is_empty()
    }
    
    /// Gets a component by index
    pub fn get<T: Component + 'static>(&self, index: usize) -> Option<&T> {
        if index < self.components.len() {
            self.components[index].as_any().downcast_ref::<T>()
        } else {
            None
        }
    }
    
    /// Gets a mutable component by index
    pub fn get_mut<T: Component + 'static>(&mut self, index: usize) -> Option<&mut T> {
        if index < self.components.len() {
            self.components[index].as_any_mut().downcast_mut::<T>()
        } else {
            None
        }
    }
}
```

### AnyComponent

```rust
/// Type-erased component trait for internal use
pub(crate) trait AnyComponent: Send {
    fn as_any(&self) -> &dyn std::any::Any;
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any;
    fn component_id(&self) -> ComponentId;
    fn mount(&mut self);
    fn update(&mut self, props_ptr: *const u8);
    fn unmount(&mut self);
    fn render(&self) -> Option<&RenderNode>;
}

impl<T: Component + 'static> AnyComponent for T {
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
    
    fn component_id(&self) -> ComponentId {
        self.id()
    }
    
    fn mount(&mut self) {
        self.mounted();
    }
    
    fn update(&mut self, props_ptr: *const u8) {
        // Implementation provided by the framework
        // Safely converts the raw pointer to the correct props type
        // and calls the component's update method
    }
    
    fn unmount(&mut self) {
        self.unmounted();
    }
    
    fn render(&self) -> Option<&RenderNode> {
        self.node()
    }
}
```

### Slots System

```rust
/// A slot in a component template
pub struct Slot {
    name: String,
    content: Option<Box<dyn AnyComponent>>,
}

impl Slot {
    /// Creates a new named slot
    pub fn new(name: impl Into<String>) -> Self {
        Self {
            name: name.into(),
            content: None,
        }
    }
    
    /// Sets the content of the slot
    pub fn set_content<T: Component + 'static>(&mut self, component: T) {
        self.content = Some(Box::new(component));
    }
    
    /// Checks if the slot has content
    pub fn has_content(&self) -> bool {
        self.content.is_some()
    }
    
    /// Gets the name of the slot
    pub fn name(&self) -> &str {
        &self.name
    }
}
```

### Children Types

```rust
/// Children of a component (used in props)
#[derive(Clone)]
pub struct Children {
    nodes: Vec<RenderNode>,
}

impl Children {
    /// Creates new empty children
    pub fn new() -> Self {
        Self { nodes: Vec::new() }
    }
    
    /// Adds a child node
    pub fn push(&mut self, node: RenderNode) {
        self.nodes.push(node);
    }
    
    /// Gets all child nodes
    pub fn nodes(&self) -> &[RenderNode] {
        &self.nodes
    }
    
    /// Checks if there are any children
    pub fn is_empty(&self) -> bool {
        self.nodes.is_empty()
    }
    
    /// Gets the number of children
    pub fn len(&self) -> usize {
        self.nodes.len()
    }
}
```

## Component Registration

```rust
/// Registers a component type with the framework
pub fn register_component<T: Component + 'static>() {
    // Implementation registers the component type for
    // use in template compilation
}

/// Factory function for creating components by name
pub fn create_component(name: &str, props: &dyn std::any::Any) -> Option<Box<dyn AnyComponent>> {
    // Implementation looks up the component type by name
    // and creates a new instance with the provided props
}
```

## Component Macros

```rust
/// Derive macro for implementing the Component trait
#[proc_macro_derive(Component, attributes(orbit))]
pub fn derive_component(input: TokenStream) -> TokenStream {
    // Implementation generates code for the Component trait
}

/// Attribute macro for defining a component
#[proc_macro_attribute]
pub fn component(attrs: TokenStream, item: TokenStream) -> TokenStream {
    // Implementation transforms the component definition
}

/// Derive macro for implementing the Props trait
#[proc_macro_derive(Props, attributes(prop))]
pub fn derive_props(input: TokenStream) -> TokenStream {
    // Implementation generates code for the Props trait
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

### Using Component Lifecycle Methods

```rust
use orbit::prelude::*;

#[derive(Props)]
pub struct TimerProps {
    pub interval: u64,
    #[prop(event)]
    pub on_tick: Option<EventHandler<()>>,
}

pub struct Timer {
    props: TimerProps,
    timer_id: Option<u32>,
}

impl Component for Timer {
    type Props = TimerProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            props,
            timer_id: None,
        }
    }
    
    fn mounted(&mut self) {
        // Start the timer when the component mounts
        let interval = self.props.interval;
        let on_tick = self.props.on_tick.clone();
        
        self.timer_id = Some(set_interval(move || {
            if let Some(handler) = &on_tick {
                handler.emit(());
            }
        }, interval));
    }
    
    fn unmounted(&mut self) {
        // Clean up the timer when the component unmounts
        if let Some(id) = self.timer_id.take() {
            clear_interval(id);
        }
    }
    
    fn should_update(&self, new_props: &Self::Props) -> bool {
        // Only update if the interval has changed
        self.props.interval != new_props.interval
    }
    
    fn updated(&mut self) {
        // Reset the timer with the new interval
        if let Some(id) = self.timer_id.take() {
            clear_interval(id);
        }
        
        let interval = self.props.interval;
        let on_tick = self.props.on_tick.clone();
        
        self.timer_id = Some(set_interval(move || {
            if let Some(handler) = &on_tick {
                handler.emit(());
            }
        }, interval));
    }
}
```

### Using ComponentBuilder

```rust
use orbit::prelude::*;
use orbit::component::ComponentBuilder;

// Define a component
#[derive(Props)]
pub struct CardProps {
    pub title: String,
    #[prop(default)]
    pub elevated: bool,
    #[prop(default)]
    pub children: Option<Children>,
}

pub struct Card {
    props: CardProps,
}

impl Component for Card {
    type Props = CardProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
}

// Create a component using the builder pattern
fn create_card() -> Card {
    ComponentBuilder::<Card>::new()
        .prop("title", "Hello, World!")
        .prop("elevated", true)
        .build()
}
```

### Component with Slots

```rust
use orbit::prelude::*;

#[derive(Props)]
pub struct LayoutProps {
    #[prop(default)]
    pub header: Option<Children>,
    #[prop(default)]
    pub content: Option<Children>,
    #[prop(default)]
    pub footer: Option<Children>,
}

pub struct Layout {
    props: LayoutProps,
    slots: HashMap<String, Slot>,
}

impl Component for Layout {
    type Props = LayoutProps;
    
    fn new(props: Self::Props) -> Self {
        let mut slots = HashMap::new();
        
        let mut header_slot = Slot::new("header");
        if let Some(header) = &props.header {
            // Populate the header slot from props
        }
        slots.insert("header".to_string(), header_slot);
        
        let mut content_slot = Slot::new("content");
        if let Some(content) = &props.content {
            // Populate the content slot from props
        }
        slots.insert("content".to_string(), content_slot);
        
        let mut footer_slot = Slot::new("footer");
        if let Some(footer) = &props.footer {
            // Populate the footer slot from props
        }
        slots.insert("footer".to_string(), footer_slot);
        
        Self { props, slots }
    }
}
```

### Dynamic Component Creation

```rust
use orbit::prelude::*;
use orbit::component::create_component;

// Function that creates a component dynamically by name
fn create_dynamic_component(name: &str, props: &dyn std::any::Any) -> Option<Box<dyn AnyComponent>> {
    create_component(name, props)
}

// Usage
fn render_dynamic_ui(component_name: &str) {
    // Create dynamic props
    let props = match component_name {
        "Button" => {
            let mut props = ButtonProps::builder();
            props = props.prop("label", "Click me");
            props.build()
        },
        "Card" => {
            let mut props = CardProps::builder();
            props = props.prop("title", "Dynamic Card");
            props.build()
        },
        _ => return,
    };
    
    // Create the component
    if let Some(component) = create_dynamic_component(component_name, &props) {
        // Use the component...
    }
}
```

### Component Composition

```rust
use orbit::prelude::*;

// Child component
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
}

// Parent component
#[derive(Props)]
pub struct ToolbarProps {
    #[prop(event)]
    pub on_save: Option<EventHandler<()>>,
    #[prop(event)]
    pub on_cancel: Option<EventHandler<()>>,
}

pub struct Toolbar {
    props: ToolbarProps,
}

impl Component for Toolbar {
    type Props = ToolbarProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
    
    fn handle_save(&mut self, _: ClickEvent) {
        if let Some(handler) = &self.props.on_save {
            handler.emit(());
        }
    }
    
    fn handle_cancel(&mut self, _: ClickEvent) {
        if let Some(handler) = &self.props.on_cancel {
            handler.emit(());
        }
    }
}

// Usage in template would look like:
// <Button label="Save" on_click={self.handle_save} />
// <Button label="Cancel" on_click={self.handle_cancel} />
```

## Component Testing

```rust
use orbit::prelude::*;
use orbit::test::{render_component, find_by_text, click};

#[test]
fn test_button_click() {
    let clicked = Rc::new(RefCell::new(false));
    let clicked_clone = clicked.clone();
    
    // Define the event handler
    let on_click = EventHandler::new(move |_| {
        *clicked_clone.borrow_mut() = true;
    });
    
    // Create button props
    let props = ButtonProps {
        label: "Click me".to_string(),
        on_click: Some(on_click),
    };
    
    // Render the component
    let component = render_component::<Button>(props);
    
    // Find the button and click it
    let button = find_by_text(&component, "Click me").unwrap();
    click(&button);
    
    // Assert that the event handler was called
    assert_eq!(*clicked.borrow(), true);
}
```

## Best Practices

### Component Design

- Keep components focused on a single responsibility
- Use props for configuration
- Avoid mutable state when possible
- Implement `should_update` to optimize rendering performance
- Clean up resources in `unmounted`

### Props Design

- Use descriptive prop names
- Provide sensible defaults where appropriate
- Use `#[prop(default)]` for optional props
- Use `#[prop(event)]` for event handlers
- Document props with comments

### Component Composition

- Prefer composition over inheritance
- Create small, reusable components
- Use slots for flexible layout

### Performance Optimization

- Implement `should_update` to prevent unnecessary updates
- Avoid excessive state changes
- Memoize expensive computations
- Clean up subscriptions and timers in `unmounted`

## See Also

- [Component Model](../core-concepts/component-model.md)
- [Props System](../core-concepts/props-system.md)
- [Advanced Component Patterns](../core-concepts/advanced-component-patterns.md)
- [Testing Strategies](../guides/testing-strategies.md)
