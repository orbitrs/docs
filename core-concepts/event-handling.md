# 🎯 Event Handling in Orbit

## 📝 Overview

Event handling is a core aspect of interactive UI applications. The Orbit Framework provides a powerful and flexible event system that supports various interaction types across different platforms while maintaining a consistent developer experience.

## 🔄 Event Flow Architecture

Orbit's event system follows a bubbling and capturing model similar to the DOM, but with optimizations specific to Rust and cross-platform environments.

### Event Phases

1. **Capture Phase** - Events travel from the root component down to the target component
2. **Target Phase** - Event reaches the component where the event originated
3. **Bubble Phase** - Events bubble up from the target component to the root

```
     ┌─────────────────┐
     │     Root        │
     │   Component     │ ◄─── 3. Bubble Phase (up)
     └────────┬────────┘
              │
     ┌────────▼────────┐
     │     Parent      │
     │    Component    │
     └────────┬────────┘
              │
┌─────────────▼─────────────┐
│        Target             │
│       Component           │ ◄─── 2. Target Phase
└─────────────┬─────────────┘
              │
              │
              │
    1. Capture Phase (down)
```

## 📋 Event Types

Orbit supports several categories of events:

### User Input Events

- **Pointer Events**: `click`, `dblclick`, `pointerdown`, `pointerup`, `pointermove`, `pointerenter`, `pointerleave`
- **Keyboard Events**: `keydown`, `keyup`, `keypress`
- **Touch Events**: `touchstart`, `touchmove`, `touchend`, `touchcancel`
- **Form Events**: `change`, `input`, `submit`, `focus`, `blur`

### Component Lifecycle Events

- **Initialization Events**: `mounted`, `initialized`
- **Update Events**: `beforeupdate`, `updated`
- **Destruction Events**: `beforedestroy`, `destroyed`

### Custom Events

Define your own events to communicate between components:

```rust
use orbitrs::events::{Event, EventData};

// Define custom event data
struct ItemSelectedEvent {
    id: String,
    value: i32,
}

// Implement EventData for your custom data
impl EventData for ItemSelectedEvent {}

// Create and dispatch the event
let event = Event::new("itemselected", ItemSelectedEvent { 
    id: "item-1".to_string(), 
    value: 42 
});
component.dispatch_event(event);
```

## 🧩 Event Handling in Components

### In `.orbit` Files

```orbit
<template>
  <button @click="increment">
    Count: {{ count }}
  </button>
</template>

<script>
use orbitrs::prelude::*;

#[component]
pub struct Counter {
    count: State<i32>,
}

impl Counter {
    pub fn new() -> Self {
        Self {
            count: State::new(0),
        }
    }
    
    // Event handler method
    pub fn increment(&mut self, _event: &ClickEvent) {
        self.count.set(*self.count.get() + 1);
    }
}
</script>
```

### Using Event Modifiers

Orbit provides event modifiers to customize event behavior:

```orbit
<template>
  <!-- Stop propagation -->
  <button @click.stop="handleClick">Click Me</button>
  
  <!-- Prevent default behavior -->
  <form @submit.prevent="handleSubmit">...</form>
  
  <!-- Only trigger once -->
  <div @mouseover.once="showTooltip">...</div>
  
  <!-- Key modifiers -->
  <input @keydown.enter="submit">
  
  <!-- Combined modifiers -->
  <div @click.stop.prevent="doSomething">...</div>
</template>
```

## 🔄 Event Delegation

Orbit uses event delegation for efficient event handling, particularly for lists and repeated items. Instead of attaching listeners to each item, attach a single listener to a parent element:

```orbit
<template>
  <ul @click="handleItemClick">
    <li v-for="item in items" data-id="{{ item.id }}">
      {{ item.name }}
    </li>
  </ul>
</template>

<script>
use orbitrs::prelude::*;

#[component]
pub struct ItemList {
    items: State<Vec<Item>>,
}

impl ItemList {
    pub fn handle_item_click(&self, event: &ClickEvent) {
        // Get the item ID from the clicked element
        if let Some(target) = event.target() {
            if let Some(id) = target.get_attribute("data-id") {
                println!("Clicked item with ID: {}", id);
            }
        }
    }
}
</script>
```

## 🌐 Cross-Platform Event Handling

Orbit abstracts platform-specific event systems into a unified API. This means you can use the same event handling code regardless of whether your application runs on:

- Web (via WebAssembly)
- Desktop (Windows, macOS, Linux)
- Mobile (iOS, Android)
- Embedded systems

Behind the scenes, Orbit maps these events to the appropriate platform-specific events:

| Orbit Event   | Web                | Desktop          | Mobile           |
|---------------|--------------------|-----------------:|------------------|
| `click`       | `click`            | `mouseup`        | `touchend`       |
| `pointerdown` | `pointerdown`      | `mousedown`      | `touchstart`     |
| `change`      | `change`           | Custom impl      | Custom impl      |

## 🧪 Event Testing

Orbit provides utilities for testing event handlers:

```rust
use orbitrs::testing::*;
use orbitrs::events::*;

#[test]
fn test_click_handler() {
    // Create a test component
    let mut component = TestComponent::<Counter>::new();
    
    // Assert initial state
    assert_eq!(*component.get_state::<i32>("count"), 0);
    
    // Simulate click event
    component.dispatch_event(Event::new("click", ClickEvent::default()));
    
    // Assert state after event
    assert_eq!(*component.get_state::<i32>("count"), 1);
}
```

## 🔒 Best Practices

1. **Delegate events** whenever possible for better performance
2. **Use appropriate event types** - don't use click events when you need hover events
3. **Be cautious with event.stopPropagation()** - it might prevent other necessary event handlers from executing
4. **Keep handlers small and focused** - extract complex logic into separate methods
5. **Consider accessibility** - ensure keyboard events accompany mouse/touch events for accessibility

## 🔄 Related Topics

- [State Management](./state-management.md) - Learn how to manage and update component state
- [Component Model](./component-model.md) - Understand how components work in Orbit
- [Rendering Architecture](./rendering-architecture.md) - Learn about the rendering process
