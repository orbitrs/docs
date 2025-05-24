# orbit::kit::hooks API Reference

The `orbit::kit::hooks` module provides React-style hooks for managing state and side effects in Orbit components. These hooks enable functional programming patterns and reusable stateful logic.

## Overview

Hooks in Orbit provide a way to:
- Manage component state without class-based components
- Create reusable stateful logic
- Handle side effects and lifecycle events
- Integrate with the reactive state system

## Core Hooks

### use_state

Creates and manages local component state.

```rust
pub fn use_state<T>(initial: T) -> (Signal<T>, impl Fn(T))
where
    T: Clone + 'static,
{
    // Returns a signal for reading the value and a setter function
}
```

**Parameters:**
- `initial: T` - The initial value for the state

**Returns:**
- `(Signal<T>, impl Fn(T))` - A tuple containing:
  - A signal for reading the current value
  - A setter function for updating the value

**Example:**
```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

#[component]
pub fn Counter() -> impl View {
    let (count, set_count) = use_state(0);
    
    let increment = use_callback(move |_| {
        set_count(count.get() + 1);
    });
    
    html! {
        <div>
            <p>Count: {count.get()}</p>
            <button on:click={increment}>Increment</button>
        </div>
    }
}
```

### use_effect

Performs side effects and manages cleanup.

```rust
pub fn use_effect<F, D>(effect: F, dependencies: D)
where
    F: FnOnce() -> Option<Box<dyn FnOnce()>> + 'static,
    D: PartialEq + 'static,
{
    // Runs the effect when dependencies change
}
```

**Parameters:**
- `effect: F` - A function that performs the side effect. Can return a cleanup function
- `dependencies: D` - Values that trigger the effect when they change

**Example:**
```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

#[component]
pub fn Timer() -> impl View {
    let (seconds, set_seconds) = use_state(0);
    
    // Effect with cleanup
    use_effect(move || {
        let interval = setInterval(move || {
            set_seconds(seconds.get() + 1);
        }, 1000);
        
        // Cleanup function
        Some(Box::new(move || {
            clearInterval(interval);
        }))
    }, ());
    
    html! {
        <div>Timer: {seconds.get()}s</div>
    }
}
```

### use_callback

Memoizes callback functions to prevent unnecessary re-renders.

```rust
pub fn use_callback<F, D>(callback: F, dependencies: D) -> impl Fn()
where
    F: Fn() + 'static,
    D: PartialEq + 'static,
{
    // Returns a memoized version of the callback
}
```

**Parameters:**
- `callback: F` - The function to memoize
- `dependencies: D` - Values that cause the callback to be recreated when they change

**Example:**
```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

#[component]
pub fn SearchBox() -> impl View {
    let (query, set_query) = use_state(String::new());
    let (results, set_results) = use_state(Vec::new());
    
    // Memoized search function
    let handle_search = use_callback(move |new_query: String| {
        set_query(new_query.clone());
        
        // Perform search (this would be async in real implementation)
        let search_results = search_api(&new_query);
        set_results(search_results);
    }, query.get().clone());
    
    html! {
        <div>
            <input 
                type="text" 
                value={query.get().clone()}
                on:input={move |e: InputEvent| handle_search(e.value())}
            />
            <ul>
                {results.get().iter().map(|result| html! {
                    <li>{result.title.clone()}</li>
                }).collect::<Vec<_>>()}
            </ul>
        </div>
    }
}
```

### use_memo

Memoizes expensive computations.

```rust
pub fn use_memo<T, F, D>(compute: F, dependencies: D) -> T
where
    T: Clone + 'static,
    F: Fn() -> T + 'static,
    D: PartialEq + 'static,
{
    // Returns a memoized value that only recomputes when dependencies change
}
```

**Parameters:**
- `compute: F` - Function that computes the value
- `dependencies: D` - Values that trigger recomputation when they change

**Example:**
```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

#[component]
pub fn ExpensiveCalculation() -> impl View {
    let (input, set_input) = use_state(0);
    
    // Expensive computation only runs when input changes
    let result = use_memo(move || {
        // Simulate expensive calculation
        (0..input.get()).fold(0, |acc, i| acc + i * i)
    }, input.get());
    
    html! {
        <div>
            <input 
                type="number" 
                value={input.get().to_string()}
                on:input={move |e: InputEvent| {
                    if let Ok(value) = e.value().parse() {
                        set_input(value);
                    }
                }}
            />
            <p>Result: {result}</p>
        </div>
    }
}
```

### use_reducer

Manages complex state with actions and reducers.

```rust
pub fn use_reducer<S, A, F>(reducer: F, initial_state: S) -> (S, impl Fn(A))
where
    S: Clone + 'static,
    A: 'static,
    F: Fn(&S, &A) -> S + 'static,
{
    // Returns current state and dispatch function
}
```

**Parameters:**
- `reducer: F` - Function that takes current state and action, returns new state
- `initial_state: S` - The initial state value

**Returns:**
- `(S, impl Fn(A))` - Tuple containing current state and dispatch function

**Example:**
```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

#[derive(Clone)]
enum CounterAction {
    Increment,
    Decrement,
    Reset,
    SetValue(i32),
}

fn counter_reducer(state: &i32, action: &CounterAction) -> i32 {
    match action {
        CounterAction::Increment => state + 1,
        CounterAction::Decrement => state - 1,
        CounterAction::Reset => 0,
        CounterAction::SetValue(value) => *value,
    }
}

#[component]
pub fn CounterWithReducer() -> impl View {
    let (count, dispatch) = use_reducer(counter_reducer, 0);
    
    html! {
        <div>
            <p>Count: {count}</p>
            <button on:click={move |_| dispatch(CounterAction::Increment)}>+</button>
            <button on:click={move |_| dispatch(CounterAction::Decrement)}>-</button>
            <button on:click={move |_| dispatch(CounterAction::Reset)}>Reset</button>
        </div>
    }
}
```

### use_context

Accesses context values provided by ancestor components.

```rust
pub fn use_context<T>() -> Option<T>
where
    T: Clone + 'static,
{
    // Returns the context value if available
}
```

**Example:**
```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

#[derive(Clone)]
pub struct Theme {
    primary_color: String,
    background_color: String,
}

#[component]
pub fn ThemedButton() -> impl View {
    let theme = use_context::<Theme>().unwrap_or_else(|| Theme {
        primary_color: "#007bff".to_string(),
        background_color: "#ffffff".to_string(),
    });
    
    html! {
        <button 
            style={format!(
                "background-color: {}; color: {}", 
                theme.primary_color, 
                theme.background_color
            )}
        >
            Click me
        </button>
    }
}
```

## Custom Hooks

You can create custom hooks by combining built-in hooks:

### use_fetch

```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

#[derive(Clone)]
pub struct FetchState<T> {
    data: Option<T>,
    loading: bool,
    error: Option<String>,
}

pub fn use_fetch<T>(url: String) -> FetchState<T>
where
    T: Clone + serde::DeserializeOwned + 'static,
{
    let (state, set_state) = use_state(FetchState {
        data: None,
        loading: true,
        error: None,
    });
    
    use_effect(move || {
        let url = url.clone();
        let set_state = set_state.clone();
        
        // Start fetch
        spawn_local(async move {
            match fetch_data::<T>(&url).await {
                Ok(data) => set_state(FetchState {
                    data: Some(data),
                    loading: false,
                    error: None,
                }),
                Err(err) => set_state(FetchState {
                    data: None,
                    loading: false,
                    error: Some(err.to_string()),
                }),
            }
        });
        
        None // No cleanup needed
    }, url);
    
    state.get().clone()
}

// Usage
#[component]
pub fn UserProfile() -> impl View {
    let user_data = use_fetch::<User>("/api/user/profile".to_string());
    
    match user_data {
        FetchState { loading: true, .. } => html! { <div>Loading...</div> },
        FetchState { error: Some(err), .. } => html! { <div>Error: {err}</div> },
        FetchState { data: Some(user), .. } => html! {
            <div>
                <h1>{user.name}</h1>
                <p>{user.email}</p>
            </div>
        },
        _ => html! { <div>No data</div> },
    }
}
```

### use_local_storage

```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;

pub fn use_local_storage<T>(key: &str, initial: T) -> (T, impl Fn(T))
where
    T: Clone + serde::Serialize + serde::DeserializeOwned + 'static,
{
    // Load initial value from localStorage
    let loaded_value = load_from_storage(key).unwrap_or(initial);
    let (value, set_value) = use_state(loaded_value);
    
    let key = key.to_string();
    let setter = use_callback(move |new_value: T| {
        save_to_storage(&key, &new_value);
        set_value(new_value);
    }, ());
    
    (value.get().clone(), setter)
}

// Usage
#[component]
pub fn Settings() -> impl View {
    let (dark_mode, set_dark_mode) = use_local_storage("dark_mode", false);
    
    html! {
        <div>
            <label>
                <input 
                    type="checkbox" 
                    checked={dark_mode}
                    on:change={move |e: ChangeEvent| set_dark_mode(e.checked())}
                />
                Dark Mode
            </label>
        </div>
    }
}
```

## Hook Rules

When using hooks, follow these rules:

1. **Only call hooks at the top level** - Don't call hooks inside loops, conditions, or nested functions
2. **Only call hooks from components** - Don't call hooks from regular Rust functions
3. **Use consistent ordering** - Always call hooks in the same order across renders

### Correct Usage

```rust
#[component]
pub fn MyComponent() -> impl View {
    let (state1, set_state1) = use_state(0);
    let (state2, set_state2) = use_state(String::new());
    
    use_effect(move || {
        // Effect logic
        None
    }, ());
    
    // Render logic...
}
```

### Incorrect Usage

```rust
#[component]
pub fn MyComponent() -> impl View {
    if some_condition {
        let (state, set_state) = use_state(0); // ❌ Don't do this
    }
    
    for i in 0..5 {
        use_effect(move || {}, ()); // ❌ Don't do this
    }
    
    // Render logic...
}
```

## Integration with Reactive System

Hooks integrate seamlessly with Orbit's reactive state system:

```rust
use orbit::prelude::*;
use orbit::kit::hooks::*;
use orbit::state::{create_signal, create_computed};

#[component]
pub fn ReactiveExample() -> impl View {
    // Hook-based state
    let (count, set_count) = use_state(0);
    
    // Reactive signals
    let signal_count = create_signal(0);
    let double_count = create_computed(move || signal_count.get() * 2);
    
    // Effect that bridges hook state and reactive signals
    use_effect(move || {
        signal_count.set(count);
        None
    }, count);
    
    html! {
        <div>
            <p>Hook count: {count}</p>
            <p>Signal count: {signal_count.get()}</p>
            <p>Double count: {double_count.get()}</p>
            <button on:click={move |_| set_count(count + 1)}>Increment</button>
        </div>
    }
}
```

## Best Practices

### Performance

- Use `use_callback` for event handlers that are passed as props
- Use `use_memo` for expensive computations
- Minimize dependencies in `use_effect` to avoid unnecessary re-runs

### State Management

- Use `use_state` for simple local state
- Use `use_reducer` for complex state logic
- Use custom hooks to share stateful logic between components

### Side Effects

- Always clean up side effects in `use_effect`
- Use dependencies array to control when effects run
- Prefer declarative effects over imperative ones

### Testing

Hooks can be tested by creating test components:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use orbit::testing::*;
    
    #[test]
    fn test_use_state_hook() {
        let mut renderer = TestRenderer::new();
        
        #[component]
        fn TestComponent() -> impl View {
            let (count, set_count) = use_state(0);
            
            html! {
                <div>
                    <span data-testid="count">{count}</span>
                    <button data-testid="increment" on:click={move |_| set_count(count + 1)}>
                        Increment
                    </button>
                </div>
            }
        }
        
        let component = renderer.render(html! { <TestComponent /> });
        
        // Test initial state
        assert_eq!(component.find_by_test_id("count").text(), "0");
        
        // Test state update
        component.find_by_test_id("increment").click();
        assert_eq!(component.find_by_test_id("count").text(), "1");
    }
}
```

## See Also

- [State Management](../core-concepts/state-management.md) - Overall state management concepts
- [Reactive State](../core-concepts/reactive-state.md) - Reactive signals and effects
- [Component Model](../core-concepts/component-model.md) - Component lifecycle and patterns
- [orbit::state API](./orbit-state.md) - Core state management APIs
