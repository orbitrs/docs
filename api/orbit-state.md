# orbit::state API Reference

The `orbit::state` module provides reactive state management capabilities for Orbit components, enabling them to respond to data changes and update efficiently.

## Overview

State management in Orbit revolves around the `State<T>` type, which wraps a value and notifies subscribers when the value changes. This enables reactive updates to component UI when data changes.

## Core Types

### State

```rust
pub struct State<T> {
    value: T,
    subscribers: Vec<Subscription<T>>,
}

impl<T: Clone> State<T> {
    /// Create a new state container with the initial value
    pub fn new(initial: T) -> Self {
        Self {
            value: initial,
            subscribers: Vec::new(),
        }
    }
    
    /// Get a reference to the current value
    pub fn get(&self) -> &T {
        &self.value
    }
    
    /// Update the value and notify all subscribers
    pub fn set(&mut self, new_value: T) {
        self.value = new_value;
        self.notify_subscribers();
    }
    
    /// Subscribe to changes in this state
    pub fn subscribe<F>(&mut self, callback: F) -> SubscriptionId 
    where 
        F: Fn(&T) + 'static 
    {
        // Implementation creates a subscription and returns its ID
        let id = SubscriptionId::new();
        self.subscribers.push(Subscription::new(id, Box::new(callback)));
        id
    }
    
    /// Unsubscribe from state changes
    pub fn unsubscribe(&mut self, id: SubscriptionId) {
        // Implementation removes the subscription with the given ID
        self.subscribers.retain(|sub| sub.id != id);
    }
    
    /// Modify the state using a function and notify subscribers
    pub fn update<F>(&mut self, updater: F) 
    where 
        F: FnOnce(&mut T),
        T: Clone
    {
        // Implementation updates the value and notifies subscribers
        updater(&mut self.value);
        self.notify_subscribers();
    }
    
    /// Apply a transformation to create a new State
    pub fn map<U: Clone, F>(&self, mapper: F) -> State<U> 
    where 
        F: Fn(&T) -> U + 'static 
    {
        // Implementation creates a derived state
        let initial = mapper(&self.value);
        State::new(initial)
    }
}
```

### SubscriptionId

```rust
/// Identifier for a state subscription
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct SubscriptionId(u64);

impl SubscriptionId {
    /// Create a new unique subscription ID
    pub fn new() -> Self {
        // Implementation generates a unique ID
        Self(generate_unique_id())
    }
}
```

### Subscription

```rust
struct Subscription<T> {
    id: SubscriptionId,
    callback: Box<dyn Fn(&T)>,
}
```

### Computed

```rust
/// A state that is computed from other states
pub struct Computed<T> {
    value: T,
    dependencies: Vec<Box<dyn AnyState>>,
    compute_fn: Box<dyn Fn() -> T>,
}

impl<T: Clone + 'static> Computed<T> {
    /// Create a new computed state
    pub fn new<F>(compute_fn: F, dependencies: Vec<Box<dyn AnyState>>) -> Self 
    where 
        F: Fn() -> T + 'static 
    {
        let initial = compute_fn();
        Self {
            value: initial,
            dependencies,
            compute_fn: Box::new(compute_fn),
        }
    }
    
    /// Get the current computed value
    pub fn get(&self) -> &T {
        &self.value
    }
    
    /// Recompute the value
    pub fn recompute(&mut self) {
        self.value = (self.compute_fn)();
    }
}
```

### StateGroup

```rust
/// A collection of related states that can be updated together
pub struct StateGroup {
    states: HashMap<String, Box<dyn AnyState>>,
}

impl StateGroup {
    /// Create a new empty state group
    pub fn new() -> Self {
        Self {
            states: HashMap::new(),
        }
    }
    
    /// Add a state to the group
    pub fn add<T: Clone + 'static>(&mut self, name: &str, state: State<T>) {
        self.states.insert(name.to_string(), Box::new(state));
    }
    
    /// Get a state by name
    pub fn get<T: Clone + 'static>(&self, name: &str) -> Option<&State<T>> {
        self.states.get(name)?.downcast_ref::<State<T>>()
    }
    
    /// Get a mutable reference to a state
    pub fn get_mut<T: Clone + 'static>(&mut self, name: &str) -> Option<&mut State<T>> {
        self.states.get_mut(name)?.downcast_mut::<State<T>>()
    }
}
```

## Advanced Features

### AnyState

```rust
/// Trait for type-erased state values
pub trait AnyState: Any {
    /// Type-erased notification
    fn notify_subscribers(&mut self);
    
    /// Get the state value as Any
    fn as_any(&self) -> &dyn Any;
    
    /// Get the mutable state value as Any
    fn as_any_mut(&mut self) -> &mut dyn Any;
}

impl<T: Clone + 'static> AnyState for State<T> {
    fn notify_subscribers(&mut self) {
        // Implementation notifies subscribers
        for sub in &self.subscribers {
            (sub.callback)(&self.value);
        }
    }
    
    fn as_any(&self) -> &dyn Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn Any {
        self
    }
}
```

### StateContext

```rust
/// A context for sharing state between components
pub struct StateContext<T: Clone + 'static> {
    state: Rc<RefCell<State<T>>>,
}

impl<T: Clone + 'static> StateContext<T> {
    /// Create a new state context with the initial value
    pub fn new(initial: T) -> Self {
        Self {
            state: Rc::new(RefCell::new(State::new(initial))),
        }
    }
    
    /// Get a clone of the current state
    pub fn get(&self) -> T {
        self.state.borrow().get().clone()
    }
    
    /// Update the state value
    pub fn set(&self, new_value: T) {
        self.state.borrow_mut().set(new_value);
    }
    
    /// Subscribe to changes in this context
    pub fn subscribe<F>(&self, callback: F) -> SubscriptionId 
    where 
        F: Fn(&T) + 'static 
    {
        self.state.borrow_mut().subscribe(callback)
    }
}
```

## Common Usage Patterns

### Basic State Management

```rust
use orbit::state::State;
use orbit::component::Component;

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
    
    fn decrement(&mut self) {
        self.count.update(|count| {
            if *count > 0 {
                *count -= 1;
            }
        });
        self.request_update();
    }
}
```

### Derived State

```rust
use orbit::state::{State, Computed};

pub struct FormState {
    username: State<String>,
    password: State<String>,
    is_valid: Computed<bool>,
}

impl FormState {
    pub fn new() -> Self {
        let username = State::new(String::new());
        let password = State::new(String::new());
        
        // Create refs to capture the states
        let username_ref = username.clone();
        let password_ref = password.clone();
        
        // Create computed state based on other states
        let is_valid = Computed::new(
            move || {
                let username = username_ref.get();
                let password = password_ref.get();
                !username.is_empty() && password.len() >= 8
            },
            vec![Box::new(username.clone()), Box::new(password.clone())]
        );
        
        Self {
            username,
            password,
            is_valid,
        }
    }
    
    pub fn set_username(&mut self, username: String) {
        self.username.set(username);
        self.is_valid.recompute();
    }
    
    pub fn set_password(&mut self, password: String) {
        self.password.set(password);
        self.is_valid.recompute();
    }
    
    pub fn is_valid(&self) -> bool {
        *self.is_valid.get()
    }
}
```

### Shared State Between Components

```rust
use std::rc::Rc;
use std::cell::RefCell;
use orbit::state::StateContext;

// Define a global app state
struct AppState {
    user: Option<User>,
    theme: Theme,
    notifications: Vec<Notification>,
}

// Create a shared context
let app_state = StateContext::new(AppState {
    user: None,
    theme: Theme::Light,
    notifications: Vec::new(),
});

// In a component
pub struct Header {
    props: HeaderProps,
    app_state: Rc<StateContext<AppState>>,
    subscription: Option<SubscriptionId>,
}

impl Component for Header {
    type Props = HeaderProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            props,
            app_state: app_state.clone(),
            subscription: None,
        }
    }
    
    fn mounted(&mut self) {
        // Subscribe to state changes
        let subscription = self.app_state.subscribe(|_| {
            self.request_update();
        });
        
        self.subscription = Some(subscription);
    }
    
    fn unmounted(&mut self) {
        // Clean up subscription
        if let Some(id) = self.subscription.take() {
            self.app_state.unsubscribe(id);
        }
    }
}
```

### State Machine

```rust
use orbit::state::State;

// Define states
enum FormStatus {
    Editing,
    Submitting,
    Success,
    Error(String),
}

pub struct FormStateMachine {
    status: State<FormStatus>,
    data: State<FormData>,
}

impl FormStateMachine {
    pub fn new() -> Self {
        Self {
            status: State::new(FormStatus::Editing),
            data: State::new(FormData::default()),
        }
    }
    
    pub fn submit(&mut self) {
        // Only allow submission from editing state
        if let FormStatus::Editing = *self.status.get() {
            self.status.set(FormStatus::Submitting);
            
            // In a real app, this would be an async operation
            let data = self.data.get().clone();
            if self.validate_data(&data) {
                self.status.set(FormStatus::Success);
            } else {
                self.status.set(FormStatus::Error("Validation failed".to_string()));
            }
        }
    }
    
    pub fn reset(&mut self) {
        self.status.set(FormStatus::Editing);
        self.data.set(FormData::default());
    }
    
    fn validate_data(&self, data: &FormData) -> bool {
        // Validation logic
        true
    }
}
```

## Best Practices

### 1. Keep State Minimal

Only track what's needed for rendering or business logic. Derived values should be computed rather than stored as separate state.

### 2. Group Related State

Use `StateGroup` or create custom state containers to group related state values. This makes the code more maintainable and reduces the chance of inconsistent state.

### 3. Clean Up Subscriptions

Always unsubscribe from state changes when components are unmounted to prevent memory leaks and unexpected behavior.

### 4. Use Computed State for Derived Values

Instead of manually updating derived state, use `Computed` to automatically derive values from other state.

### 5. State Immutability

Treat state values as immutable. Use `set` and `update` methods to create new state rather than mutating the underlying value directly.

## Performance Considerations

### Batching Updates

When making multiple state changes that should trigger a single update:

```rust
pub fn update_profile(&mut self) {
    // State updates are automatically batched in the next microtask
    self.profile.name.set(new_name.clone());
    self.profile.email.set(new_email.clone());
    self.profile.avatar.set(new_avatar.clone());
    
    // Only request a single update
    self.request_update();
}
```

### Memoization

For expensive computations, consider memoizing results:

```rust
pub struct FilteredList {
    items: State<Vec<Item>>,
    filter: State<Filter>,
    filtered_items_cache: State<Option<Vec<Item>>>,
}

impl FilteredList {
    pub fn filtered_items(&mut self) -> Vec<Item> {
        // Check if cache needs to be invalidated
        if self.filtered_items_cache.get().is_none() {
            let items = self.items.get().clone();
            let filter = self.filter.get().clone();
            
            let filtered = items.into_iter()
                .filter(|item| filter.matches(item))
                .collect();
                
            self.filtered_items_cache.set(Some(filtered));
        }
        
        self.filtered_items_cache.get().clone().unwrap_or_default()
    }
    
    pub fn invalidate_cache(&mut self) {
        self.filtered_items_cache.set(None);
    }
}
```

## Related Documentation

- [State Management](../core-concepts/state-management.md) - High-level overview of state management approaches
- [Advanced State Patterns](../core-concepts/advanced-component-patterns.md#state-management) - Advanced state management techniques
