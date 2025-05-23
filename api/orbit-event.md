# `orbit::event` API Reference

This document provides a comprehensive reference for the event system in the Orbit UI Framework.

## Overview

The `orbit::event` module provides a cross-platform event handling system that works consistently across different target platforms (Web, Native, and Embedded). The module includes core event types, event handling utilities, and an event propagation system.

## Core Types

### `Event`

```rust
pub trait Event: Any + Send + Sync {}
```

A type-erased event that can be sent through the event system. Any type that is `Any + Send + Sync` automatically implements this trait.

### `EventEmitter`

```rust
#[derive(Default, Clone)]
pub struct EventEmitter<T: Event = ()> {
    // Internal fields
}
```

Manages event listeners and event dispatch for a specific event type.

#### Methods

```rust
impl<T: Event> EventEmitter<T> {
    /// Create a new event emitter
    pub fn new() -> Self;
    
    /// Add an event listener
    pub fn add_listener<F>(&self, callback: F) -> ListenerId 
    where F: Fn(&T) + Send + Sync + 'static;
    
    /// Remove a listener by its ID
    pub fn remove_listener(&self, id: ListenerId);
    
    /// Emit an event to all registered listeners
    pub fn emit(&self, event: &T);
    
    /// Check if this emitter has any listeners
    pub fn has_listeners(&self) -> bool;
    
    /// Clear all listeners
    pub fn clear(&self);
}
```

### `EventBus`

```rust
pub struct EventBus {
    // Internal fields
}
```

A central hub for application-wide event distribution.

#### Methods

```rust
impl EventBus {
    /// Create a new event bus
    pub fn new() -> Self;
    
    /// Subscribe to events of a specific type
    pub fn subscribe<T: Event, F>(&self, callback: F) -> ListenerId
    where F: Fn(&T) + Send + Sync + 'static;
    
    /// Unsubscribe using a listener ID
    pub fn unsubscribe(&self, id: ListenerId);
    
    /// Publish an event to all subscribers
    pub fn publish<T: Event>(&self, event: &T);
}
```

## Event Types

### Mouse Events

```rust
#[derive(Debug, Clone)]
pub struct MouseEvent {
    /// X-coordinate relative to the component
    pub x: f32,
    /// Y-coordinate relative to the component
    pub y: f32,
    /// X-coordinate relative to the screen
    pub screen_x: f32,
    /// Y-coordinate relative to the screen
    pub screen_y: f32,
    /// Mouse button that was pressed/released
    pub button: MouseButton,
    /// Type of mouse event
    pub event_type: MouseEventType,
    /// Modifier keys pressed during the event
    pub modifiers: KeyModifiers,
}
```

Mouse event types:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum MouseEventType {
    Down,
    Up,
    Move,
    Enter,
    Leave,
    Click,
    DoubleClick,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum MouseButton {
    Left,
    Middle,
    Right,
    Other(u8),
}
```

### Keyboard Events

```rust
#[derive(Debug, Clone)]
pub struct KeyboardEvent {
    /// Key that was pressed/released
    pub key: Key,
    /// Key code value
    pub key_code: u32,
    /// Type of keyboard event
    pub event_type: KeyboardEventType,
    /// Modifier keys pressed during the event
    pub modifiers: KeyModifiers,
}
```

Keyboard event types:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum KeyboardEventType {
    Down,
    Up,
    Press,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct KeyModifiers {
    pub shift: bool,
    pub ctrl: bool,
    pub alt: bool,
    pub meta: bool,
}
```

### Touch Events

```rust
#[derive(Debug, Clone)]
pub struct TouchEvent {
    /// Collection of active touch points
    pub touches: Vec<Touch>,
    /// Type of touch event
    pub event_type: TouchEventType,
}

#[derive(Debug, Clone)]
pub struct Touch {
    /// Unique identifier for this touch point
    pub id: u64,
    /// X-coordinate relative to the component
    pub x: f32,
    /// Y-coordinate relative to the component
    pub y: f32,
    /// X-coordinate relative to the screen
    pub screen_x: f32,
    /// Y-coordinate relative to the screen
    pub screen_y: f32,
}
```

Touch event types:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum TouchEventType {
    Start,
    Move,
    End,
    Cancel,
}
```

### Form Events

```rust
#[derive(Debug, Clone)]
pub struct InputEvent {
    /// Current value of the input element
    pub value: String,
    /// Type of the input element
    pub input_type: InputType,
}

#[derive(Debug, Clone)]
pub struct ChangeEvent {
    /// New value after the change
    pub value: String,
    /// Old value before the change
    pub old_value: String,
}

#[derive(Debug, Clone)]
pub struct SubmitEvent {
    /// Form data as key-value pairs
    pub data: HashMap<String, String>,
}
```

### Component Lifecycle Events

```rust
#[derive(Debug, Clone)]
pub struct MountedEvent;

#[derive(Debug, Clone)]
pub struct BeforeUpdateEvent;

#[derive(Debug, Clone)]
pub struct UpdatedEvent;

#[derive(Debug, Clone)]
pub struct BeforeUnmountEvent;

#[derive(Debug, Clone)]
pub struct UnmountedEvent;
```

### Creating Custom Events

You can create custom events by implementing types that fulfill the `Event` trait requirements:

```rust
#[derive(Debug, Clone)]
pub struct ItemSelectedEvent {
    pub id: String,
    pub value: i32,
}

// The Event trait is auto-implemented for any type that meets the requirements
// No explicit implementation needed
```

## Event Handling

### In Component Templates

In `.orbit` template files, use the `@event` syntax to bind event handlers:

```html
<button @click="handle_click">Click Me</button>
<input @input="handle_input" @focus="handle_focus" />
```

The corresponding component implementation:

```rust
impl MyComponent {
    fn handle_click(&mut self, event: &MouseEvent) {
        // Handle click event
    }
    
    fn handle_input(&mut self, event: &InputEvent) {
        // Process input
    }
    
    fn handle_focus(&mut self, _event: &FocusEvent) {
        // Handle focus
    }
}
```

### Event Modifiers

Event modifiers customize event behavior:

```html
<!-- Stop propagation -->
<button @click.stop="handle_click">Click Me</button>

<!-- Prevent default behavior -->
<form @submit.prevent="handle_submit">...</form>

<!-- Only trigger once -->
<div @mouseover.once="show_tooltip">...</div>

<!-- Key modifiers -->
<input @keydown.enter="submit">

<!-- Combined modifiers -->
<div @click.stop.prevent="do_something">...</div>
```

### Event Delegation

For efficient event handling with lists and repeated items, use event delegation:

```rust
impl ItemList {
    pub fn handle_item_click(&self, event: &MouseEvent) {
        // Get target element
        if let Some(target) = event.target() {
            // Check for data attribute
            if let Some(id) = target.get_attribute("data-id") {
                println!("Clicked item with ID: {}", id);
                // Handle the click for the specific item
            }
        }
    }
}
```

## Cross-Platform Event Abstraction

Orbit abstracts platform-specific events into a unified API:

| Orbit Event   | Web                | Desktop          | Mobile           |
|---------------|--------------------|-----------------:|------------------|
| `click`       | `click`            | `mouseup`        | `touchend`       |
| `pointerdown` | `pointerdown`      | `mousedown`      | `touchstart`     |
| `change`      | `change`           | Custom impl      | Custom impl      |

This abstraction ensures your event handling code works consistently across different platforms.

## Event Flow

Events in Orbit follow a capture and bubble flow similar to the DOM:

1. **Capture Phase**: Events travel from the root component down to the target component
2. **Target Phase**: Event reaches the component where the event originated
3. **Bubble Phase**: Events bubble up from the target component to the root

## Event Testing

Orbit provides utilities for testing event handlers:

```rust
#[test]
fn test_click_handler() {
    // Create a test component
    let mut component = TestComponent::<Counter>::new();
    
    // Assert initial state
    assert_eq!(*component.get_state::<i32>("count"), 0);
    
    // Simulate click event
    let event = MouseEvent::new(MouseEventType::Click, 10.0, 20.0);
    component.dispatch_event(event);
    
    // Assert state after event
    assert_eq!(*component.get_state::<i32>("count"), 1);
}
```

## Best Practices

1. **Delegate events** whenever possible for better performance
2. **Use appropriate event types** - don't use click events when you need hover events
3. **Be cautious with event.stopPropagation()** - it might prevent other necessary event handlers from executing
4. **Keep handlers small and focused** - extract complex logic into separate methods
5. **Consider accessibility** - ensure keyboard events accompany mouse/touch events for accessibility

## Related Documentation

- [Component Model](../core-concepts/component-model.md)
- [State Management](../core-concepts/state-management.md)
- [Event Handling](../core-concepts/event-handling.md)
- [Testing Strategies](../guides/testing-strategies.md)
