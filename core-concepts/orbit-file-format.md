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

#[derive(Props)]
pub struct GreetingProps {
    name: String,
}

#[component]
pub fn Greeting(props: GreetingProps) -> impl View {
    let count = use_state(|| 0);
    
    let increment = use_callback(move |_| {
        count.update(|c| *c + 1);
    });
    
    html! {
        <div class="greeting">
            <h1>Hello, {props.name}!</h1>
            <button on:click={increment}>Count: {*count}</button>
        </div>
    }
}
</code>
```

## Template Section (`<template>`)

The template section defines the structure and layout of your component using an HTML-like syntax with Orbit-specific enhancements.

### Data Binding

Bind component data to the template using the `{{ expression }}` syntax:

```orbit
<template>
  <div>
    <p>Current value: {{ value }}</p>
    <p>Computed value: {{ compute_value() }}</p>
  </div>
</template>
```

### Conditional Rendering

Use the `o-if`, `o-else-if`, and `o-else` directives for conditional rendering:

```orbit
<template>
  <div>
    <p o-if="status == 'loading'">Loading...</p>
    <p o-else-if="status == 'error'">Error: {{ error_message }}</p>
    <p o-else>Data loaded successfully</p>
  </div>
</template>
```

### List Rendering

Use the `o-for` directive to render lists:

```orbit
<template>
  <ul>
    <li o-for="item in items" key="item.id">{{ item.name }}</li>
  </ul>
</template>
```

### Event Handling

Handle DOM events using the `@event` syntax:

```orbit
<template>
  <button @click="handle_click">Click me</button>
  <input @input="update_value" />
</template>
```

You can also pass event data:

```orbit
<template>
  <button @click="handle_click($event)">Click me</button>
</template>
```

### Attribute Binding

Bind attributes dynamically:

```orbit
<template>
  <img :src="image_url" :alt="image_description" />
  <button :disabled="is_loading">Submit</button>
</template>
```

### Class and Style Binding

Bind classes and styles dynamically:

```orbit
<template>
  <div :class="{ active: is_active, error: has_error }">Content</div>
  <div :style="{ color: text_color, fontSize: font_size + 'px' }">Styled content</div>
</template>
```

### Slots

Define slots to allow content injection from parent components:

```orbit
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">Default header</slot>
    </div>
    <div class="card-body">
      <slot>Default content</slot>
    </div>
    <div class="card-footer">
      <slot name="footer">Default footer</slot>
    </div>
  </div>
</template>
```

## Style Section (`<style>`)

The style section contains CSS that is scoped to the component by default.

### Basic Styling

```orbit
<style>
.button {
  background-color: #0070f3;
  color: white;
  padding: 8px 16px;
}

.button:hover {
  background-color: #005bb5;
}
</style>
```

### Scoped vs. Global Styles

By default, styles are scoped to the component. For global styles, use the `global` attribute:

```orbit
<style global>
:root {
  --primary-color: #0070f3;
}

body {
  margin: 0;
  padding: 0;
}
</style>
```

You can also mix scoped and global styles:

```orbit
<style>
/* Scoped to this component */
.component {
  color: var(--primary-color);
}

:global(.app-button) {
  /* Applied globally */
  background-color: #0070f3;
}
</style>
```

### Variables and Preprocessors

Use CSS variables for dynamic styling:

```orbit
<style>
.component {
  --text-color: v-bind(textColor);
  color: var(--text-color);
}
</style>
```

Orbit supports CSS preprocessors like SCSS:

```orbit
<style lang="scss">
$primary-color: #0070f3;

.button {
  background-color: $primary-color;
  
  &:hover {
    background-color: darken($primary-color, 10%);
  }
  
  .icon {
    margin-right: 8px;
  }
}
</style>
```

## Script Section (`<code lang="rust">`)

The script section contains the Rust code for your component logic.

### Component Structure

Components can be implemented in two ways:

1. **Function Components** (Recommended):

```rust
use orbit::prelude::*;

#[derive(Props)]
pub struct CounterProps {
    initial: Option<i32>,
}

#[component]
pub fn Counter(props: CounterProps) -> impl View {
    let count = use_state(|| props.initial.unwrap_or(0));
    
    let increment = use_callback(move |_| {
        count.update(|c| *c + 1);
    });
    
    let decrement = use_callback(move |_| {
        count.update(|c| *c - 1);
    });
    
    html! {
        <div>
            <button on:click={decrement}>-</button>
            <span>{*count}</span>
            <button on:click={increment}>+</button>
        </div>
    }
}
```

2. **Class Components**:

```rust
use orbit::prelude::*;

pub struct Counter {
    count: State<i32>,
}

impl Component for Counter {
    type Props = CounterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            count: State::new(props.initial.unwrap_or(0)),
        }
    }
    
    fn view(&self) -> impl View {
        let count = self.count.clone();
        
        html! {
            <div>
                <button on:click={move |_| count.update(|c| *c - 1)}>-</button>
                <span>{*self.count}</span>
                <button on:click={move |_| count.update(|c| *c + 1)}>+</button>
            </div>
        }
    }
}

#[derive(Props)]
pub struct CounterProps {
    initial: Option<i32>,
}
```

### State Management

Manage component state with reactive state:

```rust
use orbit::prelude::*;

#[component]
pub fn FormExample() -> impl View {
    let name = use_state(|| String::new());
    let email = use_state(|| String::new());
    
    let handle_submit = use_callback(|_| {
        println!("Submitted: {} <{}>", *name, *email);
    });
    
    html! {
        <form on:submit={handle_submit}>
            <input 
                type="text" 
                placeholder="Name" 
                value={name.to_string()}
                on:input={move |e: InputEvent| name.set(e.value())}
            />
            <input 
                type="email" 
                placeholder="Email" 
                value={email.to_string()}
                on:input={move |e: InputEvent| email.set(e.value())}
            />
            <button type="submit">Submit</button>
        </form>
    }
}
```

### Lifecycle Hooks

React to component lifecycle events:

```rust
use orbit::prelude::*;

#[component]
pub fn LifecycleExample() -> impl View {
    let count = use_state(|| 0);
    
    use_effect(move || {
        println!("Component mounted");
        
        // Cleanup function (runs on unmount)
        || println!("Component unmounted")
    }, ());
    
    use_effect(move || {
        println!("Count changed to {}", *count);
        
        || {}
    }, *count);
    
    html! {
        <button on:click={move |_| count.update(|c| *c + 1)}>
            Count: {*count}
        </button>
    }
}
```

## Advanced Features

### Multi-file Components

For complex components, you can split the implementation across multiple files:

**counter.orbit**
```orbit
<template>
  <div class="counter">
    <button @click="decrement">-</button>
    <span>{{ count }}</span>
    <button @click="increment">+</button>
  </div>
</template>

<style>
@import "./counter.css";
</style>

<code lang="rust">
mod logic;
use logic::*;

#[component]
pub fn Counter(props: CounterProps) -> impl View {
    let (count, increment, decrement) = use_counter_logic(props.initial);
    
    html! {
        <div class="counter">
            <button on:click={decrement}>-</button>
            <span>{*count}</span>
            <button on:click={increment}>+</button>
        </div>
    }
}

#[derive(Props)]
pub struct CounterProps {
    initial: Option<i32>,
}
</code>
```

**logic.rs**
```rust
use orbit::prelude::*;

pub fn use_counter_logic(initial: Option<i32>) -> (State<i32>, Callback<()>, Callback<()>) {
    let count = use_state(|| initial.unwrap_or(0));
    
    let increment = {
        let count = count.clone();
        use_callback(move |_| {
            count.update(|c| *c + 1);
        })
    };
    
    let decrement = {
        let count = count.clone();
        use_callback(move |_| {
            count.update(|c| *c - 1);
        })
    };
    
    (count, increment, decrement)
}
```

### Custom Directives

Create custom directives for reusable behavior:

```rust
use orbit::prelude::*;

// Define the directive
#[directive]
pub fn tooltip(el: &Element, value: &str) {
    el.set_attribute("title", value);
    // Add tooltip behavior
}

// Use in a component
#[component]
pub fn TooltipExample() -> impl View {
    html! {
        <button o-tooltip="Click to submit">
            Submit
        </button>
    }
}
```

### Server Components

Define server-only components that render on the server:

```orbit
<template>
  <div>
    <p>User profile for {{ user.name }}</p>
    <p>Email: {{ user.email }}</p>
  </div>
</template>

<code lang="rust" server>
use orbit::prelude::*;
use crate::db::User;

#[derive(Props)]
pub struct UserProfileProps {
    user_id: String,
}

#[server_component]
pub async fn UserProfile(props: UserProfileProps) -> ServerResult<impl View> {
    let user = User::fetch_by_id(&props.user_id).await?;
    
    Ok(html! {
        <div>
            <p>User profile for {user.name}</p>
            <p>Email: {user.email}</p>
        </div>
    })
}
</code>
```

## Compilation Process

When you save an `.orbit` file, the Orbit compiler:

1. Parses the file into its three sections
2. Processes the template into Rust code
3. Scopes and processes the CSS
4. Combines with the script code
5. Generates a standard Rust module

The generated code is then compiled as part of your regular Rust build process.

## Best Practices

- Keep components focused on a single responsibility
- Extract complex logic into separate Rust files
- Use CSS variables for theme consistency
- Prefer function components for simpler cases
- Use slots for flexible composition
- Document component props and behavior

## Debugging

You can debug `.orbit` files by:

1. Inspecting the generated Rust code (located in the target directory)
2. Using the Orbit DevTools browser extension
3. Adding console logs or debug statements
4. Using the Rust debugging tools with breakpoints
