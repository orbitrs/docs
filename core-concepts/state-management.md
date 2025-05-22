# State Management in Orbit

This guide explores the various approaches to state management in the Orbit Framework, from simple component state to complex application-wide state management.

## Overview

Effective state management is crucial for building reactive, maintainable UI applications. Orbit provides several state management approaches that can be used independently or combined based on your application's complexity:

1. **Component State** - Local state within a component
2. **Props System** - Parent-to-child data flow
3. **Context API** - State sharing without prop drilling
4. **Stores** - Application-wide state management
5. **Signals** - Fine-grained reactivity system

## Component State

The simplest form of state management is component-local state. This is appropriate for state that doesn't need to be shared outside the component.

### Defining Component State

```rust
pub struct Counter {
    // Component state
    count: i32,
    is_active: bool,
}

impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            count: props.initial_count.unwrap_or(0),
            is_active: false,
        }
    }
}
```

### Updating Component State

```rust
impl Counter {
    fn increment(&mut self) {
        self.count += 1;
        // The framework automatically detects state changes
        // and triggers a re-render
    }
    
    fn toggle_active(&mut self) {
        self.is_active = !self.is_active;
    }
}
```

### When to Use Component State

- UI state that's specific to a component (open/closed, active tab, etc.)
- Temporary form input values
- Animation states
- Any state that doesn't need to be accessed by other components

## Props System

Props allow you to pass data from parent to child components, creating a one-way data flow.

### Defining Props

```rust
pub struct ButtonProps {
    pub label: String,
    pub color: Option<String>,
    pub on_click: Option<Callback<ClickEvent>>,
}

impl Props for ButtonProps {}

impl Default for ButtonProps {
    fn default() -> Self {
        Self {
            label: "Button".to_string(),
            color: None,
            on_click: None,
        }
    }
}
```

### Passing Props

```orbit
<template>
  <Button
    label="Save Changes"
    color="primary"
    @click="handleSave"
  />
</template>
```

### Using Props in Components

```rust
impl Component for Button {
    type Props = ButtonProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            label: props.label,
            color: props.color.unwrap_or_else(|| "default".to_string()),
            on_click: props.on_click,
        }
    }
    
    // Rest of implementation
}
```

### Callbacks for Child-to-Parent Communication

```rust
// In child component
fn trigger_click(&mut self) {
    if let Some(on_click) = &self.on_click {
        on_click.call(ClickEvent::new());
    }
}

// In parent template
<template>
  <Button @click="handleButtonClick" />
</template>

// In parent implementation
fn handle_button_click(&mut self, _event: ClickEvent) {
    // Handle the click event from the child
    self.save_data();
}
```

### When to Use Props

- Passing data to child components
- Configuring reusable components
- Creating component composition patterns
- Handling events from child to parent

## Context API

Context provides a way to share values between components without passing props through every level of the component tree. This is useful for themes, user authentication, localization, and other application-wide concerns.

### Creating a Context

```rust
// Define context value type
pub struct ThemeContext {
    pub primary_color: String,
    pub text_color: String,
    pub is_dark_mode: bool,
}

// In your root component
pub fn create_theme_context() -> Context<ThemeContext> {
    Context::new(ThemeContext {
        primary_color: "#0070f3".to_string(),
        text_color: "#333333".to_string(),
        is_dark_mode: false,
    })
}
```

### Providing Context

```orbit
<template>
  <ContextProvider :context="theme_context">
    <App />
  </ContextProvider>
</template>

<code lang="rust">
use orbit::prelude::*;
use crate::theme::ThemeContext;

pub struct Root {
    theme_context: Context<ThemeContext>,
}

impl Component for Root {
    // Implementation details
    
    fn mounted(&mut self) {
        self.theme_context = create_theme_context();
    }
}
</code>
```

### Consuming Context

```orbit
<template>
  <div :style="{ color: theme.text_color, backgroundColor: theme.primary_color }">
    <h1>Themed Component</h1>
    <p>This component uses the theme context values.</p>
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;
use crate::theme::ThemeContext;

pub struct ThemedComponent {
    theme: Option<ThemeContext>,
}

impl Component for ThemedComponent {
    // Implementation details
    
    fn mounted(&mut self) {
        // Subscribe to context updates
        self.theme = context::<ThemeContext>();
    }
}
</script>
```

### Updating Context

```rust
// In a component that needs to update the context
fn toggle_theme(&mut self) {
    // Get the current context
    let mut theme = context_mut::<ThemeContext>();
    
    // Update the context value
    theme.is_dark_mode = !theme.is_dark_mode;
    
    if theme.is_dark_mode {
        theme.primary_color = "#2d3748".to_string();
        theme.text_color = "#f7fafc".to_string();
    } else {
        theme.primary_color = "#0070f3".to_string();
        theme.text_color = "#333333".to_string();
    }
    
    // The framework will notify all context subscribers of the change
}
```

### When to Use Context

- Theme settings
- User authentication state
- Localization preferences
- Routing information
- Feature flags
- Any state that needs to be accessed by many components at different nesting levels

## Stores

For complex applications with many state interdependencies, Orbit provides a Store pattern for centralized state management.

### Creating a Store

```rust
// Define your store state
#[derive(Clone, Default)]
pub struct AppState {
    pub user: Option<User>,
    pub todos: Vec<Todo>,
    pub filter: TodoFilter,
    pub is_loading: bool,
}

// Create a store
pub struct AppStore {
    state: AppState,
    subscribers: Vec<Box<dyn Fn(&AppState)>>,
}

impl AppStore {
    pub fn new() -> Self {
        Self {
            state: AppState::default(),
            subscribers: Vec::new(),
        }
    }
    
    pub fn get_state(&self) -> &AppState {
        &self.state
    }
    
    pub fn update<F>(&mut self, updater: F)
    where
        F: FnOnce(&mut AppState),
    {
        updater(&mut self.state);
        self.notify_subscribers();
    }
    
    pub fn subscribe<F>(&mut self, subscriber: F) -> SubscriptionId
    where
        F: Fn(&AppState) + 'static,
    {
        let id = SubscriptionId(self.subscribers.len());
        self.subscribers.push(Box::new(subscriber));
        id
    }
    
    fn notify_subscribers(&self) {
        for subscriber in &self.subscribers {
            subscriber(&self.state);
        }
    }
}
```

### Creating a Global Store Instance

```rust
// In a module like store.rs
use std::sync::Mutex;
use lazy_static::lazy_static;

lazy_static! {
    static ref STORE: Mutex<AppStore> = Mutex::new(AppStore::new());
}

pub fn get_store() -> &'static Mutex<AppStore> {
    &STORE
}
```

### Using the Store in Components

```rust
impl Component for TodoList {
    // Implementation details
    
    fn mounted(&mut self) {
        // Subscribe to store updates
        let store = get_store();
        let component_id = self.id.clone();
        
        store.lock().unwrap().subscribe(move |state| {
            // Update component with new state
            orbit::update_component(component_id.clone(), |component: &mut TodoList| {
                component.todos = state.todos.clone();
                component.filter = state.filter;
            });
        });
        
        // Initial state
        let state = store.lock().unwrap().get_state().clone();
        self.todos = state.todos;
        self.filter = state.filter;
    }
}

impl TodoList {
    fn add_todo(&mut self, title: String) {
        let store = get_store();
        store.lock().unwrap().update(|state| {
            let todo = Todo::new(title);
            state.todos.push(todo);
        });
    }
    
    fn toggle_todo(&mut self, id: u64) {
        let store = get_store();
        store.lock().unwrap().update(|state| {
            if let Some(todo) = state.todos.iter_mut().find(|t| t.id == id) {
                todo.completed = !todo.completed;
            }
        });
    }
}
```

### When to Use Stores

- Complex applications with many components sharing state
- When you need a single source of truth for application state
- When state updates need to trigger changes across the component tree
- For separating state management logic from UI components

## Signals

For fine-grained reactivity, Orbit provides a signals system inspired by reactive programming.

### Creating Signals

```rust
// In a component or shared module
let count = signal(0);
let double_count = derived(count, |c| c * 2);
```

### Using Signals in Components

```rust
pub struct SignalCounter {
    count: Signal<i32>,
    double_count: Derived<i32>,
}

impl Component for SignalCounter {
    type Props = SignalCounterProps;
    
    fn new(props: Self::Props) -> Self {
        let count = signal(props.initial.unwrap_or(0));
        let double_count = derived(count, |c| c * 2);
        
        Self {
            count,
            double_count,
        }
    }
}

impl SignalCounter {
    fn increment(&mut self) {
        self.count.update(|c| c + 1);
        // No need to manually trigger updates - signals handle this
    }
}
```

### Signal-Powered Templates

```orbit
<template>
  <div>
    <p>Count: {{ count.get() }}</p>
    <p>Double: {{ double_count.get() }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>
```

### Creating Effects

```rust
fn mounted(&mut self) {
    effect(self.count, |count| {
        println!("Count changed to: {}", count);
        // Perform side effects when count changes
        
        // Optionally return a cleanup function
        || {
            println!("Cleaning up previous effect");
        }
    });
}
```

### When to Use Signals

- When you need fine-grained reactivity
- For computed/derived values that depend on other state
- To minimize re-renders by only updating what changed
- For complex interdependent state that would be hard to manage with plain component state

## Combining Approaches

Most real-world applications will use a combination of these state management approaches:

- **Component State** for UI-specific, non-shared state
- **Props** for component composition and configuration
- **Context** for theme, auth, and other widely-used data
- **Stores** for complex application state
- **Signals** for fine-grained reactivity

### Example: Todo Application Architecture

```
App (Store Provider)
├── Header
│   └── UserBadge (reads user from Context)
├── TodoInput (dispatches actions to Store)
├── FilterBar (dispatches filter actions to Store)
└── TodoList (subscribes to Store)
    └── TodoItem (dispatches toggle/delete actions to Store)
        └── DeleteButton (uses Component State for confirmation dialog)
```

## Best Practices

1. **Start Simple**
   - Begin with component state and props
   - Only introduce more complex patterns when needed

2. **State Colocation**
   - Keep state as close as possible to where it's used
   - Lift state up only when it needs to be shared

3. **Immutable Updates**
   - Treat state as immutable
   - Create new objects/arrays instead of mutating existing ones

4. **Single Source of Truth**
   - Avoid duplicating state across components
   - Derive secondary state from primary state

5. **Selective Updates**
   - Use memoization and signals to prevent unnecessary renders
   - Subscribe only to the parts of state you need

6. **Clear Ownership**
   - Make it clear which component or store owns each piece of state
   - Document state management patterns in complex applications

7. **Consistent Patterns**
   - Use consistent approaches throughout your application
   - Document your state management strategy for team alignment

## Performance Considerations

1. **Granular State**
   - Split large state objects into smaller pieces
   - Allow components to subscribe to specific parts

2. **Memoization**
   - Use derived signals or memoized selectors for computed values
   - Prevent unnecessary recalculations

3. **Batch Updates**
   - Combine multiple state updates into a single update when possible
   - Use transactions for related state changes

4. **Virtual Lists**
   - For large lists, use virtualization
   - Only render visible items

5. **Selective Rendering**
   - Use `#[memo]` attribute for components that don't need frequent updates
   - Implement `should_update` for custom update control

## Debugging State

1. **State Inspection**
   - Use dev tools to inspect component state
   - Add debug logging for state changes

2. **Time-Travel Debugging**
   - Enable state history for stores
   - Implement undo/redo capabilities

3. **State Snapshots**
   - Add functions to export/import state
   - Create serializable state representations

## Conclusion

Orbit provides a flexible state management system that scales from simple component state to complex application-wide state. By understanding the strengths and use cases of each approach, you can choose the right tools for your specific needs, creating maintainable and performant applications.

Remember that the best state management solution is often the simplest one that meets your requirements. Start with local component state and introduce more complex patterns only as your application's needs grow.
