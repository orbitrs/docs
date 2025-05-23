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
  - [Card](#card)
  - [Input](#input)
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

### Card

**Package:** `orbit::kit::components::card`
**Version:** 0.1.0 (Assumed)
**Status:** Beta

A flexible and versatile content container component. It\'s designed to group related information or provide a distinct visual boundary for sections of UI, with customizable appearance through props.

#### Import

```rust
use orbit::kit::components::Card;
```

#### Understanding Props vs. Slots

The `Card` component can be configured using props passed in your `.orbit` templates or when creating components programmatically in Rust. For content, the `Card` utilizes slots: `header`, `footer`, and the default slot for the main content.

While the Rust `CardProps` struct includes a `children: Option<String>` field, this is primarily for scenarios where simple string content is passed programmatically. For richer, structured content, using named slots within your `.orbit` templates is the recommended approach.

#### Examples

##### Basic Usage with Title and Default Content

This example shows a simple card with a title and some text content placed in the default slot.

```orbit
<template>
  <Card title="My Simple Card">
    <p>This is the main content of the card, placed in the default slot.</p>
    <p>You can add multiple elements here.</p>
  </Card>
</template>

<code lang="rust">
use orbit::kit::components::Card;

pub struct MyComponent {}

impl Component for MyComponent {
    // implementation details
}
</code>
```

##### Card with Custom Styling Props

Demonstrates how to use props to customize the card\'s appearance.

```orbit
<template>
  <Card 
    title="Styled Card"
    elevation={3} 
    border_radius="8px" 
    bordered={true} 
    padding="24px"
  >
    <h4>Advanced Styling</h4>
    <p>This card has custom styling applied through various props like elevation, border radius, and padding.</p>
  </Card>
</template>

<code lang="rust">
use orbit::kit::components::Card;

pub struct MyStyledCardPage {}

impl Component for MyStyledCardPage {
    // implementation details
}
</code>
```

##### Card with Header, Footer, and Default Content (Slots)

This example illustrates the use of `header` and `footer` slots for more structured card layouts.

```orbit
<template>
  <Card elevation={2} border_radius="12px" padding="20px">
    <template #header>
      <h2>User Profile</h2>
      <hr />
    </template>

    <!-- This is the default slot content -->
    <p><strong>Name:</strong> Ada Lovelace</p>
    <p><strong>Occupation:</strong> Mathematician, Writer</p>
    
    <template #footer>
      <hr />
      <Button label="View Details" />
      <Button label="Contact" variant="secondary" style="margin-left: 8px;" />
    </template>
  </Card>
</template>

<code lang="rust">
use orbit::kit::components::{Card, Button}; // Assuming Button component is available and registered

pub struct MyProfileView {}

impl Component for MyProfileView {
    // implementation details
}
</code>
```

#### Props

The following props are available for the `Card` component, based on the `CardProps` struct in `orbit/src/kit/components/card.rs`.

| Name            | Type (Rust)    | Required | Default (from `card.rs`) | Description                                                                 |
|-----------------|----------------|----------|--------------------------|-----------------------------------------------------------------------------|
| `title`         | `Option<String>` | ❌        | `None`                   | The title displayed at the top of the card. Typically rendered as a heading. |
| `elevation`     | `Option<u8>`   | ❌        | `1`                      | The shadow level of the card (e.g., 0-5). Controls the perceived depth.     |
| `border_radius` | `Option<String>` | ❌        | `"4px"`                  | The border radius of the card (e.g., "4px", "8px", "0.5rem").               |
| `bordered`      | `Option<bool>` | ❌        | `false`                  | If `true`, the card will have a visible border.                             |
| `padding`       | `Option<String>` | ❌        | `"16px"`                 | The internal padding of the card (e.g., "16px", "1rem").                    |
| `children`      | `Option<String>` | ❌        | `None`                   | Primarily for simple string content when creating Card programmatically in Rust. For complex content, use slots in `.orbit` templates. This content goes into the default slot. |

*Note: While the Rust `render()` method for `Card` in `card.rs` is a placeholder, the Orbit runtime is responsible for interpreting these props and slot content to render the final HTML structure.*

#### Events

No specific events are directly emitted by the `Card` component itself. Events would typically originate from interactive child components placed within the card\'s slots (e.g., a `Button` click).

#### Slots

The `Card` component supports the following slots for content projection:

| Name      | Props | Description                                                                                                |
|-----------|-------|------------------------------------------------------------------------------------------------------------|
| `default` | -     | The main content area of the card. Any content not placed in a named slot will render here.                |
| `header`  | -     | Optional slot for a custom header or title area. Content here appears above the default slot content.      |
| `footer`  | -     | Optional slot for a custom footer area. Content here appears below the default slot content.               |

#### Methods

No public methods are exposed by the `Card` component for direct programmatic interaction.

#### Style API

Customize the appearance of the `Card` component using these CSS custom properties (variables). You can override them in your global stylesheets or scoped component styles.

| CSS Variable                | Default (from props or typical)  | Description                                                                    |
|-----------------------------|----------------------------------|--------------------------------------------------------------------------------|
| `--card-padding`            | `16px` (controlled by `padding` prop) | Internal padding of the card.                                                  |
| `--card-background`         | `#ffffff` (typical theme default)  | Background color of the card.                                                  |
| `--card-border-radius`      | `4px` (controlled by `border_radius` prop) | Border radius of the card.                                                     |
| `--card-border-color`       | `#e0e0e0` (typical if `bordered`) | Color of the border when the `bordered` prop is true.                          |
| `--card-border-width`       | `1px` (typical if `bordered`)    | Width of the border when the `bordered` prop is true.                          |
| `--card-shadow-elevation-0` | `none` (example)                 | Shadow for elevation 0 (no shadow).                                            |
| `--card-shadow-elevation-1` | `0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24)` (example) | Shadow for elevation 1. Specific values are theme-dependent.                   |
| `--card-shadow-elevation-2` | `0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23)` (example) | Shadow for elevation 2.                                                        |
| `--card-shadow-elevation-3` | `0 10px 20px rgba(0,0,0,0.19), 0 6px 6px rgba(0,0,0,0.23)` (example) | Shadow for elevation 3.                                                        |
| `--card-title-color`        | `#333333` (typical theme default)  | Color of the card title text (if using the `title` prop).                      |
| `--card-title-font-size`    | `1.25rem` (typical theme default)  | Font size of the card title text.                                              |
| `--card-title-margin-bottom`| `0.75rem` (typical theme default)  | Margin below the card title.                                                   |
| `--card-header-padding`     | `inherit` (or specific value)    | Padding for the header slot, if different from main card padding.              |
| `--card-footer-padding`     | `inherit` (or specific value)    | Padding for the footer slot, if different from main card padding.              |

##### CSS Classes

These are some of the CSS classes that might be applied to the `Card` component or its internal elements by the Orbit runtime. Styling should primarily be done via CSS variables, but these can be useful for advanced customization or testing.

| Class                       | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| `.orbit-card`                 | Base class applied to the root element of the card component.               |
| `.orbit-card-bordered`        | Applied when the `bordered` prop is `true`.                                 |
| `.orbit-card-elevation-{n}`   | Applied based on the `elevation` prop (e.g., `.orbit-card-elevation-3`).    |
| `.orbit-card-header`          | Class for the wrapper of the `header` slot content.                         |
| `.orbit-card-content`         | Class for the wrapper of the `default` slot content.                        |
| `.orbit-card-footer`          | Class for the wrapper of the `footer` slot content.                         |
| `.orbit-card-title`           | Class for the element rendering the `title` prop, if used.                  |

#### Accessibility

- **Semantic Structure:** Ensure the card\'s content is logically structured. If the `title` prop is used, it should ideally render as a heading element (e.g., `<h2>`, `<h3>`) appropriate for its context in the page hierarchy.
- **Content Headings:** If the content within the card (default slot) has its own sections, use appropriate heading levels (e.g., `<h4>`, `<h5>`) within that content.
- **Interactivity:** The base `Card` component is a container and not interactive by default. If you make a card interactive (e.g., by wrapping it in an `<a>` tag or adding a click listener to the entire card), ensure it has:
    - An appropriate ARIA `role` (e.g., `link`, `button`, `article`).
    - `tabindex="0"` to be keyboard focusable.
    - Keyboard interaction (e.g., activation with `Enter` or `Space` keys).
    - A descriptive `aria-label` if the card\'s purpose isn\'t clear from its visible content.
- **Landmark Regions:** For complex layouts, a card might represent a distinct section of a page. Consider using ARIA landmark roles like `region` (with an `aria-labelledby` pointing to the card\'s title) if appropriate, but use landmarks sparingly.
- **Clarity of Purpose:** The `title` prop or the content within the `header` slot should clearly indicate the purpose or topic of the card.

#### Related Components
- `UserCard`: A specialized card pre-configured for displaying user profiles, likely built using or similar to the base `Card`.

### Input

**Package:** `orbit::kit::components::input`
**Version:** 0.1.0 (Assumed)
**Status:** Beta

A versatile input component for capturing user text input, supporting various types, validation, and states.

#### Import

```rust
use orbit::kit::components::Input;
```

#### Examples

##### Basic Text Input

```orbit
<template>
  <Input 
    label="Your Name" 
    placeholder="Enter your full name" 
    :value="name" 
    @input="handleNameInput" 
  />
  <p>Current Name: {{ name }}</p>
</template>

<code lang="rust">
use orbit::kit::components::Input;
// Assuming a basic Component setup
// use orbit::prelude::*; 

pub struct MyForm {
    name: String,
}

impl Component for MyForm {
    type Props = (); // No external props for this example component

    fn new(_props: Self::Props) -> Self {
        Self { name: "".to_string() }
    }
    
    // ... other lifecycle methods if needed ...

    fn handle_name_input(&mut self, new_value: String) {
        self.name = new_value;
        self.update(); // Request re-render if state is managed by this component
    }
}
</code>
```

##### Input with Validation and Helper Text

This example demonstrates using the `error` and `helper_text` props, along with `required` and type-specific validation.

```orbit
<template>
  <Input 
    input_type="email" 
    label="Email Address" 
    placeholder="you@example.com" 
    :value="email" 
    :error="email_error"
    helper_text="We'll never share your email."
    @input="handleEmailInput" 
    required={true}
  />
  <Button label="Submit" @click="handleSubmit" style="margin-top: 10px;" />
</template>

<code lang="rust">
use orbit::kit::components::{Input, Button}; // Assuming Button is available and registered
// Assuming a basic Component setup
// use orbit::prelude::*;

pub struct SignUpForm {
    email: String,
    email_error: Option<String>,
}

impl Component for SignUpForm {
    type Props = (); // No external props for this example component

    fn new(_props: Self::Props) -> Self {
        Self {
            email: "".to_string(),
            email_error: None,
        }
    }

    // ... other lifecycle methods if needed ...

    fn handle_email_input(&mut self, new_value: String) {
        self.email = new_value;
        // Live validation
        if self.email.is_empty() { // Check based on required prop if passed to component
            self.email_error = Some("Email is required.".to_string());
        } else if !self.email.contains('@') {
            self.email_error = Some("Please enter a valid email address (e.g., user@example.com).".to_string());
        } else {
            self.email_error = None;
        }
        self.update(); // Request re-render
    }

    fn handle_submit(&mut self) {
        // Final validation check on submit
        self.handle_email_input(self.email.clone()); // Re-run validation logic

        if self.email_error.is_none() && !self.email.is_empty() {
            println!("Form submitted with email: {}", self.email);
            // Proceed with actual form submission logic
        } else {
            println!("Form has errors. Cannot submit.");
            // Optionally, ensure UI updates if not already handled by live validation
            self.update();
        }
    }
}
</code>
```

#### Props

Based on the `InputProps` struct in `orbit/src/kit/components/input.rs`.

| Name          | Type (Rust)             | Optional | Default      | Description                                                                 |
|---------------|-------------------------|----------|--------------|-----------------------------------------------------------------------------|
| `id`          | `Option<String>`        | ✅        | `None`       | Unique identifier for the input, crucial for associating with a `<label>`.    |
| `name`        | `Option<String>`        | ✅        | `None`       | Name of the input field, used when submitting as part of a form.            |
| `input_type`  | `String`                | ✅        | `"text"`     | Type of input (e.g., "text", "password", "email", "number", "search", "tel", "url"). |
| `value`       | `Option<String>`        | ✅        | `None`       | The current value of the input field. Bind this for controlled component behavior. |
| `placeholder` | `Option<String>`        | ✅        | `None`       | Placeholder text displayed within the input when it is empty.                 |
| `disabled`    | `Option<bool>`          | ✅        | `false`      | If `true`, the input field is disabled and cannot be interacted with.         |
| `readonly`    | `Option<bool>`          | ✅        | `false`      | If `true`, the input field's value cannot be edited by the user.            |
| `required`    | `Option<bool>`          | ✅        | `false`      | If `true`, the input field must be filled out for form validation.            |
| `label`       | `Option<String>`        | ✅        | `None`       | A visible label associated with the input field.                            |
| `error`       | `Option<String>`        | ✅        | `None`       | An error message displayed below the input, indicating a validation failure. |
| `helper_text` | `Option<String>`        | ✅        | `None`       | Supplementary text displayed below the input to provide guidance or context. |
| `class`       | `Option<String>`        | ✅        | `None`       | Additional CSS classes to apply to the input component's root wrapper.      |
| `style`       | `Option<String>`        | ✅        | `None`       | Inline CSS styles to apply to the input component's root wrapper.           |
| `on_change`   | `Option<Callback<String>>`| ✅        | `None`       | Callback triggered when the input value changes *and the element loses focus*. Provides the new value. |
| `on_input`    | `Option<Callback<String>>`| ✅        | `None`       | Callback triggered *synchronously* whenever the input value changes. Provides the new value. Ideal for live updates. |
| `on_focus`    | `Option<Callback<FocusEvent>>`| ✅    | `None`       | Callback triggered when the input field gains focus. `FocusEvent` provides event details. |
| `on_blur`     | `Option<Callback<FocusEvent>>`| ✅     | `None`       | Callback triggered when the input field loses focus. `FocusEvent` provides event details. |

*Note: `FocusEvent` is assumed to be a type like `web_sys::FocusEvent` or a custom Orbit event wrapper, providing details about the focus event.*

#### Events

The primary way to interact with the `Input` component is through the event callbacks passed as props (`on_change`, `on_input`, `on_focus`, `on_blur`), as detailed in the props table above. These allow for responding to user interactions and changes in the input's state.

#### Slots

The `Input` component does not currently define named slots for content like prefixes or suffixes. Customization of appearance, including icons, would typically be handled via CSS or by composing the `Input` component within a custom layout structure if more complex adornments are needed.

#### Methods

No specific public methods are exposed by the `Input` component for direct programmatic interaction (e.g., `input.focus()`). To interact with the underlying DOM element, you would typically manage focus or selection via standard web platform APIs if you have a reference to the element. This is often achieved through features of the templating system or by managing DOM references in your component logic if Orbit provides such mechanisms.

#### Style API

Customize the appearance of the `Input` component using these CSS custom properties (variables). You can override them in your global stylesheets or scoped component styles.

| CSS Variable                  | Default (Example)               | Description                                                              |
|-------------------------------|---------------------------------|--------------------------------------------------------------------------|
| `--input-padding-y`           | `0.5rem`                        | Vertical padding of the input field.                                     |
| `--input-padding-x`           | `0.75rem`                       | Horizontal padding of the input field.                                   |
| `--input-background`          | `#ffffff`                       | Background color of the input.                                           |
| `--input-text-color`          | `#2d3748` (dark gray)          | Text color inside the input.                                             |
| `--input-placeholder-color`   | `#a0aec0` (medium gray)        | Color of the placeholder text.                                           |
| `--input-border-color`        | `#cbd5e0` (light gray)         | Border color of the input.                                               |
| `--input-border-radius`       | `0.25rem` (4px)                 | Border radius of the input.                                              |
| `--input-border-width`        | `1px`                           | Border width of the input.                                               |
| `--input-font-size`           | `1rem`                          | Font size of the text within the input.                                  |
| `--input-line-height`         | `1.5`                           | Line height of the text within the input.                                |
| `--input-focus-border-color`  | `var(--orbit-primary-color, #3182ce)` | Border color when the input is focused (uses a theme color if available). |
| `--input-focus-shadow`        | `0 0 0 3px rgba(66, 153, 225, 0.5)` | Box shadow when the input is focused (example using a blue tint).        |
| `--input-disabled-background` | `#edf2f7` (very light gray)     | Background color when disabled.                                          |
| `--input-disabled-text-color` | `#a0aec0` (medium gray)        | Text color when disabled.                                                |
| `--input-disabled-border-color`| `#e2e8f0` (lighter gray)       | Border color when disabled.                                              |
| `--input-error-border-color`  | `var(--orbit-danger-color, #e53e3e)` | Border color when there is an error (uses a theme color if available).   |
| `--input-error-focus-shadow`  | `0 0 0 3px rgba(229, 62, 62, 0.5)` | Box shadow when an erroneous input is focused (example using a red tint).|
| `--input-label-color`         | `#4a5568` (gray)               | Color for the label text.                                                |
| `--input-label-font-size`     | `0.875rem` (14px)               | Font size for the label text.                                            |
| `--input-label-margin-bottom` | `0.25rem` (4px)                 | Margin below the label.                                                  |
| `--input-message-font-size`   | `0.75rem` (12px)                | Font size for helper text and error messages.                            |
| `--input-helper-text-color`   | `#718096` (slate gray)         | Color for the helper text.                                               |
| `--input-error-text-color`    | `var(--orbit-danger-color, #c53030)` | Color for the error message text (uses a theme color if available).      |

*Note: Default values for theme colors like `--orbit-primary-color` or `--orbit-danger-color` should be defined in a global theme stylesheet for these variables to be effective.*

##### CSS Classes

These are some of the CSS classes that might be applied by the Orbit runtime to structure and style the `Input` component. Styling should primarily be done via CSS variables for consistency, but these classes can be targeted for more specific overrides or for end-to-end testing.

| Class                         | Description                                                                    |
|-------------------------------|--------------------------------------------------------------------------------|
| `.orbit-input-container`      | Root container for the entire input group (label, input, messages).            |
| `.orbit-input-label`          | Applied to the `<label>` element.                                              |
| `.orbit-input-control`        | Applied to the actual `<input>` HTML element.                                  |
| `.orbit-input-message-group`  | Container for helper text and error messages below the input.                  |
| `.orbit-input-helper-text`    | Applied to the element displaying helper text.                                 |
| `.orbit-input-error-text`     | Applied to the element displaying an error message.                            |
| `.orbit-input--disabled`      | Modifier class on `.orbit-input-container` when the input is disabled.         |
| `.orbit-input--readonly`      | Modifier class on `.orbit-input-container` when the input is read-only.        |
| `.orbit-input--error`         | Modifier class on `.orbit-input-container` when an error is present.           |
| `.orbit-input--focused`       | Modifier class on `.orbit-input-container` when the input control has focus.   |
| `.orbit-input--has-value`     | Modifier class on `.orbit-input-container` when the input control has a value. |

#### Accessibility

Ensuring the `Input` component is accessible is critical for all users:

- **Labeling:**
    - Always provide a visible label using the `label` prop. This is crucial for screen reader users and improves usability for everyone.
    - The component must internally associate the `<label>` with the `<input>` element using the `for` attribute on the label and a matching `id` on the input. If an `id` is not provided via the `id` prop, the component should ideally generate a unique ID to ensure this association.
- **Descriptive Text:**
    - Use the `helper_text` prop for hints, formatting instructions, or other supplementary information.
    - Use the `error` prop to provide clear, concise, and actionable error messages when validation fails.
    - These messages (helper text and error message) must be programmatically associated with the input using `aria-describedby`. The component should manage unique IDs for these descriptive elements and correctly link them to the input field.
- **Required Fields:**
    - If the `required` prop is `true`, the component should visually indicate that the field is mandatory (e.g., with an asterisk `*` next to the label).
    - Programmatically, it must set `aria-required="true"` on the `<input>` element.
- **Error Indication & Recovery:**
    - When the `error` prop is set (indicating a validation error), the component should visually highlight the input as invalid (e.g., using a red border, an error icon).
    - It must also set `aria-invalid="true"` on the `<input>` element. The error message itself (linked by `aria-describedby`) provides specific information about the error and how to correct it.
- **Disabled State:**
    - If the `disabled` prop is `true`, the component must set the `disabled` attribute on the `<input>` element. This prevents interaction, removes it from the tab order, and typically grays out the input.
- **Read-Only State:**
    - If the `readonly` prop is `true`, the component must set the `readonly` attribute on the `<input>` element. The user can still focus the input, select its content, and copy it, but cannot change its value.
- **Keyboard Navigation:**
    - The input must be fully focusable and operable using the keyboard alone (e.g., tabbing to the input, typing, using arrow keys within certain input types if applicable).
- **Focus Indication:**
    - A clear and highly visible focus indicator must be present when the input receives keyboard focus. This is often handled by browser defaults but can and should be enhanced using CSS (e.g., via `--input-focus-border-color`, `--input-focus-shadow`) to ensure it meets contrast requirements.
- **Placeholder Text:**
    - Placeholder text (provided via the `placeholder` prop) should only be used as a supplementary hint for the expected data format or an example. It should *not* be used as a replacement for a visible `<label>` element, as placeholder text disappears on input and is not consistently read by all assistive technologies.
- **Color Contrast:**
    - Ensure sufficient color contrast (at least 4.5:1 for normal text, 3:1 for large text) between text and its background, for border colors against adjacent backgrounds, and for focus indicators. This applies to all states (normal, focus, disabled, error).
- **Autocomplete:**
    - Where appropriate (e.g., for common data fields like name, email, address), consider adding the `autocomplete` attribute to the `<input>` element with a suitable value to assist users with autofill. This is not directly a prop but a good practice for inputs within forms.

By adhering to these practices, the `Input` component can be used to create accessible and user-friendly forms.

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
