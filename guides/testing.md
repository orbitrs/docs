# 🧪 Testing Guide

## 📝 Overview

Comprehensive testing is crucial for building reliable Orbit applications. This guide covers different testing approaches and tools available in the Orbit ecosystem to help you build robust and maintainable UI components and applications.

## 🧩 Types of Tests

### Unit Tests

Unit tests verify that individual components and functions work correctly in isolation:

```rust
use orbitrs::testing::*;
use my_app::components::Counter;

#[test]
fn test_counter_increment() {
    // Create a test instance of the Counter component
    let mut component = TestComponent::<Counter>::new();
    
    // Assert initial state
    assert_eq!(*component.get_state::<i32>("count"), 0);
    
    // Call the increment method
    component.call_method("increment", &[]);
    
    // Verify the state was updated
    assert_eq!(*component.get_state::<i32>("count"), 1);
}
```

### Component Tests

Component tests focus on testing the rendering and behavior of individual components:

```rust
use orbitrs::testing::*;
use my_app::components::UserCard;

#[test]
fn test_user_card_renders_correctly() {
    // Create test data
    let user = User {
        id: "123".to_string(),
        name: "Jane Doe".to_string(),
        email: "jane@example.com".to_string(),
    };
    
    // Render the component with props
    let component = render_component(UserCard::new, |props| {
        props.insert("user", user);
    });
    
    // Query rendered output
    let name_element = component.query_selector("h2.user-name");
    let email_element = component.query_selector("p.user-email");
    
    // Verify rendered content
    assert_eq!(name_element.text_content(), "Jane Doe");
    assert_eq!(email_element.text_content(), "jane@example.com");
}
```

### Integration Tests

Integration tests verify that multiple components work together correctly:

```rust
use orbitrs::testing::*;
use my_app::components::{TodoList, TodoForm};

#[test]
fn test_adding_todo_item() {
    // Render the parent component that contains both TodoForm and TodoList
    let app = render_component(App::new, |_| {});
    
    // Find the input field and submit button
    let input = app.query_selector("input[name='todo-input']");
    let button = app.query_selector("button[type='submit']");
    
    // Simulate user input
    input.set_value("Buy groceries");
    
    // Simulate button click
    button.click();
    
    // Verify the new todo was added to the list
    let todo_items = app.query_selector_all("ul.todo-list li");
    assert_eq!(todo_items.length(), 1);
    assert!(todo_items[0].text_content().contains("Buy groceries"));
}
```

### End-to-End Tests

End-to-end tests verify that entire user flows work correctly:

```rust
use orbitrs::testing::e2e::*;

#[tokio::test]
async fn test_user_registration_flow() {
    // Start an application instance
    let app = TestApp::new().await;
    
    // Navigate to registration page
    app.goto("/register").await;
    
    // Fill out the registration form
    app.fill("input[name='username']", "testuser").await;
    app.fill("input[name='email']", "test@example.com").await;
    app.fill("input[name='password']", "securepassword").await;
    app.fill("input[name='confirm_password']", "securepassword").await;
    
    // Submit the form
    app.click("button[type='submit']").await;
    
    // Verify redirection to dashboard
    assert_eq!(app.current_url().await, "http://localhost:8080/dashboard");
    
    // Verify welcome message
    let welcome_message = app.text_content("h1.welcome-message").await;
    assert!(welcome_message.contains("Welcome, testuser"));
}
```

## 🛠️ Testing Utilities

### Test Renderers

Orbit provides several test renderers optimized for different testing scenarios:

```rust
use orbitrs::testing::*;

// Memory renderer - fastest, minimal overhead
let memory_renderer = MemoryRenderer::new();
let component = render_with(memory_renderer, MyComponent::new, |props| {
    // Set props as needed
});

// Headless DOM renderer - useful for DOM-related testing
let dom_renderer = HeadlessRenderer::new();
let component = render_with(dom_renderer, MyComponent::new, |props| {
    // Set props as needed
});

// Visual renderer - can generate screenshots for visual testing
let visual_renderer = VisualRenderer::new(800, 600); // width, height
let component = render_with(visual_renderer, MyComponent::new, |props| {
    // Set props as needed
});
let screenshot = component.take_screenshot();
```

### Component Querying

Find elements and inspect the component structure:

```rust
use orbitrs::testing::*;

// Render a component for testing
let component = render_component(MyComponent::new, |props| {
    props.insert("items", vec!["Item 1", "Item 2"]);
});

// Query elements (similar to CSS selectors)
let button = component.query_selector("button.primary");
let list_items = component.query_selector_all("ul.list li");

// Check properties and attributes
assert_eq!(button.get_attribute("disabled"), None); // Not disabled
assert_eq!(list_items.length(), 2);
assert_eq!(list_items[0].text_content(), "Item 1");

// Query by test ID (recommended for stable tests)
let submit_button = component.query_by_test_id("submit-button");
assert!(submit_button.is_visible());
```

### Event Simulation

Simulate user interactions:

```rust
use orbitrs::testing::*;
use orbitrs::events::*;

// Render component
let component = render_component(SearchForm::new, |_| {});

// Find elements
let input = component.query_selector("input[type='search']");
let button = component.query_selector("button[type='submit']");

// Simulate typing
input.set_value("test query");

// Simulate click
button.click();

// Check if the search was triggered
let results_container = component.query_selector(".search-results");
assert!(results_container.is_visible());

// Simulate keyboard events
input.dispatch_event(KeyboardEvent::new("keydown", |e| {
    e.key = "Enter".to_string();
}));

// Simulate custom events
component.dispatch_custom_event("custom-event", json!({
    "detail": { "value": 42 }
}));
```

### State and Props Testing

Test component state and props:

```rust
use orbitrs::testing::*;

// Create a testable component
let mut component = TestComponent::<Counter>::new();

// Set initial props
component.set_prop("initial_count", 10);

// Initialize the component
component.mount();

// Get current state
let count = component.get_state::<i32>("count");
assert_eq!(*count, 10);

// Call a method
component.call_method("increment", &[]);

// Verify state was updated
let updated_count = component.get_state::<i32>("count");
assert_eq!(*updated_count, 11);
```

### Snapshot Testing

Verify that component rendering doesn't change unexpectedly:

```rust
use orbitrs::testing::*;

#[test]
fn test_button_snapshot() {
    // Render a component
    let button = render_component(Button::new, |props| {
        props.insert("label", "Click Me");
        props.insert("variant", "primary");
    });
    
    // Take a snapshot of the rendered output
    let snapshot = button.snapshot();
    
    // Verify against stored snapshot
    assert_matches_snapshot!(snapshot, "button_primary");
    
    // You can also use JSON snapshots for more detailed structure testing
    let structure = button.structure_snapshot();
    assert_matches_snapshot!(structure, "button_primary_structure");
}
```

## 📊 Test Coverage

### Measuring Coverage

Orbit provides tools for measuring test coverage:

```bash
# Run tests with coverage
orbiton test --coverage

# Generate coverage report
orbiton test --coverage --report
```

### Coverage Reports

Coverage reports provide insights into which parts of your code are tested:

```
Component Coverage Summary:
--------------------------
src/components/Button.orbit   | 95.2% | ✅
src/components/Input.orbit    | 87.3% | ✅
src/components/Card.orbit     | 42.1% | ❌
src/components/Modal.orbit    | 68.9% | ⚠️

Event Handler Coverage:
--------------------
src/components/Button.orbit   | 100%  | ✅
src/components/Input.orbit    | 75.0% | ⚠️
src/components/Card.orbit     | 33.3% | ❌
src/components/Modal.orbit    | 80.0% | ✅

Overall Coverage: 73.5%
```

## 🚀 Testing Best Practices

### Component Testing

1. **Test props and state interactions**:
   ```rust
   #[test]
   fn test_button_disabled_state() {
       let button = render_component(Button::new, |props| {
           props.insert("disabled", true);
       });
       
       assert!(button.has_class("disabled"));
       assert_eq!(button.get_attribute("disabled"), Some("true"));
   }
   ```

2. **Test lifecycle methods**:
   ```rust
   #[test]
   fn test_component_lifecycle() {
       let mut component = TestComponent::<DataLoader>::new();
       
       // Mount the component (triggers mounted lifecycle hook)
       component.mount();
       
       // Verify mounted hook side effects
       assert!(component.get_state::<bool>("is_loading"));
       
       // Simulate async data load completion
       component.set_state("data", vec!["item1", "item2"]);
       component.set_state("is_loading", false);
       
       // Trigger update lifecycle
       component.update();
       
       // Verify the component shows loaded data
       let rendered = component.render();
       assert!(rendered.text_content().contains("item1"));
   }
   ```

3. **Test event handlers**:
   ```rust
   #[test]
   fn test_input_change_handler() {
       let input = render_component(Input::new, |_| {});
       
       // Initial value should be empty
       assert_eq!(input.value(), "");
       
       // Simulate input change
       input.set_value("New value");
       
       // Check if value was updated
       assert_eq!(input.value(), "New value");
       
       // Verify change event was emitted with correct value
       let events = input.emitted_events();
       assert_eq!(events.len(), 1);
       assert_eq!(events[0].name, "change");
       assert_eq!(events[0].detail.get("value"), "New value");
   }
   ```

### Test Organization

Organize tests in a way that mirrors your application structure:

```
my_app/
├── src/
│   ├── components/
│   │   ├── button.orbit
│   │   └── input.orbit
│   └── pages/
│       └── home.orbit
└── tests/
    ├── unit/
    │   ├── components/
    │   │   ├── button_test.rs
    │   │   └── input_test.rs
    │   └── pages/
    │       └── home_test.rs
    ├── integration/
    │   └── form_submission_test.rs
    └── e2e/
        └── user_flow_test.rs
```

## 🧠 Advanced Testing Topics

### Testing Async Components

Test components that perform asynchronous operations:

```rust
use orbitrs::testing::*;

#[tokio::test]
async fn test_async_data_loading() {
    // Mock API response
    mock_api_response("/api/users", json!([
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ]));
    
    // Render the component that fetches data
    let component = render_component(UserList::new, |_| {});
    
    // Initially should show loading state
    assert!(component.query_selector(".loading").is_visible());
    
    // Wait for the async operation to complete
    await_stable_ui(&component).await;
    
    // Loading indicator should be gone
    assert!(!component.query_selector(".loading").is_visible());
    
    // List should contain items from the mock response
    let items = component.query_selector_all(".user-item");
    assert_eq!(items.length(), 2);
    assert_eq!(items[0].text_content(), "Alice");
    assert_eq!(items[1].text_content(), "Bob");
}
```

### Testing With Mocks

Mock dependencies to isolate component testing:

```rust
use orbitrs::testing::*;
use orbitrs::mocks::*;

#[test]
fn test_theme_context() {
    // Create a mock theme context
    let theme_mock = mock_context(ThemeContext {
        current_theme: "dark".to_string(),
        is_high_contrast: false,
    });
    
    // Render component with mock context provider
    let component = render_with_context(ThemeAwareButton::new, theme_mock, |props| {
        props.insert("label", "Save");
    });
    
    // Verify the component uses the theme context
    assert!(component.has_class("dark-theme"));
    
    // Update the mock context
    theme_mock.update(|theme| {
        theme.current_theme = "light".to_string();
    });
    
    // Verify component reacts to context change
    assert!(!component.has_class("dark-theme"));
    assert!(component.has_class("light-theme"));
}
```

### Visual Regression Testing

Ensure UI appearance doesn't change unexpectedly:

```rust
use orbitrs::testing::visual::*;

#[test]
fn test_button_appearance() {
    // Create visual test environment
    let test_env = VisualTestEnvironment::new(VisualTestOptions {
        width: 800,
        height: 600,
        device_pixel_ratio: 2.0,
        // Additional options...
    });
    
    // Render component
    let button = test_env.render(Button::new, |props| {
        props.insert("label", "Submit");
        props.insert("variant", "primary");
    });
    
    // Take screenshot
    let screenshot = button.take_screenshot();
    
    // Compare with baseline
    let comparison = screenshot.compare_with_baseline("button_primary");
    
    // Assert the visual difference is within tolerance
    assert!(comparison.diff_percentage < 0.1);
    
    // Or save as new baseline if approved
    if env::var("UPDATE_BASELINES").is_ok() {
        screenshot.save_as_baseline("button_primary");
    }
}
```

## 🛣️ Testing Strategy

### Test Pyramid

Follow the test pyramid approach to balance test coverage, speed, and maintainability:

```
    /\
   /  \
  /    \
 / E2E  \           Few end-to-end tests
/--------\
/          \
/ Integration \     More integration tests
/--------------\
/                \
/     Unit        \  Many unit tests
/------------------\
```

### Continuous Integration

Set up automated testing in your CI pipeline:

```yaml
# .github/workflows/test.yml example
name: Tests

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          profile: minimal
          toolchain: stable
          
      - name: Install Orbiton
        run: cargo install orbiton-cli
        
      - name: Run unit tests
        run: orbiton test --unit
        
      - name: Run integration tests
        run: orbiton test --integration
        
      - name: Run coverage
        run: orbiton test --coverage --report
        
      - name: Upload coverage report
        uses: actions/upload-artifact@v2
        with:
          name: coverage-report
          path: target/coverage
```

## 🔄 Related Topics

- [Component Model](../core-concepts/component-model.md) - Understanding Orbit's component model
- [State Management](../core-concepts/state-management.md) - Learn about state management in Orbit
- [Event Handling](../core-concepts/event-handling.md) - Event system in Orbit
- [Performance Optimization](./performance-optimization.md) - Optimize your tested components
