# Testing Strategies for Orbit Applications

This guide covers comprehensive testing approaches for Orbit Framework applications, from unit testing individual components to end-to-end testing of complete applications.

## Overview

A robust testing strategy for Orbit applications typically includes:

1. **Unit Tests**: Testing individual components and functions in isolation
2. **Integration Tests**: Testing interactions between components
3. **Rendering Tests**: Testing that components render correctly
4. **State Management Tests**: Testing state changes and effects
5. **End-to-End Tests**: Testing complete user flows in a browser-like environment

## Test Setup

### Prerequisites

- Rust test framework (built into Rust via `cargo test`)
- Web-driver for browser testing (if doing E2E tests)
- Orbit test utilities

### Basic Project Configuration

Ensure your `Cargo.toml` includes the necessary testing dependencies:

```toml
[dev-dependencies]
orbit-test = { version = "0.1.0" }
wasm-bindgen-test = "0.3"
mockito = "0.31"
```

## Unit Testing Components

### Component Unit Test Structure

Unit tests for Orbit components typically follow this structure:

1. Create the component with test props
2. Simulate user interactions or state changes
3. Assert that the component's state or output matches expectations

### Example: Testing a Counter Component

```rust
use orbit::prelude::*;
use orbit_test::TestRenderer;

#[test]
fn test_counter_increment() {
    // 1. Create component with initial props
    let mut test_renderer = TestRenderer::new();
    let component_id = test_renderer.render_component::<Counter>(CounterProps {
        initial: Some(5),
        step: Some(2),
    });
    
    // Access the component instance
    let counter = test_renderer.get_component::<Counter>(component_id);
    assert_eq!(counter.count, 5);
    
    // 2. Simulate a button click to trigger increment
    test_renderer.dispatch_event(component_id, "increment-button", "click", ());
    
    // 3. Assert the component updated correctly
    let counter = test_renderer.get_component::<Counter>(component_id);
    assert_eq!(counter.count, 7); // 5 + 2 = 7
}
```

### Testing Component Props

Test that your component correctly handles different prop values:

```rust
#[test]
fn test_counter_props() {
    let mut test_renderer = TestRenderer::new();
    
    // Test with explicit values
    let c1 = test_renderer.render_component::<Counter>(CounterProps {
        initial: Some(10),
        step: Some(5),
    });
    
    // Test with default values
    let c2 = test_renderer.render_component::<Counter>(CounterProps {
        initial: None,
        step: None,
    });
    
    let counter1 = test_renderer.get_component::<Counter>(c1);
    let counter2 = test_renderer.get_component::<Counter>(c2);
    
    assert_eq!(counter1.count, 10);
    assert_eq!(counter1.step, 5);
    assert_eq!(counter2.count, 0);  // Default value
    assert_eq!(counter2.step, 1);   // Default value
}
```

### Testing Component Methods

Test individual methods within a component:

```rust
#[test]
fn test_counter_methods() {
    let mut counter = Counter { count: 0, step: 1 };
    
    counter.increment();
    assert_eq!(counter.count, 1);
    
    counter.decrement();
    assert_eq!(counter.count, 0);
    
    counter.step = 5;
    counter.increment();
    assert_eq!(counter.count, 5);
}
```

## Testing Component Rendering

### Testing the Component Output

Verify that a component renders the expected structure:

```rust
#[test]
fn test_counter_renders_correctly() {
    let mut test_renderer = TestRenderer::new();
    let component_id = test_renderer.render_component::<Counter>(CounterProps {
        initial: Some(42),
        step: Some(1),
    });
    
    // Get the rendered HTML
    let html = test_renderer.get_html(component_id);
    
    // Verify key elements are present
    assert!(html.contains(">42<"));
    assert!(html.contains("counter-value"));
    assert!(html.contains("increment-button"));
    assert!(html.contains("decrement-button"));
}
```

### Testing Dynamic Content

Test that dynamic content updates correctly:

```rust
#[test]
fn test_counter_updates_display() {
    let mut test_renderer = TestRenderer::new();
    let component_id = test_renderer.render_component::<Counter>(CounterProps::default());
    
    // Initial state
    let html = test_renderer.get_html(component_id);
    assert!(html.contains(">0<"));
    
    // After incrementing
    test_renderer.dispatch_event(component_id, "increment-button", "click", ());
    let html = test_renderer.get_html(component_id);
    assert!(html.contains(">1<"));
    
    // After decrementing
    test_renderer.dispatch_event(component_id, "decrement-button", "click", ());
    let html = test_renderer.get_html(component_id);
    assert!(html.contains(">0<"));
}
```

## Testing Component Events

### Testing Event Handlers

Test that event handlers work correctly:

```rust
#[test]
fn test_counter_event_handlers() {
    let mut test_renderer = TestRenderer::new();
    let component_id = test_renderer.render_component::<Counter>(CounterProps::default());
    
    // Test click event on increment button
    test_renderer.dispatch_event(component_id, "increment-button", "click", ());
    let counter = test_renderer.get_component::<Counter>(component_id);
    assert_eq!(counter.count, 1);
    
    // Test click event on decrement button
    test_renderer.dispatch_event(component_id, "decrement-button", "click", ());
    let counter = test_renderer.get_component::<Counter>(component_id);
    assert_eq!(counter.count, 0);
}
```

### Testing Custom Events

Test components that emit custom events:

```rust
#[test]
fn test_counter_emits_change_event() {
    let mut test_renderer = TestRenderer::new();
    
    // Set up event listener
    let mut event_received = false;
    let mut new_value = 0;
    
    test_renderer.add_event_listener("counter-changed", |event| {
        event_received = true;
        new_value = event.value;
    });
    
    let component_id = test_renderer.render_component::<Counter>(CounterProps::default());
    
    // Trigger event
    test_renderer.dispatch_event(component_id, "increment-button", "click", ());
    
    // Verify event was emitted
    assert!(event_received);
    assert_eq!(new_value, 1);
}
```

## Integration Testing

### Testing Component Composition

Test that components interact correctly when composed:

```rust
#[test]
fn test_counter_app_integration() {
    let mut test_renderer = TestRenderer::new();
    
    // Render parent component that contains Counter
    let app_id = test_renderer.render_component::<CounterApp>(CounterAppProps::default());
    
    // Find the Counter within the app
    let counter_component_id = test_renderer.find_component::<Counter>(app_id);
    assert!(counter_component_id.is_some());
    
    // Test interaction through parent component
    test_renderer.dispatch_event(app_id, "app-increment-button", "click", ());
    
    // Verify Counter's state changed
    let counter = test_renderer.get_component::<Counter>(counter_component_id.unwrap());
    assert_eq!(counter.count, 1);
}
```

### Testing Component Communication

Test that parent and child components communicate properly:

```rust
#[test]
fn test_parent_child_communication() {
    let mut test_renderer = TestRenderer::new();
    let parent_id = test_renderer.render_component::<Parent>(ParentProps::default());
    
    // Parent should render a Child component
    let child_id = test_renderer.find_component::<Child>(parent_id).unwrap();
    
    // Child emits event
    test_renderer.dispatch_event(child_id, "notify-button", "click", ());
    
    // Parent should react to child event
    let parent = test_renderer.get_component::<Parent>(parent_id);
    assert!(parent.notification_received);
}
```

## State Management Tests

### Testing Local State

Test that local component state updates correctly:

```rust
#[test]
fn test_form_state_updates() {
    let mut test_renderer = TestRenderer::new();
    let form_id = test_renderer.render_component::<UserForm>(UserFormProps::default());
    
    // Simulate user input
    test_renderer.dispatch_input_event(form_id, "name-input", "John Doe");
    test_renderer.dispatch_input_event(form_id, "email-input", "john@example.com");
    
    // Check state was updated
    let form = test_renderer.get_component::<UserForm>(form_id);
    assert_eq!(form.user.name, "John Doe");
    assert_eq!(form.user.email, "john@example.com");
}
```

### Testing Global State (Store)

Test components that interact with a global state store:

```rust
#[test]
fn test_counter_with_global_store() {
    // Set up a test store
    let store = orbit::store::TestStore::new(AppState {
        counter: 0,
        user: None,
    });
    
    let mut test_renderer = TestRenderer::with_store(store.clone());
    let component_id = test_renderer.render_component::<ConnectedCounter>(ConnectedCounterProps::default());
    
    // Component should initialize with store value
    let counter = test_renderer.get_component::<ConnectedCounter>(component_id);
    assert_eq!(counter.count, 0);
    
    // Update store
    store.dispatch(Action::IncrementCounter);
    
    // Component should update from store
    let counter = test_renderer.get_component::<ConnectedCounter>(component_id);
    assert_eq!(counter.count, 1);
    
    // Component action should update store
    test_renderer.dispatch_event(component_id, "increment-button", "click", ());
    assert_eq!(store.state().counter, 2);
}
```

## Testing Asynchronous Logic

### Testing API Calls

Test components that make API calls using a mock server:

```rust
use mockito::{mock, server_url};

#[test]
fn test_data_fetching_component() {
    // Set up mock API
    let _m = mock("GET", "/api/users/1")
        .with_status(200)
        .with_header("content-type", "application/json")
        .with_body(r#"{"id": 1, "name": "John Doe"}"#)
        .create();
    
    // Override API URL with mock server URL
    std::env::set_var("API_BASE_URL", server_url());
    
    // Render component
    let mut test_renderer = TestRenderer::new();
    let component_id = test_renderer.render_component::<UserProfile>(UserProfileProps {
        user_id: 1,
    });
    
    // Component starts with loading state
    let profile = test_renderer.get_component::<UserProfile>(component_id);
    assert!(profile.loading);
    
    // Wait for async operation to complete
    test_renderer.wait_for_component_update(component_id);
    
    // Check component updated with data
    let profile = test_renderer.get_component::<UserProfile>(component_id);
    assert!(!profile.loading);
    assert_eq!(profile.user.name, "John Doe");
}
```

### Testing Timeouts and Animations

Test components with timers or animations:

```rust
#[test]
fn test_timed_notification() {
    let mut test_renderer = TestRenderer::new();
    let component_id = test_renderer.render_component::<NotificationToast>(NotificationToastProps {
        message: "Test notification".to_string(),
        duration_ms: 100, // Short duration for testing
    });
    
    // Toast should be visible initially
    let toast = test_renderer.get_component::<NotificationToast>(component_id);
    assert!(toast.visible);
    
    // Advance mock time
    test_renderer.advance_time(150); // ms
    
    // Toast should hide automatically
    let toast = test_renderer.get_component::<NotificationToast>(component_id);
    assert!(!toast.visible);
}
```

## End-to-End Testing

For browser-based end-to-end testing, you can use tools like WebDriver bindings for Rust:

```rust
use thirtyfour::prelude::*;

#[tokio::test]
async fn test_counter_e2e() -> WebDriverResult<()> {
    let driver = WebDriver::new("http://localhost:4444").await?;
    
    // Navigate to your app (running locally or in a test environment)
    driver.get("http://localhost:8080").await?;
    
    // Find elements
    let counter_value = driver.find_element(By::Id("counter-value")).await?;
    let increment_button = driver.find_element(By::Id("increment-button")).await?;
    
    // Check initial state
    assert_eq!(counter_value.text().await?, "0");
    
    // Interact with the app
    increment_button.click().await?;
    
    // Check updated state
    assert_eq!(counter_value.text().await?, "1");
    
    // Clean up
    driver.quit().await?;
    
    Ok(())
}
```

## Snapshot Testing

Snapshot testing compares the rendered output with a previously saved "snapshot" to detect unintended changes:

```rust
#[test]
fn test_button_component_snapshot() {
    let mut test_renderer = TestRenderer::new();
    let component_id = test_renderer.render_component::<Button>(ButtonProps {
        label: "Click me".to_string(),
        variant: ButtonVariant::Primary,
        disabled: false,
    });
    
    // Generate HTML snapshot
    let html = test_renderer.get_html(component_id);
    
    // Compare with saved snapshot
    assert_snapshot_matches!("button_primary", html);
}
```

## Test Organization

### Directory Structure

Organize your tests with a clear structure:

```
src/
  components/
    counter.orbit
    user_form.orbit
  ...
tests/
  components/
    counter_test.rs
    user_form_test.rs
  integration/
    form_submission_test.rs
  e2e/
    user_flow_test.rs
```

### Test Helpers and Fixtures

Create reusable test helpers and fixtures:

```rust
// test_helpers.rs
pub fn create_test_user() -> User {
    User {
        id: 1,
        name: "Test User".to_string(),
        email: "test@example.com".to_string(),
    }
}

pub fn render_test_app() -> (TestRenderer, ComponentId) {
    let mut renderer = TestRenderer::new();
    let app_id = renderer.render_component::<App>(AppProps::default());
    (renderer, app_id)
}
```

## Running Tests

### Running All Tests

```bash
cargo test
```

### Running Specific Tests

```bash
# Run tests in a specific file
cargo test --test counter_test

# Run a specific test by name
cargo test test_counter_increment

# Run tests with a specific pattern in their name
cargo test counter
```

### Running with Wasm

For web-specific tests:

```bash
wasm-pack test --chrome
```

## Continuous Integration

Add testing to your CI pipeline:

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          
      - name: Install wasm-pack
        run: cargo install wasm-pack
          
      - name: Run tests
        run: cargo test
        
      - name: Run wasm tests
        run: wasm-pack test --headless --chrome
```

## Testing Best Practices

1. **Test Behavior, Not Implementation**: Focus on what the component does, not how it does it
2. **Isolate Tests**: Each test should be independent and not rely on the state from other tests
3. **Mock External Dependencies**: Use mocks for API calls, storage, etc.
4. **Test Edge Cases**: Test boundary conditions and error scenarios
5. **Keep Tests Fast**: Optimize test performance to maintain a quick feedback loop
6. **Test Accessibility**: Include tests for accessibility features
7. **Maintain Test Coverage**: Aim for high test coverage, especially for critical components

## Conclusion

A comprehensive testing strategy is essential for building robust Orbit applications. By combining unit tests, integration tests, and end-to-end tests, you can ensure your application functions correctly and remains maintainable as it grows.

## Related Resources

- [Component Model Documentation](../core-concepts/component-model.md)
- [State Management](../core-concepts/state-management.md)
- [Event Handling](../core-concepts/event-handling.md)
- [Orbit Test API Reference](../api/orbit-test-api.md)
