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

### Props Trait

```rust
pub trait Props: Sized + Default {
    /// Create props from attributes in template
    fn from_attrs(attrs: &[Attribute]) -> Result<Self, PropError> {
        // Default implementation provided by the framework
        Ok(Self::default())
    }
    
    /// Merge with default props
    fn merge_defaults(&mut self, defaults: Self) {
        // Default implementation provided by the framework
        // This is a no-op by default
    }
}
```

### ComponentId

```rust
/// Unique identifier for a component instance
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct ComponentId(u64);

impl ComponentId {
    /// Create a new unique component ID
    pub fn new() -> Self {
        // Implementation generates a unique ID
        Self(generate_unique_id())
    }
    
    /// Get the raw ID value
    pub fn raw(&self) -> u64 {
        self.0
    }
}
```

### RenderNode

```rust
/// Representation of a rendered component in the DOM or render tree
pub struct RenderNode {
    id: NodeId,
    component_id: ComponentId,
    children: Vec<NodeId>,
    parent: Option<NodeId>,
    data: Box<dyn Any>,
}

impl RenderNode {
    /// Get the node ID
    pub fn id(&self) -> NodeId {
        self.id
    }
    
    /// Get the component ID associated with this node
    pub fn component_id(&self) -> ComponentId {
        self.component_id
    }
    
    /// Get references to child nodes
    pub fn children(&self) -> &[NodeId] {
        &self.children
    }
    
    /// Get the parent node, if any
    pub fn parent(&self) -> Option<NodeId> {
        self.parent
    }
    
    /// Get the underlying data as a specific type
    pub fn data<T: 'static>(&self) -> Option<&T> {
        self.data.downcast_ref::<T>()
    }
}
```

### ComponentContext

```rust
/// Context available to components during rendering and lifecycle methods
pub struct ComponentContext {
    id: ComponentId,
    parent_id: Option<ComponentId>,
    renderer: Rc<RefCell<dyn Renderer>>,
    event_bus: Rc<RefCell<EventBus>>,
}

impl ComponentContext {
    /// Get the component ID
    pub fn id(&self) -> ComponentId {
        self.id
    }
    
    /// Get the parent component ID, if any
    pub fn parent_id(&self) -> Option<ComponentId> {
        self.parent_id
    }
    
    /// Access the renderer
    pub fn renderer(&self) -> &Rc<RefCell<dyn Renderer>> {
        &self.renderer
    }
    
    /// Access the event bus
    pub fn event_bus(&self) -> &Rc<RefCell<EventBus>> {
        &self.event_bus
    }
    
    /// Register a context value
    pub fn register_context<T: 'static>(&self, value: T) {
        // Implementation provided by the framework
    }
    
    /// Get a context value
    pub fn context<T: 'static>(&self) -> Option<T> where T: Clone {
        // Implementation provided by the framework
        None
    }
}
```

## Common Usage Patterns

### Basic Component Implementation

```rust
use orbit::component::{Component, Props};

pub struct ButtonProps {
    pub label: String,
    pub on_click: Option<Box<dyn Fn()>>,
}

impl Default for ButtonProps {
    fn default() -> Self {
        Self {
            label: "Button".to_string(),
            on_click: None,
        }
    }
}

impl Props for ButtonProps {}

pub struct Button {
    props: ButtonProps,
    is_hovered: bool,
}

impl Component for Button {
    type Props = ButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            props,
            is_hovered: false,
        }
    }
    
    fn mounted(&mut self) {
        println!("Button mounted: {}", self.props.label);
    }
    
    fn unmounted(&mut self) {
        println!("Button unmounted: {}", self.props.label);
    }
}

impl Button {
    pub fn handle_click(&mut self) {
        if let Some(on_click) = &self.props.on_click {
            on_click();
        }
    }
}
```

### Component with Children

```rust
use orbit::component::{Component, Props, Children};

pub struct CardProps {
    pub title: String,
    pub children: Children,
}

impl Default for CardProps {
    fn default() -> Self {
        Self {
            title: "Card".to_string(),
            children: Children::new(),
        }
    }
}

impl Props for CardProps {}

pub struct Card {
    props: CardProps,
}

impl Component for Card {
    type Props = CardProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
    
    // Render implementation would include slots for children
}
```

### Component with Optimization

```rust
use orbit::component::{Component, Props};

pub struct DataTableProps {
    pub data: Vec<DataRow>,
    pub columns: Vec<ColumnDef>,
}

impl Default for DataTableProps {
    fn default() -> Self {
        Self {
            data: Vec::new(),
            columns: Vec::new(),
        }
    }
}

impl Props for DataTableProps {}

pub struct DataTable {
    props: DataTableProps,
    sorted_data: Vec<DataRow>,
}

impl Component for DataTable {
    type Props = DataTableProps;
    
    fn new(props: Self::Props) -> Self {
        let mut sorted_data = props.data.clone();
        // Initial sort...
        
        Self {
            props,
            sorted_data,
        }
    }
    
    fn should_update(&self, new_props: &Self::Props) -> bool {
        // Only update if data or columns change
        self.props.data != new_props.data || 
        self.props.columns != new_props.columns
    }
}
```

## Advanced Features

### Component Registration

```rust
use orbit::component::{register_component, ComponentRegistry};

// Register a component type at the application level
register_component::<Button>("Button");

// Get the registry
let registry = ComponentRegistry::global();

// Create a component instance by name
let button = registry.create("Button", ButtonProps::default())?;
```

### Slots

```rust
use orbit::component::{Slot, NamedSlots};

pub struct TabsProps {
    pub active_tab: String,
    pub slots: NamedSlots,
}

impl TabsProps {
    pub fn tab(&self, name: &str) -> Option<&Slot> {
        self.slots.get(name)
    }
}

// Usage in template:
// <Tabs active_tab="settings">
//   <template v-slot:home>Home Content</template>
//   <template v-slot:settings>Settings Content</template>
// </Tabs>
```

### Dynamic Components

```rust
use orbit::component::{DynamicComponent, ComponentType};

pub struct DynamicProps {
    pub component_type: ComponentType,
    pub props: Box<dyn Props>,
}

pub struct DynamicWrapper {
    component: Box<dyn Component>,
}

impl Component for DynamicWrapper {
    type Props = DynamicProps;
    
    fn new(props: Self::Props) -> Self {
        let component = props.component_type.create(props.props);
        Self { component }
    }
    
    // Delegate lifecycle methods to the inner component
    fn mounted(&mut self) {
        self.component.mounted();
    }
    
    fn updated(&mut self) {
        self.component.updated();
    }
    
    fn unmounted(&mut self) {
        self.component.unmounted();
    }
}
```

### Component Inheritance

While Rust doesn't have traditional inheritance, you can use composition and delegation patterns:

```rust
use orbit::component::{Component, Props};

pub struct BaseButtonProps {
    pub label: String,
}

pub struct Button {
    props: BaseButtonProps,
}

// Extended button with icon
pub struct IconButtonProps {
    pub base: BaseButtonProps,
    pub icon: String,
}

pub struct IconButton {
    base: Button,
    props: IconButtonProps,
}

impl Component for IconButton {
    type Props = IconButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            base: Button::new(props.base.clone()),
            props,
        }
    }
    
    // Delegate lifecycle methods or override as needed
    fn mounted(&mut self) {
        self.base.mounted();
        // Additional icon button mounting logic
    }
}
```

## Error Handling

```rust
/// Errors that can occur when working with props
#[derive(Debug, thiserror::Error)]
pub enum PropError {
    #[error("Missing required prop: {0}")]
    MissingRequiredProp(String),
    
    #[error("Invalid prop type: {0}")]
    InvalidPropType(String),
    
    #[error("Prop validation failed: {0}")]
    ValidationFailed(String),
}

/// Errors that can occur when working with components
#[derive(Debug, thiserror::Error)]
pub enum ComponentError {
    #[error("Component not found: {0}")]
    ComponentNotFound(String),
    
    #[error("Failed to create component: {0}")]
    CreationFailed(String),
    
    #[error("Prop error: {0}")]
    PropError(#[from] PropError),
}
```

## Best Practices

### 1. Keep Components Focused

Each component should have a single responsibility. If a component is becoming complex, consider breaking it into smaller, more focused components.

### 2. Optimize Renders with `should_update`

Implement `should_update` to prevent unnecessary re-renders, especially for components with expensive rendering logic.

### 3. Clean Up Resources

Always clean up resources in the `unmounted` method to prevent memory leaks, especially for components that set up timers, event listeners, or other subscriptions.

### 4. Use Appropriate Prop Types

Design props to be type-safe and intuitive. Use `Option<T>` for optional props and provide sensible defaults.

### 5. Document Component Contracts

Clearly document the expected props, events, and behavior of your components to make them easier to use and maintain.

## Related Documentation

- [Component Model](../core-concepts/component-model.md) - High-level overview of the component system
- [Advanced Component Patterns](../core-concepts/advanced-component-patterns.md) - Advanced usage techniques
- [Template Syntax](../core-concepts/template-syntax.md) - How to define component templates
