# Component API Reference

This document provides a comprehensive reference for the Orbit Framework's component system and the OrbitKit component library. It includes detailed documentation on component structure, lifecycle, and usage patterns, along with specific API references for standard components.

## Table of Contents

- [Component API Overview](#component-api-overview)
  - [Component Structure](#component-structure)
  - [Component Lifecycle](#component-lifecycle)
  - [Props System](#props-system)
  - [Events and Callbacks](#events-and-callbacks)
  - [Component Registration](#component-registration)
- [OrbitKit Component Library](#orbitkit-component-library)
  - [Component Inventory](#component-inventory)
  - [Installation and Setup](#installation-and-setup)
- [Standard Components](#standard-components)
  - [Button](#button)
  - [UserCard](#usercard)
- [Custom Component Development](#custom-component-development)
  - [Component Composition](#component-composition)
  - [Integration with OrbitKit](#integration-with-orbitkit)

## Component API Overview

### Component Structure

Orbit components are defined using the `.orbit` file format, which combines markup (HTML templates), styling (CSS), and behavior (Rust code) in a single file. Each component must implement the `Component` trait from the Orbit core library.

```rust
use orbit::prelude::*;

pub struct MyComponent {
    prop_value: String,
    internal_state: i32,
}

impl Component for MyComponent {
    type Props = MyComponentProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            prop_value: props.value,
            internal_state: 0,
        }
    }
    
    fn mounted(&mut self) {
        // Component is now in the DOM
    }
    
    fn updated(&mut self) {
        // Component has re-rendered
    }
    
    fn unmounted(&mut self) {
        // Component is about to be removed from the DOM
    }
}
```

### Component Lifecycle

Components in Orbit follow a predictable lifecycle:

1. **Creation**: `fn new(props: Self::Props) -> Self`
   - Component is instantiated with initial props
   - Initial state is established

2. **Mounting**: `fn mounted(&mut self)`
   - Component is added to the DOM
   - Ideal for initialization requiring DOM access
   - Setup subscriptions, timers, or external resources

3. **Updating**: `fn updated(&mut self)`
   - Called after component re-renders due to state/prop changes
   - Access to the updated DOM
   - Opportunity to integrate with external libraries

4. **Unmounting**: `fn unmounted(&mut self)`
   - Component is about to be removed from the DOM
   - Clean up subscriptions, timers, or resources

### Props System

Props define the public API of a component, specifying what inputs it can accept from parent components. In Orbit, props are defined as a struct that implements the `Props` trait.

```rust
#[derive(Props)]
pub struct MyComponentProps {
    pub value: String,
    #[prop(default = "Default text")]
    pub label: String,
    #[prop(default)]
    pub disabled: bool,
    #[prop(optional)]
    pub on_change: Option<Callback<String>>,
}
```

#### Prop Types

Orbit supports the following prop types:

- **String**: For text content
- **Number**: For numeric values (i32, i64, f32, f64)
- **Boolean**: For true/false flags
- **Array**: For lists of values
- **Object**: For structured data
- **Callback**: For event handling

### Events and Callbacks

Components can emit events using callbacks passed as props:

```rust
// In the parent component
#[derive(Props)]
pub struct ParentProps {}

pub struct ParentComponent {
    value: String,
}

impl Component for ParentComponent {
    // ...
    
    fn render(&self) -> Node {
        html! {
            <MyComponent 
                value="Initial value"
                on_change={self.callback(|new_value| {
                    self.value = new_value;
                    self.update();
                })}
            />
        }
    }
}

// In the child component
impl MyComponent {
    fn handle_internal_change(&mut self, new_value: String) {
        self.internal_value = new_value.clone();
        
        // Emit change event to parent
        if let Some(on_change) = &self.props.on_change {
            on_change.emit(new_value);
        }
    }
}
```

### Component Registration

To use custom components in templates, they need to be registered with the Orbit runtime:

```rust
// In your main application entry point
fn main() {
    let mut app = OrbitApp::new();
    
    // Register components
    app.register_component::<Button>("Button");
    app.register_component::<UserCard>("UserCard");
    app.register_component::<MyComponent>("MyComponent");
    
    // Start the application
    app.run();
}
```

## Core Component Library

Orbit includes a built-in component library providing a set of pre-built, accessible, and customizable UI components.

### Component Inventory

| Component | Description | Status |
|-----------|-------------|--------|
| Button | Standard button with various styles and states | Stable |
| UserCard | Card displaying user information | Stable |
| Input | Text input field | Beta |
| Card | Generic content container | Beta |
| Dropdown | Dropdown selection menu | Experimental |
| Modal | Dialog/modal window | Experimental |

### Usage

The core components are available directly from the orbit crate:

```rust
// In your Cargo.toml
[dependencies]
orbit = "0.1.0"

// In your code
use orbit::components::{Button, UserCard};
```

### Registration

Register components in your application:

```rust
fn main() {
    let mut app = OrbitApp::new();
    
    // Register the built-in components
    app.register_core_components();
    
    // Or register individual components as needed
    app.register_component::<Button>("Button");
    app.register_component::<UserCard>("UserCard");
    
    // Start the application
    app.run();
}
```

## Standard Components

### Button

**Package:** `orbitkit::components::button`
**Version:** 0.1.0
**Status:** Stable

A versatile button component supporting various styles, states, and events.

#### Import

```rust
use orbitkit::components::Button;
```

#### Examples

##### Basic Usage

```orbit
<template>
  <Button label="Click Me" @click="handleClick" />
</template>

<code lang="rust">
use orbitkit::components::Button;

pub struct MyComponent {}

impl Component for MyComponent {
    // implementation details
    
    fn handle_click(&mut self) {
        println!("Button clicked!");
    }
}
</code>
```

##### Advanced Usage

```orbit
<template>
  <Button 
    :label="computed_label"
    variant="primary"
    :disabled="is_submitting"
    @click="handleSubmit">
    <template #icon>
      <svg><!-- SVG icon content --></svg>
    </template>
  </Button>
</template>
```

#### Props

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `label` | `String` | ✅ | - | Text display on the button |
| `disabled` | `bool` | ❌ | `false` | Whether the button is disabled |
| `variant` | `ButtonVariant` | ❌ | `ButtonVariant::Default` | Button style variant: "default", "primary", "secondary", "danger" |
| `size` | `ButtonSize` | ❌ | `ButtonSize::Medium` | Button size: "small", "medium", "large" |
| `icon_position` | `IconPosition` | ❌ | `IconPosition::Start` | Position of the icon: "start", "end" |

#### Events

| Name | Payload Type | Description |
|------|--------------|-------------|
| `click` | `MouseEvent` | Triggered when the button is clicked |
| `focus` | `FocusEvent` | Triggered when the button receives focus |
| `blur` | `FocusEvent` | Triggered when the button loses focus |

#### Slots

| Name | Props | Description |
|------|-------|-------------|
| `default` | - | Default content slot (overrides label prop) |
| `icon` | - | Custom icon content |

#### Methods

| Name | Signature | Description |
|------|-----------|-------------|
| `click` | `fn click(&self)` | Programmatically click the button |
| `focus` | `fn focus(&self)` | Programmatically focus the button |

#### Style API

| CSS Variable | Default | Description |
|--------------|---------|-------------|
| `--button-color` | `#ffffff` | Text color |
| `--button-background` | `#0070f3` | Background color (for primary variant) |
| `--button-border-radius` | `0.25rem` | Border radius |
| `--button-padding` | `0.5rem 1rem` | Button padding |

##### CSS Classes

| Class | Description |
|-------|-------------|
| `.btn-primary` | Applied for primary variant |
| `.btn-secondary` | Applied for secondary variant |
| `.btn-danger` | Applied for danger variant |
| `.btn-disabled` | Applied when button is disabled |

#### Accessibility

This component follows the [WAI-ARIA Button Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/button/). It implements the following ARIA attributes:

- `role="button"` (when semantically appropriate)
- `aria-disabled="true"` (when disabled)
- `aria-pressed="true|false"` (for toggle buttons)

The component is keyboard accessible and can be activated with Space or Enter keys.

### UserCard

**Package:** `orbitkit::components::user_card`
**Version:** 0.1.0
**Status:** Stable

A card component for displaying user information with customizable layout and fields.

#### Import

```rust
use orbitkit::components::UserCard;
```

#### Examples

##### Basic Usage

```orbit
<template>
  <UserCard :user="current_user" />
</template>

<code lang="rust">
use orbitkit::components::UserCard;
use orbitkit::models::User;

pub struct MyComponent {
    current_user: User,
}

impl Component for MyComponent {
    // implementation details
    
    fn mounted(&mut self) {
        // Example of loading user data
        self.current_user = User {
            name: "Jane Doe".to_string(),
            age: 32,
            // other user properties...
        };
        self.update();
    }
}
</code>
```

##### Advanced Usage

```orbit
<template>
  <UserCard 
    :user="selected_user"
    variant="detailed"
    :show_actions="true"
    @select="handleUserSelect">
    <template #actions>
      <Button label="Edit" @click="handleEditUser" />
      <Button label="Delete" variant="danger" @click="handleDeleteUser" />
    </template>
  </UserCard>
</template>
```

#### Props

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `user` | `User` | ✅ | - | User data object with name, age, etc. |
| `variant` | `UserCardVariant` | ❌ | `UserCardVariant::Standard` | Card display variant: "standard", "compact", "detailed" |
| `show_avatar` | `bool` | ❌ | `true` | Whether to display the user avatar |
| `show_actions` | `bool` | ❌ | `false` | Whether to show the actions area |
| `tags` | `Vec<String>` | ❌ | `vec![]` | List of tags to display on the card |

#### Events

| Name | Payload Type | Description |
|------|--------------|-------------|
| `select` | `User` | Triggered when the card is selected/clicked |

#### Slots

| Name | Props | Description |
|------|-------|-------------|
| `avatar` | `{ user: User }` | Custom avatar content |
| `actions` | `{ user: User }` | Custom action buttons |
| `header` | `{ user: User }` | Custom header content |
| `footer` | `{ user: User }` | Custom footer content |

#### Methods

| Name | Signature | Description |
|------|-----------|-------------|
| `select` | `fn select(&self)` | Programmatically select the user card |

#### Style API

| CSS Variable | Default | Description |
|--------------|---------|-------------|
| `--usercard-padding` | `1rem` | Card padding |
| `--usercard-background` | `#ffffff` | Card background color |
| `--usercard-border-radius` | `0.5rem` | Card border radius |
| `--usercard-shadow` | `0 2px 4px rgba(0,0,0,0.1)` | Card shadow |

##### CSS Classes

| Class | Description |
|-------|-------------|
| `.usercard-detailed` | Applied for detailed variant |
| `.usercard-compact` | Applied for compact variant |
| `.usercard-selected` | Applied when the card is selected |

#### Accessibility

This component follows the [WAI-ARIA Card Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/card/). It implements:

- Semantic HTML elements for content structure
- Proper heading hierarchy
- `tabindex="0"` for keyboard focusability
- `aria-selected="true|false"` when used in a selectable context

## Custom Component Development

### Component Composition

Orbit encourages component composition for building complex UIs:

```orbit
<template>
  <Card>
    <UserCard 
      :user="user" 
      variant="compact" 
      :show_actions="false" 
    />
    <div class="actions">
      <Button label="View Details" @click="viewDetails" />
      <Button label="Message" @click="messageUser" />
    </div>
  </Card>
</template>
```

### Integration with OrbitKit

To extend or customize OrbitKit components:

1. **Composition**: Wrap OrbitKit components in your own components
   ```rust
   pub struct CustomButton {
       label: String,
       custom_prop: String,
   }
   
   impl Component for CustomButton {
       // Implementation
       
       fn render(&self) -> Node {
           html! {
               <Button 
                   label={self.label.clone()}
                   variant="primary"
                   class="custom-button"
               >
                   <span class="custom-content">{self.custom_prop.clone()}</span>
               </Button>
           }
       }
   }
   ```

2. **Inheritance**: Extend OrbitKit component classes
   ```rust
   pub struct EnhancedButton {
       button: Button,
       analytics_id: String,
   }
   
   impl Component for EnhancedButton {
       // Implementation details
       
       fn mounted(&mut self) {
           // Add analytics tracking
           self.track_render(self.analytics_id.clone());
       }
   }
   ```

3. **Theme Customization**: Override CSS variables
   ```css
   :root {
     --button-background: #ff6b6b;
     --button-color: #1a1a2e;
     --button-border-radius: 0;
   }
   ```

For more detailed information on component development, see the [Advanced Component Patterns](../core-concepts/advanced-component-patterns.md) guide.
