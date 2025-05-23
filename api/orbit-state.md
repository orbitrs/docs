# orbit::state API Reference

The `orbit::state` module provides reactive state management capabilities for Orbit components, enabling them to respond to data changes and update efficiently.

## Overview

State management in Orbit revolves around several key abstractions:

1. `State<T>` - A reactive container for a single value
2. `Store<S, A>` - A Redux-inspired state container with actions and reducers
3. `StateHook` - Hooks for using state in functional components
4. `Computed` - Derived state that updates when dependencies change

These abstractions provide a flexible system for managing application state at different scales, from simple component-local state to complex application-wide state management.

## Core Types

### State

The `State<T>` type is a reactive container for a single value:

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
        self.subscribers.retain(|s| s.id != id);
    }
    
    /// Update the value using a function
    pub fn update<F>(&mut self, updater: F)
    where
        F: FnOnce(&mut T),
        T: Clone,
    {
        let mut new_value = self.value.clone();
        updater(&mut new_value);
        self.set(new_value);
    }
    
    /// Map this state to a new state with a transformation function
    pub fn map<U, F>(&self, mapper: F) -> State<U>
    where
        F: Fn(&T) -> U + 'static,
        U: Clone,
    {
        let initial = mapper(&self.value);
        let mut mapped = State::new(initial);
        
        let mapper_clone = Box::new(mapper);
        self.subscribe(move |value| {
            let new_value = mapper_clone(value);
            mapped.set(new_value);
        });
        
        mapped
    }
}
```

### Store

The `Store<S, A>` type provides Redux-inspired state management with actions and reducers:

```rust
pub trait Action: Clone + 'static {}
pub trait Reducer<S, A: Action>: Fn(&mut S, &A) + 'static {}

impl<S, A: Action, F: Fn(&mut S, &A) + 'static> Reducer<S, A> for F {}

pub struct Store<S, A: Action> {
    state: S,
    reducer: Box<dyn Reducer<S, A>>,
    subscribers: Vec<Box<dyn Fn(&S)>>,
    middleware: Vec<Box<dyn Fn(&S, &A, &mut dyn FnMut(A))>>,
}

impl<S: Clone + 'static, A: Action> Store<S, A> {
    /// Create a new store with initial state and reducer
    pub fn new<R: Reducer<S, A>>(initial_state: S, reducer: R) -> Self {
        Self {
            state: initial_state,
            reducer: Box::new(reducer),
            subscribers: Vec::new(),
            middleware: Vec::new(),
        }
    }
    
    /// Get the current state
    pub fn get_state(&self) -> &S {
        &self.state
    }
    
    /// Dispatch an action to update the state
    pub fn dispatch(&mut self, action: A) {
        // Process through middleware first
        let mut final_action = Some(action);
        
        for middleware in &self.middleware {
            let current_action = final_action.take();
            if let Some(action) = current_action {
                middleware(&self.state, &action, &mut |a| {
                    final_action = Some(a);
                });
            } else {
                break;
            }
        }
        
        // Apply the action if it made it through middleware
        if let Some(action) = final_action {
            (self.reducer)(&mut self.state, &action);
            self.notify_subscribers();
        }
    }
    
    /// Subscribe to state changes
    pub fn subscribe<F>(&mut self, callback: F) -> SubscriptionId
    where
        F: Fn(&S) + 'static,
    {
        let id = SubscriptionId::new();
        self.subscribers.push(Box::new(callback));
        id
    }
    
    /// Add middleware to the store
    pub fn add_middleware<F>(&mut self, middleware: F)
    where
        F: Fn(&S, &A, &mut dyn FnMut(A)) + 'static,
    {
        self.middleware.push(Box::new(middleware));
    }
    
    /// Create a selector that derives data from the state
    pub fn select<T, F>(&self, selector: F) -> Computed<T>
    where
        F: Fn(&S) -> T + 'static,
        T: Clone + PartialEq + 'static,
    {
        let initial = selector(&self.state);
        let computed = Computed::new(initial);
        
        let selector_clone = Box::new(selector);
        let computed_clone = computed.clone();
        
        self.subscribe(move |state| {
            let new_value = selector_clone(state);
            computed_clone.set_if_changed(new_value);
        });
        
        computed
    }
}
```

### Computed

The `Computed<T>` type represents a derived state value that only updates when its value actually changes:

```rust
pub struct Computed<T: PartialEq> {
    inner: Rc<RefCell<ComputedInner<T>>>,
}

struct ComputedInner<T: PartialEq> {
    value: T,
    subscribers: Vec<Box<dyn Fn(&T)>>,
}

impl<T: Clone + PartialEq> Computed<T> {
    /// Create a new computed value
    pub fn new(initial: T) -> Self {
        Self {
            inner: Rc::new(RefCell::new(ComputedInner {
                value: initial,
                subscribers: Vec::new(),
            })),
        }
    }
    
    /// Get the current value
    pub fn get(&self) -> T {
        self.inner.borrow().value.clone()
    }
    
    /// Set the value if it has changed
    pub fn set_if_changed(&self, new_value: T) {
        let mut inner = self.inner.borrow_mut();
        if new_value != inner.value {
            inner.value = new_value;
            self.notify_subscribers();
        }
    }
    
    /// Subscribe to changes in the computed value
    pub fn subscribe<F>(&self, callback: F) -> SubscriptionId
    where
        F: Fn(&T) + 'static,
    {
        let mut inner = self.inner.borrow_mut();
        let id = SubscriptionId::new();
        inner.subscribers.push(Box::new(callback));
        id
    }
    
    /// Clone this computed value
    pub fn clone(&self) -> Self {
        Self {
            inner: self.inner.clone(),
        }
    }
}
```

### StateHook

The `StateHook` module provides React-like hooks for using state in components:

```rust
pub fn use_state<T: Clone + 'static>(initial: T) -> (T, impl Fn(T)) {
    // Implementation creates a state object and returns
    // a getter and setter
    let state = State::new(initial);
    let state_ref = Rc::new(RefCell::new(state));
    
    let value = state_ref.borrow().get().clone();
    
    let setter = {
        let state_ref = state_ref.clone();
        move |new_value: T| {
            state_ref.borrow_mut().set(new_value);
        }
    };
    
    (value, setter)
}

pub fn use_reducer<S: Clone + 'static, A: Action>(
    reducer: impl Reducer<S, A>,
    initial_state: S,
) -> (S, impl Fn(A)) {
    // Implementation creates a store and returns
    // the state and a dispatch function
    let store = Store::new(initial_state, reducer);
    let store_ref = Rc::new(RefCell::new(store));
    
    let state = store_ref.borrow().get_state().clone();
    
    let dispatch = {
        let store_ref = store_ref.clone();
        move |action: A| {
            store_ref.borrow_mut().dispatch(action);
        }
    };
    
    (state, dispatch)
}
```

## State Patterns

### AtomicState

An atomic piece of state that can be shared and updated:

```rust
pub struct Atom<T: Clone + 'static> {
    state: Rc<RefCell<State<T>>>,
}

impl<T: Clone + 'static> Atom<T> {
    /// Create a new atom with an initial value
    pub fn new(initial: T) -> Self {
        Self {
            state: Rc::new(RefCell::new(State::new(initial))),
        }
    }
    
    /// Get the current value
    pub fn get(&self) -> T {
        self.state.borrow().get().clone()
    }
    
    /// Set a new value
    pub fn set(&self, new_value: T) {
        self.state.borrow_mut().set(new_value);
    }
    
    /// Update the value with a function
    pub fn update<F>(&self, updater: F)
    where
        F: FnOnce(&mut T),
    {
        self.state.borrow_mut().update(updater);
    }
    
    /// Subscribe to changes
    pub fn subscribe<F>(&self, callback: F) -> SubscriptionId
    where
        F: Fn(&T) + 'static,
    {
        self.state.borrow_mut().subscribe(callback)
    }
}
```

### StateMachine

A state pattern for managing complex state transitions:

```rust
pub struct StateMachine<S: Clone + PartialEq + 'static, E: Clone + 'static> {
    current_state: State<S>,
    transitions: HashMap<S, HashMap<E, Box<dyn Fn(&S, &E) -> S>>>,
}

impl<S: Clone + PartialEq + Eq + Hash + 'static, E: Clone + 'static> StateMachine<S, E> {
    /// Create a new state machine with an initial state
    pub fn new(initial: S) -> Self {
        Self {
            current_state: State::new(initial),
            transitions: HashMap::new(),
        }
    }
    
    /// Add a state transition
    pub fn add_transition<F>(&mut self, from: S, event: E, transition: F)
    where
        F: Fn(&S, &E) -> S + 'static,
    {
        self.transitions
            .entry(from)
            .or_insert_with(HashMap::new)
            .insert(event, Box::new(transition));
    }
    
    /// Send an event to the state machine
    pub fn send(&mut self, event: E) -> bool {
        let current = self.current_state.get().clone();
        
        if let Some(state_transitions) = self.transitions.get(&current) {
            if let Some(transition) = state_transitions.get(&event) {
                let new_state = transition(&current, &event);
                self.current_state.set(new_state);
                return true;
            }
        }
        
        false
    }
    
    /// Get the current state
    pub fn state(&self) -> S {
        self.current_state.get().clone()
    }
    
    /// Subscribe to state changes
    pub fn subscribe<F>(&mut self, callback: F) -> SubscriptionId
    where
        F: Fn(&S) + 'static,
    {
        self.current_state.subscribe(callback)
    }
}
```

## Usage Examples

### Basic State

```rust
use orbit::prelude::*;
use orbit::state::State;

pub struct Counter {
    count: State<i32>,
}

impl Component for Counter {
    type Props = ();
    
    fn new(_props: Self::Props) -> Self {
        Self {
            count: State::new(0),
        }
    }
    
    fn increment(&mut self) {
        self.count.update(|count| *count += 1);
        self.request_update();
    }
    
    fn decrement(&mut self) {
        self.count.update(|count| *count -= 1);
        self.request_update();
    }
    
    fn count(&self) -> i32 {
        *self.count.get()
    }
}
```

### Using Store with Actions and Reducer

```rust
use orbit::prelude::*;
use orbit::state::{Store, Action};

// State
#[derive(Clone)]
pub struct TodoState {
    todos: Vec<Todo>,
    filter: Filter,
}

#[derive(Clone)]
pub struct Todo {
    id: usize,
    text: String,
    completed: bool,
}

#[derive(Clone, PartialEq)]
pub enum Filter {
    All,
    Active,
    Completed,
}

// Actions
#[derive(Clone)]
pub enum TodoAction {
    Add(String),
    Toggle(usize),
    Remove(usize),
    SetFilter(Filter),
    ClearCompleted,
}

impl Action for TodoAction {}

// Component
pub struct TodoApp {
    store: Store<TodoState, TodoAction>,
    next_id: usize,
}

impl Component for TodoApp {
    type Props = ();
    
    fn new(_props: Self::Props) -> Self {
        // Define reducer
        let reducer = |state: &mut TodoState, action: &TodoAction| {
            match action {
                TodoAction::Add(text) => {
                    let id = state.todos.len();
                    state.todos.push(Todo {
                        id,
                        text: text.clone(),
                        completed: false,
                    });
                },
                TodoAction::Toggle(id) => {
                    if let Some(todo) = state.todos.iter_mut().find(|t| t.id == *id) {
                        todo.completed = !todo.completed;
                    }
                },
                TodoAction::Remove(id) => {
                    state.todos.retain(|t| t.id != *id);
                },
                TodoAction::SetFilter(filter) => {
                    state.filter = filter.clone();
                },
                TodoAction::ClearCompleted => {
                    state.todos.retain(|t| !t.completed);
                },
            }
        };
        
        // Create initial state
        let initial_state = TodoState {
            todos: Vec::new(),
            filter: Filter::All,
        };
        
        // Create store
        let store = Store::new(initial_state, reducer);
        
        Self {
            store,
            next_id: 0,
        }
    }
    
    fn add_todo(&mut self, text: String) {
        self.store.dispatch(TodoAction::Add(text));
        self.next_id += 1;
    }
    
    fn toggle_todo(&mut self, id: usize) {
        self.store.dispatch(TodoAction::Toggle(id));
    }
    
    fn remove_todo(&mut self, id: usize) {
        self.store.dispatch(TodoAction::Remove(id));
    }
    
    fn set_filter(&mut self, filter: Filter) {
        self.store.dispatch(TodoAction::SetFilter(filter));
    }
    
    fn clear_completed(&mut self) {
        self.store.dispatch(TodoAction::ClearCompleted);
    }
    
    fn filtered_todos(&self) -> Vec<&Todo> {
        let state = self.store.get_state();
        match state.filter {
            Filter::All => state.todos.iter().collect(),
            Filter::Active => state.todos.iter().filter(|t| !t.completed).collect(),
            Filter::Completed => state.todos.iter().filter(|t| t.completed).collect(),
        }
    }
    
    fn mounted(&mut self) {
        // Subscribe to state changes
        self.store.subscribe(|_| {
            self.request_update();
        });
    }
}
```

### Using State Hooks

```rust
use orbit::prelude::*;
use orbit::state::use_state;

pub struct Counter {
    props: CounterProps,
    count: State<i32>,
    set_count: Box<dyn Fn(i32)>,
}

#[derive(Props)]
pub struct CounterProps {
    #[prop(default = "0")]
    pub initial: i32,
}

impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        // Create state hook
        let (count, set_count) = use_state(props.initial);
        
        Self {
            props,
            count: State::new(count),
            set_count: Box::new(set_count),
        }
    }
    
    fn increment(&mut self) {
        let current = *self.count.get();
        (self.set_count)(current + 1);
    }
    
    fn decrement(&mut self) {
        let current = *self.count.get();
        (self.set_count)(current - 1);
    }
}
```

### Using StateMachine

```rust
use orbit::prelude::*;
use orbit::state::StateMachine;

// Define states and events
#[derive(Clone, PartialEq, Eq, Hash)]
enum PlayerState {
    Idle,
    Playing,
    Paused,
    Loading,
    Error,
}

#[derive(Clone)]
enum PlayerEvent {
    Load,
    Play,
    Pause,
    Stop,
    Error,
    Loaded,
}

pub struct MediaPlayer {
    props: MediaPlayerProps,
    state_machine: StateMachine<PlayerState, PlayerEvent>,
}

#[derive(Props)]
pub struct MediaPlayerProps {
    pub src: String,
}

impl Component for MediaPlayer {
    type Props = MediaPlayerProps;
    
    fn new(props: Self::Props) -> Self {
        // Create state machine with initial state
        let mut state_machine = StateMachine::new(PlayerState::Idle);
        
        // Define transitions
        state_machine.add_transition(
            PlayerState::Idle,
            PlayerEvent::Load,
            |_, _| PlayerState::Loading
        );
        
        state_machine.add_transition(
            PlayerState::Loading,
            PlayerEvent::Loaded,
            |_, _| PlayerState::Idle
        );
        
        state_machine.add_transition(
            PlayerState::Loading, 
            PlayerEvent::Error,
            |_, _| PlayerState::Error
        );
        
        state_machine.add_transition(
            PlayerState::Idle,
            PlayerEvent::Play,
            |_, _| PlayerState::Playing
        );
        
        state_machine.add_transition(
            PlayerState::Playing,
            PlayerEvent::Pause,
            |_, _| PlayerState::Paused
        );
        
        state_machine.add_transition(
            PlayerState::Paused,
            PlayerEvent::Play,
            |_, _| PlayerState::Playing
        );
        
        state_machine.add_transition(
            PlayerState::Playing,
            PlayerEvent::Stop,
            |_, _| PlayerState::Idle
        );
        
        state_machine.add_transition(
            PlayerState::Paused,
            PlayerEvent::Stop,
            |_, _| PlayerState::Idle
        );
        
        Self {
            props,
            state_machine,
        }
    }
    
    fn mounted(&mut self) {
        // Subscribe to state changes
        self.state_machine.subscribe(|_| {
            self.request_update();
        });
        
        // Initial load
        self.state_machine.send(PlayerEvent::Load);
    }
    
    fn play(&mut self) {
        self.state_machine.send(PlayerEvent::Play);
    }
    
    fn pause(&mut self) {
        self.state_machine.send(PlayerEvent::Pause);
    }
    
    fn stop(&mut self) {
        self.state_machine.send(PlayerEvent::Stop);
    }
    
    fn is_playing(&self) -> bool {
        self.state_machine.state() == PlayerState::Playing
    }
    
    fn is_paused(&self) -> bool {
        self.state_machine.state() == PlayerState::Paused
    }
}
```

### Using Computed Values

```rust
use orbit::prelude::*;
use orbit::state::{State, Computed};

pub struct TodoList {
    todos: State<Vec<Todo>>,
    filter: State<Filter>,
    filtered_todos: Computed<Vec<Todo>>,
    stats: Computed<TodoStats>,
}

#[derive(Clone)]
pub struct Todo {
    id: usize,
    text: String,
    completed: bool,
}

#[derive(Clone, PartialEq)]
pub enum Filter {
    All,
    Active,
    Completed,
}

#[derive(Clone, PartialEq)]
pub struct TodoStats {
    total: usize,
    active: usize,
    completed: usize,
}

impl Component for TodoList {
    type Props = ();
    
    fn new(_props: Self::Props) -> Self {
        let todos = State::new(Vec::new());
        let filter = State::new(Filter::All);
        
        // Create computed value for filtered todos
        let filtered_todos = {
            let todos = todos.clone();
            let filter = filter.clone();
            
            Computed::new(Vec::new())
        };
        
        // Create computed value for stats
        let stats = {
            let todos = todos.clone();
            
            Computed::new(TodoStats {
                total: 0,
                active: 0,
                completed: 0,
            })
        };
        
        // Set up subscriptions
        {
            let filtered_todos = filtered_todos.clone();
            let filter_ref = filter.clone();
            
            todos.subscribe(move |todos| {
                let filter_value = filter_ref.get();
                let filtered = match filter_value {
                    Filter::All => todos.clone(),
                    Filter::Active => todos.iter()
                        .filter(|t| !t.completed)
                        .cloned()
                        .collect(),
                    Filter::Completed => todos.iter()
                        .filter(|t| t.completed)
                        .cloned()
                        .collect(),
                };
                
                filtered_todos.set_if_changed(filtered);
            });
        }
        
        {
            let stats = stats.clone();
            
            todos.subscribe(move |todos| {
                let completed = todos.iter().filter(|t| t.completed).count();
                let total = todos.len();
                
                stats.set_if_changed(TodoStats {
                    total,
                    active: total - completed,
                    completed,
                });
            });
        }
        
        Self {
            todos,
            filter,
            filtered_todos,
            stats,
        }
    }
    
    fn add_todo(&mut self, text: String) {
        self.todos.update(|todos| {
            let id = todos.len();
            todos.push(Todo {
                id,
                text,
                completed: false,
            });
        });
    }
    
    fn toggle_todo(&mut self, id: usize) {
        self.todos.update(|todos| {
            if let Some(todo) = todos.iter_mut().find(|t| t.id == id) {
                todo.completed = !todo.completed;
            }
        });
    }
    
    fn remove_todo(&mut self, id: usize) {
        self.todos.update(|todos| {
            todos.retain(|t| t.id != id);
        });
    }
    
    fn set_filter(&mut self, filter: Filter) {
        self.filter.set(filter);
    }
    
    fn get_todos(&self) -> Vec<Todo> {
        self.filtered_todos.get()
    }
    
    fn get_stats(&self) -> TodoStats {
        self.stats.get()
    }
}
```

## Advanced Patterns

### Middleware Implementation

```rust
use orbit::prelude::*;
use orbit::state::{Store, Action};

// Logger middleware
fn logger_middleware<S: Clone + 'static, A: Action + std::fmt::Debug>(
    state: &S,
    action: &A,
    next: &mut dyn FnMut(A),
) {
    println!("Dispatching action: {:?}", action);
    next(action.clone());
}

// Thunk middleware for async actions
fn thunk_middleware<S: Clone + 'static, A: Action>(
    state: &S,
    action: &A,
    next: &mut dyn FnMut(A),
) {
    // Check if this action is a thunk
    if let Some(thunk) = action.downcast_ref::<ThunkAction<S, A>>() {
        // Execute the thunk function
        thunk.execute(state);
    } else {
        // Pass normal actions through
        next(action.clone());
    }
}

// Thunk action type
struct ThunkAction<S, A: Action> {
    func: Box<dyn Fn(&S) + Send + 'static>,
}

impl<S, A: Action> ThunkAction<S, A> {
    fn new<F: Fn(&S) + Send + 'static>(func: F) -> Self {
        Self { func: Box::new(func) }
    }
    
    fn execute(&self, state: &S) {
        (self.func)(state);
    }
}

impl<S, A: Action> Clone for ThunkAction<S, A> {
    fn clone(&self) -> Self {
        unreachable!("Thunks should be consumed by middleware")
    }
}

// Usage example
impl TodoApp {
    fn new() -> Self {
        let store = Store::new(initial_state, reducer);
        
        // Add middleware
        store.add_middleware(logger_middleware);
        store.add_middleware(thunk_middleware);
        
        Self { store }
    }
    
    fn load_todos_async(&self) {
        // Create an async thunk
        let thunk = ThunkAction::new(|state| {
            // Perform async operation...
            // Then dispatch a regular action with the result
        });
        
        self.store.dispatch(thunk);
    }
}
```

### Selector Pattern

```rust
use orbit::prelude::*;
use orbit::state::{Store, Selector};

// Define selectors
pub struct TodoSelectors;

impl TodoSelectors {
    pub fn filtered_todos(state: &TodoState, filter: Filter) -> Vec<Todo> {
        match filter {
            Filter::All => state.todos.clone(),
            Filter::Active => state.todos.iter()
                .filter(|t| !t.completed)
                .cloned()
                .collect(),
            Filter::Completed => state.todos.iter()
                .filter(|t| t.completed)
                .cloned()
                .collect(),
        }
    }
    
    pub fn active_count(state: &TodoState) -> usize {
        state.todos.iter().filter(|t| !t.completed).count()
    }
    
    pub fn completed_count(state: &TodoState) -> usize {
        state.todos.iter().filter(|t| t.completed).count()
    }
    
    pub fn has_completed(state: &TodoState) -> bool {
        state.todos.iter().any(|t| t.completed)
    }
}

// Usage with store.select
impl TodoApp {
    fn mounted(&mut self) {
        // Create computed values with selectors
        let active_count = self.store.select(TodoSelectors::active_count);
        let has_completed = self.store.select(TodoSelectors::has_completed);
        
        // Use in rendering
        active_count.subscribe(|count| {
            // Update UI with active count
            self.request_update();
        });
    }
}
```

## Best Practices

### State Organization

- **Component-Local State**: Use `State<T>` for simple component-internal state
- **Shared State**: Use `Store<S, A>` for state shared between components
- **Derived State**: Use `Computed<T>` for values derived from other state
- **Complex State Logic**: Use `StateMachine<S, E>` for state with complex transitions

### Performance

- Use computed values to avoid recalculating derived data
- Implement proper equality checks to prevent unnecessary updates
- Use fine-grained state to minimize update scope
- Consider batch updates for multiple related changes

### Testability

- Separate state logic from components when possible
- Create pure reducer functions that are easy to test
- Use dependency injection for stores to facilitate testing
- Test state transitions independently of UI components

### Common Patterns

- **Event Sourcing**: Store events rather than state directly
- **Command Pattern**: Separate command creation from execution
- **CQRS**: Separate read and write operations on state
- **Immutable Updates**: Always create new state objects rather than mutating

## See Also

- [State Management Guide](../core-concepts/state-management.md)
- [Advanced State Patterns](../core-concepts/advanced-state-patterns.md)
- [Performance Optimization](../guides/performance-optimization.md)
