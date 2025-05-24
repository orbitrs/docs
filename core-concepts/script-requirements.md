# Script Requirements in Orbit Components

This document provides comprehensive guidance on the script section requirements in `.orbit` files, including patterns, best practices, and extensive examples for implementing component logic.

## Overview

The script section in `.orbit` files contains the Rust implementation of component logic. Every component must follow specific patterns and implement required traits to integrate with the Orbit framework.

## Basic Structure Requirements

The script section must include these mandatory elements:

### 1. Component Struct Definition

```rust
use orbit::prelude::*;

pub struct MyComponent {
    // Component state fields
    count: State<i32>,
    name: String,
    context: Context,
}
```

### 2. Props Struct Definition

```rust
#[derive(Debug, Clone)]
pub struct MyComponentProps {
    pub initial_count: i32,
    pub label: String,
    pub disabled: bool,
}

impl Props for MyComponentProps {}
```

### 3. Component Trait Implementation

```rust
impl Component for MyComponent {
    type Props = MyComponentProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        Self {
            count: State::new(props.initial_count),
            name: props.label,
            context,
        }
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        // Implementation required
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

## Component Patterns

### 1. Simple Stateless Component

```rust
use orbit::prelude::*;

// Minimal component with only props
pub struct ButtonComponent {
    context: Context,
    label: String,
    disabled: bool,
}

#[derive(Debug, Clone)]
pub struct ButtonProps {
    pub label: String,
    pub disabled: bool,
}

impl Props for ButtonProps {}

impl Component for ButtonComponent {
    type Props = ButtonProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        Self {
            context,
            label: props.label,
            disabled: props.disabled,
        }
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        // Render implementation
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

### 2. Stateful Component with State Management

```rust
use orbit::prelude::*;

pub struct CounterComponent {
    context: Context,
    count: State<i32>,
    step: i32,
    max_count: Option<i32>,
}

#[derive(Debug, Clone)]
pub struct CounterProps {
    pub initial_count: i32,
    pub step: i32,
    pub max_count: Option<i32>,
}

impl Props for CounterProps {}

impl Component for CounterComponent {
    type Props = CounterProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        Self {
            context,
            count: State::new(props.initial_count),
            step: props.step,
            max_count: props.max_count,
        }
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        // Render with state
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}

impl CounterComponent {
    pub fn increment(&self) {
        let current = *self.count.get();
        if let Some(max) = self.max_count {
            if current + self.step <= max {
                self.count.set(current + self.step);
            }
        } else {
            self.count.set(current + self.step);
        }
    }
    
    pub fn decrement(&self) {
        let current = *self.count.get();
        self.count.set(current - self.step);
    }
    
    pub fn reset(&self) {
        self.count.set(0);
    }
}
```

### 3. Component with Lifecycle Methods

```rust
use orbit::prelude::*;

pub struct TimerComponent {
    context: Context,
    elapsed: State<u64>,
    interval: u64,
    timer_id: Option<TimerId>,
}

#[derive(Debug, Clone)]
pub struct TimerProps {
    pub interval_ms: u64,
    pub auto_start: bool,
}

impl Props for TimerProps {}

impl Component for TimerComponent {
    type Props = TimerProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        Self {
            context,
            elapsed: State::new(0),
            interval: props.interval_ms,
            timer_id: None,
        }
    }
    
    fn mount(&mut self) -> Result<(), ComponentError> {
        // Start timer when component mounts
        self.start_timer();
        Ok(())
    }
    
    fn unmount(&mut self) -> Result<(), ComponentError> {
        // Clean up timer when component unmounts
        self.stop_timer();
        Ok(())
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}

impl TimerComponent {
    fn start_timer(&mut self) {
        // Timer implementation
    }
    
    fn stop_timer(&mut self) {
        // Timer cleanup
    }
}
```

### 4. Component with Event Handling

```rust
use orbit::prelude::*;

pub struct FormComponent {
    context: Context,
    username: State<String>,
    email: State<String>,
    is_valid: State<bool>,
    submit_count: State<u32>,
}

#[derive(Debug, Clone)]
pub struct FormProps {
    pub validation_enabled: bool,
    pub max_attempts: u32,
}

impl Props for FormProps {}

impl Component for FormComponent {
    type Props = FormProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        Self {
            context,
            username: State::new(String::new()),
            email: State::new(String::new()),
            is_valid: State::new(false),
            submit_count: State::new(0),
        }
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}

impl FormComponent {
    pub fn handle_username_change(&self, value: String) {
        self.username.set(value);
        self.validate_form();
    }
    
    pub fn handle_email_change(&self, value: String) {
        self.email.set(value);
        self.validate_form();
    }
    
    pub fn handle_submit(&self) -> Result<(), String> {
        if !*self.is_valid.get() {
            return Err("Form is not valid".to_string());
        }
        
        let current_count = *self.submit_count.get();
        self.submit_count.set(current_count + 1);
        
        // Submit logic here
        Ok(())
    }
    
    fn validate_form(&self) {
        let username_valid = !self.username.get().is_empty();
        let email_valid = self.email.get().contains('@');
        
        self.is_valid.set(username_valid && email_valid);
    }
}
```

### 5. Component with Context and Dependency Injection

```rust
use orbit::prelude::*;

pub struct UserProfileComponent {
    context: Context,
    user_data: State<Option<User>>,
    loading: State<bool>,
    error: State<Option<String>>,
    api_service: Box<dyn ApiService>,
}

#[derive(Debug, Clone)]
pub struct UserProfileProps {
    pub user_id: String,
    pub show_avatar: bool,
}

impl Props for UserProfileProps {}

impl Component for UserProfileComponent {
    type Props = UserProfileProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        // Get API service from context or create default
        let api_service = context
            .get_service::<Box<dyn ApiService>>()
            .unwrap_or_else(|| Box::new(DefaultApiService::new()));
            
        Self {
            context,
            user_data: State::new(None),
            loading: State::new(false),
            error: State::new(None),
            api_service,
        }
    }
    
    fn mount(&mut self) -> Result<(), ComponentError> {
        // Load user data when component mounts
        self.load_user_data();
        Ok(())
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}

impl UserProfileComponent {
    fn load_user_data(&self) {
        self.loading.set(true);
        self.error.set(None);
        
        // Async data loading would go here
        // self.api_service.get_user(&user_id).await
    }
}

// Supporting types
trait ApiService: Send + Sync {
    fn get_user(&self, id: &str) -> Result<User, String>;
}

struct DefaultApiService;

impl DefaultApiService {
    fn new() -> Self {
        Self
    }
}

impl ApiService for DefaultApiService {
    fn get_user(&self, _id: &str) -> Result<User, String> {
        // Mock implementation
        Ok(User {
            id: "123".to_string(),
            name: "John Doe".to_string(),
            email: "john@example.com".to_string(),
        })
    }
}

#[derive(Debug, Clone)]
struct User {
    id: String,
    name: String,
    email: String,
}
```

### 6. Component with Custom Hooks

```rust
use orbit::prelude::*;

pub struct SearchComponent {
    context: Context,
    search_hook: SearchHook,
    results_hook: PaginationHook<SearchResult>,
}

#[derive(Debug, Clone)]
pub struct SearchProps {
    pub placeholder: String,
    pub debounce_ms: u64,
    pub page_size: usize,
}

impl Props for SearchProps {}

impl Component for SearchComponent {
    type Props = SearchProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        let search_hook = SearchHook::new(props.debounce_ms);
        let results_hook = PaginationHook::new(props.page_size);
        
        Self {
            context,
            search_hook,
            results_hook,
        }
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}

impl SearchComponent {
    pub fn handle_search_input(&self, value: String) {
        self.search_hook.set_query(value);
    }
    
    pub fn handle_page_change(&self, page: usize) {
        self.results_hook.set_page(page);
    }
}

// Custom hooks
struct SearchHook {
    query: State<String>,
    debounce_ms: u64,
    last_search: State<Option<String>>,
}

impl SearchHook {
    fn new(debounce_ms: u64) -> Self {
        Self {
            query: State::new(String::new()),
            debounce_ms,
            last_search: State::new(None),
        }
    }
    
    fn set_query(&self, query: String) {
        self.query.set(query);
        // Debouncing logic would go here
    }
}

struct PaginationHook<T> {
    items: State<Vec<T>>,
    current_page: State<usize>,
    page_size: usize,
    total_items: State<usize>,
}

impl<T> PaginationHook<T> {
    fn new(page_size: usize) -> Self {
        Self {
            items: State::new(vec![]),
            current_page: State::new(0),
            page_size,
            total_items: State::new(0),
        }
    }
    
    fn set_page(&self, page: usize) {
        self.current_page.set(page);
    }
}

#[derive(Debug, Clone)]
struct SearchResult {
    id: String,
    title: String,
    description: String,
}
```

## Advanced Patterns

### 1. Component with Async Operations

```rust
use orbit::prelude::*;
use std::future::Future;

pub struct AsyncDataComponent {
    context: Context,
    data: State<Option<ApiResponse>>,
    loading_states: LoadingStates,
    error_handler: ErrorHandler,
}

#[derive(Debug, Clone)]
pub struct AsyncDataProps {
    pub endpoint: String,
    pub auto_refresh_interval: Option<u64>,
    pub retry_attempts: u32,
}

impl Props for AsyncDataProps {}

impl Component for AsyncDataComponent {
    type Props = AsyncDataProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        Self {
            context,
            data: State::new(None),
            loading_states: LoadingStates::new(),
            error_handler: ErrorHandler::new(props.retry_attempts),
        }
    }
    
    fn mount(&mut self) -> Result<(), ComponentError> {
        // Start data fetching
        self.fetch_data();
        Ok(())
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}

impl AsyncDataComponent {
    fn fetch_data(&self) {
        self.loading_states.set_loading(true);
        
        // Async fetch implementation would go here
        // spawn_local(async move { ... });
    }
    
    fn handle_data_received(&self, data: ApiResponse) {
        self.data.set(Some(data));
        self.loading_states.set_loading(false);
        self.error_handler.reset();
    }
    
    fn handle_error(&self, error: String) {
        self.loading_states.set_loading(false);
        self.error_handler.handle_error(error);
    }
}

// Supporting types
struct LoadingStates {
    loading: State<bool>,
    refreshing: State<bool>,
}

impl LoadingStates {
    fn new() -> Self {
        Self {
            loading: State::new(false),
            refreshing: State::new(false),
        }
    }
    
    fn set_loading(&self, loading: bool) {
        self.loading.set(loading);
    }
}

struct ErrorHandler {
    current_error: State<Option<String>>,
    retry_count: State<u32>,
    max_retries: u32,
}

impl ErrorHandler {
    fn new(max_retries: u32) -> Self {
        Self {
            current_error: State::new(None),
            retry_count: State::new(0),
            max_retries,
        }
    }
    
    fn handle_error(&self, error: String) {
        self.current_error.set(Some(error));
        let current = *self.retry_count.get();
        self.retry_count.set(current + 1);
    }
    
    fn reset(&self) {
        self.current_error.set(None);
        self.retry_count.set(0);
    }
    
    fn can_retry(&self) -> bool {
        *self.retry_count.get() < self.max_retries
    }
}

#[derive(Debug, Clone)]
struct ApiResponse {
    data: String,
    timestamp: u64,
}
```

### 2. Component with Complex State Management

```rust
use orbit::prelude::*;

pub struct TodoListComponent {
    context: Context,
    todos: State<Vec<Todo>>,
    filter: State<TodoFilter>,
    stats: TodoStats,
    undo_stack: UndoStack<TodoAction>,
}

#[derive(Debug, Clone)]
pub struct TodoListProps {
    pub initial_todos: Vec<Todo>,
    pub enable_undo: bool,
    pub max_undo_steps: usize,
}

impl Props for TodoListProps {}

impl Component for TodoListComponent {
    type Props = TodoListProps;
    
    fn create(props: Self::Props, context: Context) -> Self {
        let todos = State::new(props.initial_todos.clone());
        let stats = TodoStats::new(todos.clone());
        
        Self {
            context,
            todos,
            filter: State::new(TodoFilter::All),
            stats,
            undo_stack: UndoStack::new(props.max_undo_steps),
        }
    }
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        Ok(vec![])
    }
    
    fn as_any(&self) -> &dyn std::any::Any {
        self
    }
    
    fn as_any_mut(&mut self) -> &mut dyn std::any::Any {
        self
    }
}

impl TodoListComponent {
    pub fn add_todo(&self, text: String) {
        let new_todo = Todo::new(text);
        let action = TodoAction::Add(new_todo.clone());
        
        self.todos.update(|todos| {
            todos.push(new_todo);
        });
        
        self.undo_stack.push(action);
        self.stats.update();
    }
    
    pub fn toggle_todo(&self, id: String) {
        let mut old_todo = None;
        
        self.todos.update(|todos| {
            if let Some(todo) = todos.iter_mut().find(|t| t.id == id) {
                old_todo = Some(todo.clone());
                todo.completed = !todo.completed;
            }
        });
        
        if let Some(todo) = old_todo {
            let action = TodoAction::Toggle(todo);
            self.undo_stack.push(action);
        }
        
        self.stats.update();
    }
    
    pub fn delete_todo(&self, id: String) {
        let mut deleted_todo = None;
        let mut deleted_index = None;
        
        self.todos.update(|todos| {
            if let Some(index) = todos.iter().position(|t| t.id == id) {
                deleted_todo = Some(todos.remove(index));
                deleted_index = Some(index);
            }
        });
        
        if let (Some(todo), Some(index)) = (deleted_todo, deleted_index) {
            let action = TodoAction::Delete(todo, index);
            self.undo_stack.push(action);
        }
        
        self.stats.update();
    }
    
    pub fn undo(&self) {
        if let Some(action) = self.undo_stack.pop() {
            match action {
                TodoAction::Add(todo) => {
                    self.todos.update(|todos| {
                        todos.retain(|t| t.id != todo.id);
                    });
                }
                TodoAction::Delete(todo, index) => {
                    self.todos.update(|todos| {
                        todos.insert(index, todo);
                    });
                }
                TodoAction::Toggle(old_todo) => {
                    self.todos.update(|todos| {
                        if let Some(todo) = todos.iter_mut().find(|t| t.id == old_todo.id) {
                            todo.completed = old_todo.completed;
                        }
                    });
                }
            }
            self.stats.update();
        }
    }
    
    pub fn set_filter(&self, filter: TodoFilter) {
        self.filter.set(filter);
    }
    
    pub fn get_filtered_todos(&self) -> Vec<Todo> {
        let todos = self.todos.get().clone();
        match *self.filter.get() {
            TodoFilter::All => todos,
            TodoFilter::Active => todos.into_iter().filter(|t| !t.completed).collect(),
            TodoFilter::Completed => todos.into_iter().filter(|t| t.completed).collect(),
        }
    }
}

// Supporting types
#[derive(Debug, Clone)]
struct Todo {
    id: String,
    text: String,
    completed: bool,
    created_at: u64,
}

impl Todo {
    fn new(text: String) -> Self {
        Self {
            id: uuid::Uuid::new_v4().to_string(),
            text,
            completed: false,
            created_at: std::time::SystemTime::now()
                .duration_since(std::time::UNIX_EPOCH)
                .unwrap()
                .as_secs(),
        }
    }
}

#[derive(Debug, Clone, Copy)]
enum TodoFilter {
    All,
    Active,
    Completed,
}

#[derive(Debug, Clone)]
enum TodoAction {
    Add(Todo),
    Delete(Todo, usize),
    Toggle(Todo),
}

struct TodoStats {
    todos: State<Vec<Todo>>,
    total: State<usize>,
    completed: State<usize>,
    active: State<usize>,
}

impl TodoStats {
    fn new(todos: State<Vec<Todo>>) -> Self {
        Self {
            todos,
            total: State::new(0),
            completed: State::new(0),
            active: State::new(0),
        }
    }
    
    fn update(&self) {
        let todos = self.todos.get();
        let total = todos.len();
        let completed = todos.iter().filter(|t| t.completed).count();
        let active = total - completed;
        
        self.total.set(total);
        self.completed.set(completed);
        self.active.set(active);
    }
}

struct UndoStack<T> {
    stack: State<Vec<T>>,
    max_size: usize,
}

impl<T> UndoStack<T> {
    fn new(max_size: usize) -> Self {
        Self {
            stack: State::new(vec![]),
            max_size,
        }
    }
    
    fn push(&self, item: T) {
        self.stack.update(|stack| {
            stack.push(item);
            if stack.len() > self.max_size {
                stack.remove(0);
            }
        });
    }
    
    fn pop(&self) -> Option<T> {
        self.stack.update(|stack| stack.pop()).flatten()
    }
}
```

## Best Practices

### 1. State Management Guidelines

- **Use `State<T>` for reactive values** that should trigger re-renders
- **Use regular fields for static values** that don't change during component lifecycle
- **Minimize state complexity** by breaking large states into smaller, focused pieces
- **Use derived state** computed from primary state when possible

### 2. Props Guidelines

- **Always implement `Clone` for props** to support framework requirements
- **Use `Option<T>` for optional props** with meaningful defaults
- **Validate props** in the `create` method when necessary
- **Keep props immutable** - use state for mutable component data

### 3. Lifecycle Guidelines

- **Use `mount`** for initialization that requires DOM access or external resources
- **Use `unmount`** for cleanup (timers, subscriptions, external resources)
- **Keep lifecycle methods lightweight** and avoid heavy computation
- **Handle errors gracefully** in lifecycle methods

### 4. Performance Guidelines

- **Minimize state updates** by batching changes when possible
- **Use `should_update`** to prevent unnecessary re-renders
- **Lazy-load expensive computations** using memoization
- **Avoid creating closures in render** methods

### 5. Error Handling

```rust
impl Component for MyComponent {
    // ... other methods
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        match self.try_render() {
            Ok(nodes) => Ok(nodes),
            Err(e) => {
                // Log error and return fallback UI
                eprintln!("Render error: {}", e);
                Ok(vec![create_error_node(&e)])
            }
        }
    }
}

impl MyComponent {
    fn try_render(&self) -> Result<Vec<Node>, String> {
        // Actual render logic that might fail
        Ok(vec![])
    }
}

fn create_error_node(error: &str) -> Node {
    // Create error display node
    Node::new(None)
}
```

## Common Patterns Summary

| Pattern | Use Case | Key Features |
|---------|----------|--------------|
| **Stateless** | Display components, pure UI | Props only, no internal state |
| **Stateful** | Interactive components | Local state management with `State<T>` |
| **Lifecycle** | Resource management | Mount/unmount hooks for setup/cleanup |
| **Event-driven** | User interactions | Event handlers, state updates |
| **Async** | Data fetching, API calls | Loading states, error handling |
| **Complex State** | Applications, forms | Multiple state pieces, undo/redo |

## Script Section Checklist

- [ ] Component struct defined with appropriate fields
- [ ] Props struct defined with `Clone` trait
- [ ] `Component` trait implemented with all required methods
- [ ] Event handlers implemented as needed
- [ ] Lifecycle methods implemented for resource management
- [ ] Error handling included in render method
- [ ] State management follows reactive patterns
- [ ] Props validation included where appropriate
- [ ] Performance considerations addressed
- [ ] Documentation comments added for public APIs

This comprehensive guide covers the essential patterns and requirements for implementing robust component scripts in the Orbit framework.
