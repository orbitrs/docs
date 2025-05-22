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
  use orbitui::prelude::*;
  
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

Orbit components follow a predictable lifecycle:

1. **Creation**: `fn new(props: Self::Props) -> Self`
   - Component is instantiated with initial props
   - Initial state is set up

2. **Mounting**: `fn mounted(&mut self)`
   - Component is added to the DOM/rendering tree
   - Good place for initialization that requires DOM access
   - Setup subscriptions or timers

3. **Updating**: `fn updated(&mut self)`
   - Called after the component re-renders due to state/prop changes
   - Access to the updated DOM
   - Opportunity to integrate with non-Orbit libraries

4. **Unmounting**: `fn unmounted(&mut self)`
   - Component is about to be removed from the DOM
   - Clean up any subscriptions, timers, or resources

Here's a complete implementation example:

```rust
impl Component for MyComponent {
    type Props = MyComponentProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            name: props.name,
        }
    }
    
    fn mounted(&mut self) {
        println!("Component mounted");
        // Setup code here
    }
    
    fn updated(&mut self) {
        println!("Component updated");
        // Post-update code here
    }
    
    fn unmounted(&mut self) {
        println!("Component unmounted");
        // Cleanup code here
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
  use orbitui::prelude::*;
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
    use orbitui::test_utils::*;
    
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
