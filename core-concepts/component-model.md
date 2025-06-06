# Orbit Component Model

This document provides a comprehensive overview of the Orbit Framework's component model, explaining the structure, lifecycle, and best practices for creating components.

## Overview

Components are the building blocks of Orbit applications. Each component encapsulates:

1. **Structure** (markup/template)
2. **Appearance** (styles)
3. **Behavior** (Rust code)

Orbit uses a single-file component format (`.orbit`) inspired by Vue, Blazor, and similar frameworks, but with Rust as the programming language.

## Component File Structure

A typical `.orbit` file has three sections:

```orbit
<template>
  <!-- HTML-like markup structure -->
  <div class="greeting">
    <h1>Hello, {{ name }}!</h1>
    <button @click="handleClick">Click me</button>
  </div>
</template>

<style>
  /* CSS styling */
  .greeting {
    padding: 20px;
    border: 1px solid #ccc;
  }
  
  h1 {
    color: #0070f3;
  }
</style>

<code lang="rust">
  // Rust code
  use orbit::prelude::*;
  
  pub struct MyComponent {
      name: String,
  }
  
  pub struct MyComponentProps {
      pub name: String,
  }
  
  impl Props for MyComponentProps {}
  
  impl Component for MyComponent {
      type Props = MyComponentProps;
      
      fn new(props: Self::Props) -> Self {
          Self {
              name: props.name,
          }
      }
      
      // Other required methods...
  }
  
  impl MyComponent {
      fn handle_click(&mut self) {
          println!("Button clicked!");
      }
  }
</code>
```

## Component Lifecycle

Orbit components follow a predictable lifecycle with comprehensive hooks:

1. **Creation**: `fn create(props: Self::Props, context: Context) -> Self`
   - Component is instantiated with initial props and context
   - Initial state is set up
   - Component ID is assigned

2. **Initialization**: `fn initialize(&mut self) -> Result<(), ComponentError>`
   - Immediately after creation
   - Set up initial state and register lifecycle hooks
   - Runs before the component is mounted

3. **Before Mount**: `fn before_mount(&mut self) -> Result<(), ComponentError>`
   - Just before component is added to the component tree
   - Preparation for mounting

4. **Mounting**: `fn mount(&mut self) -> Result<(), ComponentError>`
   - Basic mounting logic
   - Component is added to the component tree

5. **Enhanced Mount**: `fn on_mount(&mut self, context: &MountContext) -> Result<(), ComponentError>`
   - Called with detailed mount context information
   - Access to parent component ID and mounting options

6. **After Mount**: `fn after_mount(&mut self) -> Result<(), ComponentError>`
   - After component is fully mounted
   - Good place for initialization that requires DOM/tree access
   - Setup subscriptions or timers

7. **State Change**: `fn state_changed(&mut self, state_key: &str) -> Result<(), ComponentError>`
   - When component state changes
   - Can trigger component updates

8. **Update**: `fn on_update(&mut self, changes: &StateChanges) -> Result<(), ComponentError>`
   - Enhanced update with state change information
   - Runs when state changes are detected

9. **Before Update**: `fn before_update(&mut self, new_props: &Self::Props) -> Result<(), ComponentError>`
   - Called before props are updated
   - Access to both old and new props

10. **Update Props**: `fn update(&mut self, props: Self::Props) -> Result<(), ComponentError>`
    - Update component with new props
    - Should update internal state based on props

11. **After Update**: `fn after_update(&mut self) -> Result<(), ComponentError>`
    - Called after the component has been updated
    - Access to the updated component state

12. **Before Unmount**: `fn before_unmount(&mut self) -> Result<(), ComponentError>`
    - Component is about to be removed
    - Prepare for cleanup

13. **Unmount**: `fn unmount(&mut self) -> Result<(), ComponentError>`
    - Basic unmount logic
    - Component is removed from the component tree

14. **Enhanced Unmount**: `fn on_unmount(&mut self, context: &UnmountContext) -> Result<(), ComponentError>`
    - Called with detailed unmount context (reason, parent, etc)
    - More controlled unmounting with context

15. **After Unmount**: `fn after_unmount(&mut self) -> Result<(), ComponentError>`
    - Final cleanup after unmounting
    - Release resources, unsubscribe from events

16. **Cleanup**: `fn cleanup(&mut self) -> Result<(), ComponentError>`
    - Automatic framework-provided cleanup
    - Cleans up state subscriptions, event listeners, etc.

Here's a simplified implementation example:

```rust
impl Component for MyComponent {
    type Props = MyComponentProps;
    
    fn component_id(&self) -> orbit::component::ComponentId {
        orbit::component::ComponentId::new()
    }
    
    fn create(props: Self::Props, context: Context) -> Self {
        Self {
            context,
            name: props.name,
            // Initialize other state
        }
    }
    
    fn initialize(&mut self) -> Result<(), ComponentError> {
        println!("Component initialized");
        // Register lifecycle hooks, set up initial state
        Ok(())
    }
    
    fn mount(&mut self) -> Result<(), ComponentError> {
        println!("Component mounted");
        // Basic mounting logic
        Ok(())
    }
    
    fn on_mount(&mut self, context: &MountContext) -> Result<(), ComponentError> {
        println!("Component mounted with parent {:?}", context.parent_id);
        // Enhanced mounting with context
        Ok(())
    }
    
    fn update(&mut self, props: Self::Props) -> Result<(), ComponentError> {
        println!("Component updated with new props");
        self.name = props.name;
        Ok(())
    }
    
    fn before_unmount(&mut self) -> Result<(), ComponentError> {
        println!("Component will unmount soon");
        // Prepare for cleanup
        Ok(())
    }
    
    fn unmount(&mut self) -> Result<(), ComponentError> {
        println!("Component unmounted");
        // Cleanup resources
        Ok(())
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        // Render the component
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}
```

## Props System

Props are the mechanism for passing data from parent to child components.

### Defining Props

```rust
pub struct ButtonProps {
    pub label: String,
    pub color: Option<String>,
    pub size: ButtonSize,
    pub on_click: Option<Callback<ClickEvent>>,
}

impl Props for ButtonProps {}

// Default implementation for props
impl Default for ButtonProps {
    fn default() -> Self {
        Self {
            label: "Button".to_string(),
            color: None,
            size: ButtonSize::Medium,
            on_click: None,
        }
    }
}
```

### Using Props in Templates

```orbit
<template>
  <Button
    label="Click me!"
    color="primary"
    :size="dynamicSize"
    @click="handleButtonClick"
  />
</template>
```

## State Management

Components can manage internal state that changes over time.

### Local Component State

State variables are defined in the component struct:

```rust
pub struct Counter {
    // Props
    label: String,
    
    // State
    count: i32,
    is_active: bool,
}
```

### Reactive State Updates

When state changes, the component automatically re-renders:

```rust
impl Counter {
    fn increment(&mut self) {
        self.count += 1;
        // The framework automatically detects this change and re-renders
    }
    
    fn toggle_active(&mut self) {
        self.is_active = !self.is_active;
    }
}
```

### State Initialization from Props

Props can be used to initialize state:

```rust
impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            label: props.label,
            count: props.initial_count.unwrap_or(0),
            is_active: false,
        }
    }
}
```

## Event Handling

Orbit provides several ways to handle events.

### DOM-like Events

Similar to HTML events, but with Rust handlers:

```orbit
<template>
  <button @click="handleClick">Click Me</button>
  <input @input="handleInput" @focus="handleFocus" />
</template>

<code lang="rust">
  impl MyComponent {
      fn handle_click(&mut self, event: ClickEvent) {
          // Handle click event
      }
      
      fn handle_input(&mut self, event: InputEvent) {
          let value = event.target_value();
          // Process input
      }
      
      fn handle_focus(&mut self, _event: FocusEvent) {
          // Handle focus
      }
  }
</code>
```

### Custom Component Events

Components can emit and handle custom events:

```rust
// Child component
impl ChildComponent {
    fn trigger_custom_event(&mut self) {
        let data = CustomEventData { value: self.value };
        self.emit("custom-event", data);
    }
}

// Parent component
<template>
  <ChildComponent @custom-event="handleCustomEvent" />
</template>

<code lang="rust">
  impl ParentComponent {
      fn handle_custom_event(&mut self, data: CustomEventData) {
          println!("Custom event received with value: {}", data.value);
      }
  }
</code>
```

## Templates and Expressions

### Data Binding

```orbit
<template>
  <div>{{ message }}</div>
  <input :value="inputValue" />
  <div :class="{ active: isActive, disabled: !isEnabled }">
    Styled content
  </div>
</template>
```

### Conditionals

```orbit
<template>
  <div v-if="count > 0">Positive</div>
  <div v-else-if="count < 0">Negative</div>
  <div v-else>Zero</div>
</template>
```

### Lists/Iterations

```orbit
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>
```

### Slots

Slots allow components to accept and render child content:

```orbit
<!-- Component definition -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">Default Header</slot>
    </div>
    <div class="card-body">
      <slot>Default Content</slot>
    </div>
    <div class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<!-- Component usage -->
<template>
  <Card>
    <template #header>
      <h2>Custom Header</h2>
    </template>
    
    <p>This goes in the default slot</p>
    
    <template #footer>
      <button>Action</button>
    </template>
  </Card>
</template>
```

## Styling

### Component-Scoped CSS

Styles in the `<style>` section are automatically scoped to the component by default:

```orbit
<style>
  /* These styles only apply to elements in this component */
  .button {
    background-color: blue;
    color: white;
  }
</style>
```

### Global Styles

You can opt out of style scoping:

```orbit
<style scoped="false">
  /* These styles apply globally */
  body {
    font-family: Arial, sans-serif;
  }
</style>
```

### Theming

Orbit supports CSS variables for theming:

```orbit
<style>
  .button {
    background-color: var(--primary-color, #0070f3);
    color: var(--primary-text-color, white);
  }
</style>
```

## Component Composition

### Basic Composition

```orbit
<template>
  <div>
    <Header title="My App" />
    <Sidebar :items="menuItems" />
    <MainContent :data="contentData" />
    <Footer />
  </div>
</template>

<code lang="rust">
  use orbit::prelude::*;
  use crate::components::{Header, Sidebar, MainContent, Footer};
  
  pub struct App {
      menu_items: Vec<MenuItem>,
      content_data: ContentData,
  }
  
  // Implementation details...
</code>
```

### Component Registration

Components must be properly imported and referenced:

```rust
// In main.rs or lib.rs
mod components {
    pub mod header;
    pub mod sidebar;
    pub mod main_content;
    pub mod footer;
}

// In component file
use crate::components::header::Header;
use crate::components::sidebar::Sidebar;
// etc.
```

## Advanced Patterns

### Component Hooks

Hooks allow you to reuse stateful logic between components:

```rust
// Define a hook
fn use_counter(initial: i32) -> (i32, Box<dyn Fn()>, Box<dyn Fn()>) {
    let count = use_state(initial);
    
    let increment = Box::new(move || {
        count.set(count.get() + 1);
    });
    
    let decrement = Box::new(move || {
        count.set(count.get() - 1);
    });
    
    (count.get(), increment, decrement)
}

// Use the hook in a component
impl Component for Counter {
    fn setup(&mut self) {
        let (count, increment, decrement) = use_counter(0);
        self.count = count;
        self.increment = increment;
        self.decrement = decrement;
    }
}
```

### Context API

Context provides a way to share values between components without passing props:

```orbit
<!-- Context provider -->
<template>
  <ThemeProvider :theme="currentTheme">
    <App />
  </ThemeProvider>
</template>

<!-- Context consumer -->
<template>
  <ThemeConsumer>
    {theme => (
      <div :style="{ backgroundColor: theme.background, color: theme.text }">
        Themed content
      </div>
    )}
  </ThemeConsumer>
</template>
```

## Performance Optimization

### Memo Components

Prevent unnecessary re-renders:

```rust
#[derive(PartialEq)]
pub struct ExpensiveProps {
    pub data: Vec<DataItem>,
}

impl Props for ExpensiveProps {}

#[memo]
pub struct ExpensiveComponent {
    data: Vec<DataItem>,
}

impl Component for ExpensiveComponent {
    type Props = ExpensiveProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            data: props.data,
        }
    }
    
    // This component will only re-render if props.data changes
}
```

### Virtualization

For long lists, use virtualization to only render visible items:

```orbit
<template>
  <VirtualList
    :items="longList"
    :item-height="50"
    :visible-height="500"
  >
    {item => <ListItem :data="item" />}
  </VirtualList>
</template>
```

## Testing Components

### Unit Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use orbit::test_utils::*;
    
    #[test]
    fn test_counter_increment() {
        // Create test component
        let props = CounterProps {
            initial: Some(5),
            ..Default::default()
        };
        
        let mut counter = Counter::new(props);
        
        // Test initial state
        assert_eq!(counter.count, 5);
        
        // Test method
        counter.increment();
        assert_eq!(counter.count, 6);
    }
    
    #[test]
    fn test_counter_render() {
        // Render test
        let wrapper = render_component::<Counter>(CounterProps::default());
        
        // Assert DOM structure
        assert!(wrapper.contains_element("button"));
        assert_eq!(wrapper.find(".counter-display").text(), "0");
        
        // Simulate event
        wrapper.find("button").trigger("click");
        
        // Assert updated state
        assert_eq!(wrapper.find(".counter-display").text(), "1");
    }
}
```

### Integration Testing

```rust
#[test]
fn test_app_integration() {
    let app = render_app();
    
    // Test component interaction
    app.find("#login-button").trigger("click");
    
    // Assert expected state change
    assert!(app.contains_element("#user-profile"));
    assert!(!app.contains_element("#login-form"));
}
```

## Best Practices

1. **Keep Components Focused**: Each component should do one thing well
2. **Prop Validation**: Use proper types and validation for props
3. **Minimize State**: Keep state as local as possible
4. **Use Composition**: Compose small components rather than building monoliths
5. **Performance Awareness**: Be mindful of rendering costs, especially in loops
6. **Clear Component APIs**: Design intuitive component interfaces
7. **Documentation**: Document component props, events, and usage examples
8. **Accessibility**: Ensure components are accessible by default

## Advanced Component Lifecycle Patterns

While the basic component lifecycle (new → mounted → updated → unmounted) covers most use cases, there are advanced patterns that can help manage complex component behavior more effectively.

### Handling Asynchronous Initialization

Components often need to load data from external sources during initialization. Here's a pattern for handling asynchronous initialization:

```rust
impl Component for DataComponent {
    type Props = DataComponentProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            data: None,
            loading: true,
            error: None,
            props,
        }
    }
    
    fn mounted(&mut self) {
        let id = self.props.id.clone();
        
        // Start loading data asynchronously
        orbit::tasks::spawn(async move {
            match fetch_data(id).await {
                Ok(data) => {
                    // Handle successful data fetch
                    self.data = Some(data);
                    self.loading = false;
                    self.request_update(); // Trigger re-render
                },
                Err(e) => {
                    // Handle error case
                    self.error = Some(e.to_string());
                    self.loading = false;
                    self.request_update(); // Trigger re-render
                }
            }
        });
    }
}
```

### Lifecycle Hooks and Effects

For more complex components, you can implement a React-like effects system:

```rust
pub struct ProfileComponent {
    user_id: String,
    user_data: Option<UserData>,
    subscription: Option<Subscription>,
}

impl Component for ProfileComponent {
    // ... other component methods ...
    
    fn updated(&mut self, old_props: Option<&Self::Props>) {
        // Check if a specific prop changed
        if let Some(old_props) = old_props {
            if old_props.user_id != self.props.user_id {
                // User ID changed, reload data
                self.load_user_data(self.props.user_id.clone());
            }
        }
    }
    
    fn unmounted(&mut self) {
        // Clean up subscription when component is removed
        if let Some(subscription) = self.subscription.take() {
            subscription.unsubscribe();
        }
    }
}

impl ProfileComponent {
    fn load_user_data(&mut self, user_id: String) {
        // Clean up existing subscription
        if let Some(subscription) = self.subscription.take() {
            subscription.unsubscribe();
        }
        
        // Create new subscription
        self.subscription = Some(
            UserService::get_user_updates(user_id)
                .subscribe(|update| {
                    self.user_data = Some(update);
                    self.request_update();
                })
        );
    }
}
```

### Component Lifecycle Debugging

When debugging component lifecycle issues, you can implement comprehensive logging:

```rust
impl Component for DebugComponent {
    // ... other component methods ...
    
    fn new(props: Self::Props) -> Self {
        log::debug!("DebugComponent::new - Creating with props: {:?}", props);
        Self { /* ... */ }
    }
    
    fn mounted(&mut self) {
        log::debug!("DebugComponent::mounted - Component ID: {}", self.id());
        // Profile render performance
        let start = std::time::Instant::now();
        
        // Initialization code
        
        let elapsed = start.elapsed();
        log::debug!("DebugComponent::mounted - Initialization took {:?}", elapsed);
    }
    
    fn updated(&mut self) {
        log::debug!("DebugComponent::updated - Component state: {:?}", self.state);
    }
    
    fn unmounted(&mut self) {
        log::debug!("DebugComponent::unmounted - Cleanup for component ID: {}", self.id());
    }
}
```

### Optimizing Re-renders with `should_update`

In performance-critical applications, you can implement `should_update` to prevent unnecessary renders:

```rust
impl Component for OptimizedComponent {
    // ... other component methods ...
    
    fn should_update(&self, new_props: &Self::Props) -> bool {
        // Only update if the important props changed
        self.props.important_value != new_props.important_value ||
        self.props.another_critical_field != new_props.another_critical_field
        
        // Ignore changes to rarely-updated or non-visual props
        // self.props.metadata != new_props.metadata
    }
}
```

### Component Composition Implementation Details

When implementing composition patterns like `RenderPropComponent`, `SlottedComponent`, or `FlexibleCompoundComponent`, the framework maintains internal context references that aren't always actively used in the component lifecycle:

```rust
pub struct RenderPropComponent<T, R>
where
    R: RenderProp<T>,
{
    component_id: ComponentId,
    data: T,
    renderer: R,
    #[allow(dead_code)]
    context: Context,  // Maintained for future lifecycle hooks
}
```

Similarly, the `MemoComponent` implementation includes a cached render field that's reserved for future optimizations:

```rust
pub struct MemoComponent<T>
where
    T: Component + Memoizable,
{
    component: T,
    last_memo_key: Option<T::MemoKey>,
    #[allow(dead_code)]
    cached_render: Option<Vec<Node>>,  // Reserved for future optimizations
    cache: Arc<MemoCache<T::MemoKey, Vec<Node>>},
}
```

These fields are marked with `#[allow(dead_code)]` as they're maintained for API consistency and future enhancements but may not be actively used in all scenarios.

### Managing Complex State Transitions

For components with multiple state transitions:

```rust
enum ComponentState {
    Initial,
    Loading,
    Loaded(Data),
    Error(String),
}

impl Component for StatefulComponent {
    // ... other component methods ...
    
    fn render(&self) -> RenderResult {
        match &self.state {
            ComponentState::Initial => html! { <div>Initializing...</div> },
            ComponentState::Loading => html! { <LoadingSpinner /> },
            ComponentState::Loaded(data) => html! { 
                <div>
                    <h1>{{ data.title }}</h1>
                    <p>{{ data.description }}</p>
                </div>
            },
            ComponentState::Error(msg) => html! { <ErrorDisplay message={{ msg }} /> },
        }
    }
    
    fn mounted(&mut self) {
        self.state = ComponentState::Loading;
        self.request_update();
        
        let task = orbit::tasks::spawn(async move {
            match load_data().await {
                Ok(data) => {
                    self.state = ComponentState::Loaded(data);
                },
                Err(e) => {
                    self.state = ComponentState::Error(e.to_string());
                }
            }
            self.request_update();
        });
        
        self.task = Some(task);
    }
    
    fn unmounted(&mut self) {
        // Cancel the task if still running
        if let Some(task) = self.task.take() {
            task.cancel();
        }
    }
}
```

### Parent-Child Lifecycle Coordination

When components need to coordinate with their children:

```rust
impl Component for ParentComponent {
    // ... other component methods ...
    
    fn mounted(&mut self) {
        // Create a context that children can access
        let context = Context::new(SharedData {
            theme: self.props.theme.clone(),
            user: self.props.user.clone(),
        });
        
        // Register the context for children to find
        self.register_context(context);
    }
    
    fn unmounted(&mut self) {
        // Notify children that parent is being unmounted
        self.children.iter_mut().for_each(|child| {
            child.parent_unmounting();
        });
        
        // Clear contexts
        self.clear_contexts();
    }
}
```

## Component Tree Management

The Orbit Framework manages components in a hierarchical tree structure, providing efficient management of parent-child relationships, lifecycle coordination, and update propagation.

### Component Tree Structure

The `ComponentTree` is responsible for:

1. **Component Registration**: Adding and removing components from the tree
2. **Relationship Management**: Tracking parent-child relationships between components
3. **Lifecycle Coordination**: Ensuring proper propagation of lifecycle events through the tree
4. **Update Scheduling**: Coordinating efficient updates across the tree

### Key Tree Management Features

- **Hierarchical Component Rendering**: Components render in proper dependency order
- **Efficient Updates**: Only affected components are updated
- **Parent-Child Communication**: Framework-managed parent-child relationships
- **Automatic Cleanup**: When parents are removed, children are automatically unmounted

### Component Tree API

```rust
// Create a component tree with application context
let tree = ComponentTree::new(context);

// Add a component to the tree
tree.add_component(component_instance)?;

// Set a component as the root of the tree
tree.set_root(component_id)?;

// Add child to parent
tree.add_child(parent_id, child_id)?;

// Mount a component in the tree
tree.mount_component(component_id)?;

// Get the lifecycle manager for a component
let lifecycle_manager = tree.get_lifecycle_manager(component_id)?;

// Unmount and remove a component
tree.remove_component(component_id)?;
```

### State Change Detection

Components in the tree have automatic state change detection:

1. **State Tracking**: The framework tracks component state automatically
2. **Batched Updates**: Multiple state changes are batched for efficiency
3. **Propagation Control**: Updates can be controlled with change priorities

```rust
// Component code triggering an update
fn increment_counter(&mut self) {
    self.counter += 1;
    // Framework automatically detects the state change
    // and schedules an update with default priority
}

// Explicitly requesting an update
fn force_update(&mut self) -> Result<(), ComponentError> {
    self.request_update()
}
```

The component tree ensures that updates are processed efficiently and in the correct order, with parent components being updated before their children.
