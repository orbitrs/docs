# Props and Event Handling System

## Overview

The Props and Event Handling System in the Orbit UI Framework provides a robust set of capabilities for managing component properties and UI events. This document outlines the key features and patterns for using these systems effectively.

## Type-safe Props System

The props system allows you to define type-safe, validated properties for your components.

### Key Features

- **Type checking**: All props are strongly typed for compile-time safety
- **Validation**: Props can be validated against custom rules
- **Default values**: Props can have default values
- **Required vs. optional props**: Props can be marked as required

### Example Usage

```rust
// Define props with the macro
define_props! {
    #[derive(Clone, Debug)]
    pub struct ButtonProps {
        pub label: String = "Button",  // With default value
        pub disabled: bool = false,    // With default value
        pub primary: bool = false,     // With default value
        pub on_click: Option<Callback<MouseEvent>> = None, // Optional callback
    }
}

// Create a validator
struct ButtonPropsValidator;

impl PropValidator<ButtonProps> for ButtonPropsValidator {
    fn validate(&self, props: &ButtonProps) -> Result<(), PropValidationError> {
        if props.label.is_empty() {
            return Err(PropValidationError::InvalidValue {
                name: "label".to_string(),
                reason: "Button label cannot be empty".to_string(),
            });
        }
        Ok(())
    }
}

// Use the props with validation
let button_props = ButtonProps::builder()
    .with_validator(ButtonPropsValidator)
    .build()?
    .label("Submit".to_string())
    .primary(true);
```

## Event Delegation System

The event delegation system allows for efficient handling of events throughout the component tree.

### Key Features

- **Event bubbling and capturing**: Events can travel up and down the component tree
- **Event delegation**: Events can be delegated to parent components
- **Stop propagation**: Events can be stopped from further propagation
- **Prevent default**: Default behavior can be prevented

### Event Propagation Phases

1. **Capturing Phase**: Events travel down from parent to target
2. **Target Phase**: Events are handled at the target component
3. **Bubbling Phase**: Events travel up from target to parent

### Example Usage

```rust
// Capture phase (parent to target)
delegate.capture(|event: &mut DelegatedEvent<MouseEvent>| {
    println!("Capturing phase: Button clicked at ({}, {})", event.event.x, event.event.y);
});

// Bubbling phase (target to parent)
delegate.bubble(|event: &mut DelegatedEvent<MouseEvent>| {
    if event.event.event_type == MouseEventType::Click {
        println!("Bubbling phase: Button clicked!");
        
        // Call the on_click callback if provided
        if let Some(on_click) = &self.props.on_click {
            on_click.call(event.event.clone());
        }
    }
});

// Target phase (at the component)
delegate.on(|event: &mut DelegatedEvent<MouseEvent>| {
    // Stop propagation if needed
    event.stop_propagation();
    // Prevent default action if needed
    event.prevent_default();
});
```

## Parent-Child Communication

The framework provides multiple patterns for communication between parent and child components.

### Key Methods

1. **Props Passing**: Parents can pass props to children
2. **Callback Props**: Children can call functions passed by parents
3. **Context System**: Values can be shared down the component tree

### Context System

The context system allows data to be passed down the component tree without prop drilling.

```rust
// In parent component
context.provide(some_value);

// In any child component
if let Some(value) = context.consume() {
    // Use the value
}
```

### Callback Props

Callbacks allow children to communicate with parents.

```rust
// In parent component
let on_submit = context.callback(|data| {
    // Handle data from child
    println!("Received: {:?}", data);
});

// Pass to child component
let child_props = ChildProps {
    on_submit: Some(on_submit),
};

// In child component
if let Some(on_submit) = &self.props.on_submit {
    on_submit.call(data);
}
```

## Best Practices

1. Use the `define_props!` macro for prop definitions
2. Implement validators for props with complex rules
3. Use `build_delegation_tree` to set up event delegation for complex component trees
4. Use capturing phase for preprocessing events
5. Use bubbling phase for default event handling
6. Use callbacks for child-to-parent communication
7. Use context for deep data sharing
