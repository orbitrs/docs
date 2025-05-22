# The .orbit File Format

## Overview

The `.orbit` file format is a unified component format for the Orbit UI Framework, combining markup, styling, and Rust logic into a single file. This approach, inspired by single-file component formats from frameworks like Vue, Blazor, and Svelte, allows developers to encapsulate all aspects of a component in one cohesive unit, promoting better organization and maintainability.

## File Structure

An `.orbit` file consists of three main sections:

1. **Template Section** (`<template>`) - Contains the HTML/XML markup for the component
2. **Style Section** (`<style>`) - Contains the CSS styling for the component
3. **Script Section** (`<code lang="rust">`) - Contains the Rust code for the component

### Basic Template

```orbit
<template>
  <div class="greeting">
    <h1>Hello, {{ name }}!</h1>
    <button @click="increment">Count: {{ count }}</button>
  </div>
</template>

<style>
.greeting {
  font-family: Arial, sans-serif;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

h1 {
  color: #0070f3;
}

button {
  background-color: #0070f3;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}
</style>

<code lang="rust">
use orbit::prelude::*;

pub struct Greeting {
    name: String,
    count: i32,
}

pub struct GreetingProps {
    pub name: String,
}

impl Props for GreetingProps {}

impl Component for Greeting {
    type Props = GreetingProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            name: props.name,
            count: 0,
        }
    }
    
    fn render(&self) -> Node {
        // In a real implementation, this would be generated from the template
        node!("div", { class: "greeting" }, [
            node!("h1", {}, [text!("Hello, {}!", self.name)]),
            node!("button", { onclick: self.increment }, [text!("Count: {}", self.count)])
        ])
    }
}

impl Greeting {
    pub fn increment(&mut self) {
        self.count += 1;
    }
}
</code>
```

## Template Syntax

The template section uses an HTML-like syntax with added directives and interpolation features for dynamic content.

### Text Interpolation

Use double curly braces to interpolate Rust expressions into text content:

```html
<p>Hello, {{ user.name }}!</p>
<div>Total: {{ calculate_total() }}</div>
```

### Attributes

There are several ways to bind values to attributes:

- **Static attributes**: `<div class="container">...</div>`
- **Dynamic attributes** with the `:` prefix: `<div :class="containerClass">...</div>`
- **Boolean attributes** that are included based on a condition: `<button :disabled="isLoading">...</button>`
- **Multiple attribute bindings** with object syntax: `<div :class="{ active: isActive, hidden: !isVisible }">...</div>`

### Event Handling

Event handlers are specified using the `@` prefix:

```html
<button @click="increment">Add</button>
<input @input="updateValue" @focus="highlightField" @blur="validateField" />
```

The handlers refer to methods defined in the Rust part of the component.

### Conditionals

You can conditionally render elements using `v-if`, `v-else-if`, and `v-else` directives:

```html
<div v-if="count > 0">Positive</div>
<div v-else-if="count < 0">Negative</div>
<div v-else>Zero</div>
```

### Loops

Use the `v-for` directive for list rendering:

```html
<ul>
  <li v-for="item in items" :key="item.id">{{ item.name }}</li>
</ul>
```

### Components

Reference other components using their PascalCase names:

```html
<UserProfile :user="currentUser" @updated="handleUserUpdate" />
<Card title="Information">
  This is the card content.
</Card>
```

### Slots

Define content insertion points using `<slot>`:

```html
<div class="card">
  <div class="card-header">
    <slot name="header">Default Header</slot>
  </div>
  <div class="card-body">
    <slot>Default Content</slot>
  </div>
</div>
```

Parent components can fill these slots:

```html
<Card>
  <template #header>
    <h3>Custom Header</h3>
  </template>
  
  <p>This content goes into the default slot.</p>
</Card>
```

## Style Section

The `<style>` section contains CSS that applies to the component. By default, styles are automatically scoped to the component to prevent conflicts.

### Scoped Styles

```html
<style>
  .button {
    /* These styles only apply to .button inside this component */
    background-color: blue;
    color: white;
  }
</style>
```

### Global Styles

To create global styles that apply throughout the application, use `scoped="false"`:

```html
<style scoped="false">
  /* These styles will apply globally */
  .global-button {
    background-color: green;
  }
</style>
```

## Code Section

The `<code lang="rust">` section contains the Rust implementation of the component.

### Component Requirements

The script section must:

1. Define a public struct for the component
2. Implement the `Component` trait for the struct
3. Define a Props type and implement the `Props` trait for it
4. Implement required methods like `new` and any others according to the component's needs

### Example Implementation

```rust
use orbit::prelude::*;

// Component data structure
pub struct Counter {
    count: i32,
    step: i32,
}

// Props definition
pub struct CounterProps {
    pub initial: Option<i32>,
    pub step: Option<i32>,
}

impl Props for CounterProps {}

// Component implementation
impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            count: props.initial.unwrap_or(0),
            step: props.step.unwrap_or(1),
        }
    }
    
    fn mounted(&mut self) {
        println!("Counter mounted with initial value: {}", self.count);
    }
    
    fn unmounted(&mut self) {
        println!("Counter unmounted with final value: {}", self.count);
    }
}

// Component methods
impl Counter {
    pub fn increment(&mut self) {
        self.count += self.step;
    }
    
    pub fn decrement(&mut self) {
        self.count -= self.step;
    }
    
    pub fn reset(&mut self) {
        self.count = 0;
    }
}
```

## Multi-file Components

For larger components, you can split an `.orbit` file into multiple files:

- `Component.orbit` - Main component file with the template and basic structure
- `Component.orbit.rs` - Additional Rust logic
- `Component.orbit.css` - Additional CSS styles

The compiler will automatically combine these files during the build process.

## Compilation Process

The `.orbit` file is compiled to Rust code as part of the build process:

1. The template section is parsed and converted to a Rust implementation of the `render` method.
2. Styles are processed and applied with component-specific selectors if scoped.
3. The Rust code is integrated with the generated code from the template.

The `orbiton` CLI tool handles this compilation process, either directly or as part of the standard build workflow.

## Best Practices

- Keep components focused on a single responsibility
- Use props for data and configuration that comes from parent components
- Use local state for data that's specific to the component
- Extract complex logic into separate methods
- Consider splitting large components into smaller subcomponents
- Use slots to create flexible, reusable component templates

## Related Resources

- [Component Model](./component-model.md) - Learn more about the component system
- [State Management](./state-management.md) - In-depth guide to state management
- [Event Handling](./event-handling.md) - Working with events in Orbit
