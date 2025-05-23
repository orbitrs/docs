---
title: Template Syntax
description: Comprehensive guide to Orbit's template syntax with examples of advanced features and patterns
---

<!-- filepath: /Volumes/EXT/repos/orbitrs/docs/core-concepts/template-syntax.md -->
# Template Syntax

This document provides a comprehensive guide to Orbit's template syntax, covering both basic and advanced features to help you build dynamic, responsive user interfaces.

## Basic Template Structure

Orbit templates use an HTML-like syntax that will be familiar to web developers, while adding powerful reactivity features:

```orbit
<template>
  <div class="container">
    <h1>{{ title }}</h1>
    <p>{{ content }}</p>
  </div>
</template>
```

## Text Interpolation

### Basic Interpolation

Use double curly braces `{{ }}` to display component data:

```orbit
<template>
  <p>Hello, {{ user.name }}!</p>
</template>
```

### Raw HTML

By default, interpolation escapes HTML to prevent XSS attacks. To render raw HTML (use with caution):

```orbit
<template>
  <div v-html="richTextContent"></div>
</template>
```

### Expressions

Interpolation supports expressions:

```orbit
<template>
  <p>{{ user.first_name + ' ' + user.last_name }}</p>
  <p>{{ price * 1.20 }} (including 20% tax)</p>
  <p>{{ isActive ? 'Active' : 'Inactive' }}</p>
</template>
```

## Directives

Directives are special attributes with the `o-` prefix that apply reactive behavior to DOM elements.

### Conditional Rendering

#### o-if, o-else-if, o-else

```orbit
<template>
  <div>
    <p o-if="status === 'loading'">Loading...</p>
    <p o-else-if="status === 'error'">Error: {{ errorMessage }}</p>
    <div o-else>
      <h2>{{ title }}</h2>
      <p>{{ content }}</p>
    </div>
  </div>
</template>
```

#### o-show

Unlike `o-if` which adds/removes elements from the DOM, `o-show` toggles visibility with CSS:

```orbit
<template>
  <div o-show="isVisible">
    This content will be hidden with display: none when isVisible is false
  </div>
</template>
```

### List Rendering

#### o-for

Iterate over arrays or collections:

```orbit
<template>
  <ul>
    <li o-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>
```

With index:

```orbit
<template>
  <ul>
    <li o-for="(item, index) in items" :key="item.id">
      {{ index + 1 }}. {{ item.name }}
    </li>
  </ul>
</template>
```

Iterating over objects:

```orbit
<template>
  <ul>
    <li o-for="(value, key) in userObject" :key="key">
      {{ key }}: {{ value }}
    </li>
  </ul>
</template>
```

Range-based iteration:

```orbit
<template>
  <div>
    <span o-for="n in 5" :key="n">{{ n }} </span>
    <!-- Renders: 1 2 3 4 5 -->
  </div>
</template>
```

### Binding Attributes

#### o-bind (shorthand: `:`)

Dynamically bind HTML attributes:

```orbit
<template>
  <img o-bind:src="imageUrl" o-bind:alt="imageAlt">
  <!-- Shorthand -->
  <img :src="imageUrl" :alt="imageAlt">
</template>
```

Binding multiple attributes with an object:

```orbit
<template>
  <div o-bind="objectOfAttrs">...</div>
</template>
```

### Event Handling

#### o-on (shorthand: `@`)

Listen to DOM events:

```orbit
<template>
  <button o-on:click="handleClick">Click Me</button>
  <!-- Shorthand -->
  <button @click="handleClick">Click Me</button>
</template>
```

Event modifiers:

```orbit
<template>
  <!-- Stop propagation -->
  <button @click.stop="handleClick">Click Me</button>
  
  <!-- Prevent default behavior -->
  <form @submit.prevent="submitForm">...</form>
  
  <!-- Keyboard events -->
  <input @keyup.enter="submit">
  
  <!-- Combining modifiers -->
  <div @click.stop.prevent="handleAction">...</div>
</template>
```

Common event modifiers:
- `.stop` - calls `event.stopPropagation()`
- `.prevent` - calls `event.preventDefault()`
- `.capture` - uses capture phase event handler
- `.self` - only triggers if event.target is the element itself
- `.once` - event listener is removed after first trigger

Key modifiers:
- `.enter`, `.tab`, `.delete`, `.esc`, `.space`, `.up`, `.down`, `.left`, `.right`
- `.ctrl`, `.alt`, `.shift`, `.meta`

### Two-Way Binding

#### o-model

Create two-way binding for form inputs:

```orbit
<template>
  <input o-model="username">
  <p>The username is: {{ username }}</p>
</template>
```

Modifiers for `o-model`:
- `.lazy` - sync after change events instead of input
- `.number` - convert input to number
- `.trim` - trim whitespace

```orbit
<template>
  <!-- Update on change rather than every keystroke -->
  <input o-model.lazy="message">
  
  <!-- Automatically convert to number -->
  <input o-model.number="age" type="number">
  
  <!-- Trim whitespace -->
  <input o-model.trim="username">
</template>
```

## Advanced Template Features

### Template References

Access DOM elements directly with refs:

```orbit
<template>
  <input o-ref="emailInput">
  <button @click="focusInput">Focus Input</button>
</template>

<code lang="rust">
  use orbit::prelude::*;
  use web_sys::HtmlInputElement;
  
  impl MyComponent {
      fn focus_input(&self) {
          if let Some(input) = self.refs.get("emailInput") {
              if let Some(input_elem) = input.dyn_ref::<HtmlInputElement>() {
                  let _ = input_elem.focus();
              }
          }
      }
  }
</code>
```

### Slots

Slots allow content projection from parent to child components:

```orbit
<!-- Parent.orbit -->
<template>
  <Child>
    <p>This content will be projected into the child's slot</p>
  </Child>
  
  <ChildWithNamedSlots>
    <template o-slot:header>
      <h2>Custom Header</h2>
    </template>
    
    <template o-slot:default>
      <p>Default slot content</p>
    </template>
    
    <template o-slot:footer>
      <p>Footer content</p>
    </template>
  </ChildWithNamedSlots>
</template>

<!-- Child.orbit -->
<template>
  <div class="child">
    <slot></slot> <!-- Parent content will go here -->
  </div>
</template>

<!-- ChildWithNamedSlots.orbit -->
<template>
  <div class="child-with-slots">
    <header>
      <slot name="header">
        <!-- Default header content if no slot content provided -->
        <h2>Default Header</h2>
      </slot>
    </header>
    
    <main>
      <slot></slot>
    </main>
    
    <footer>
      <slot name="footer">
        <!-- Default footer content -->
        <p>Default Footer</p>
      </slot>
    </footer>
  </div>
</template>
```

### Scoped Slots

Pass data from child to parent with scoped slots:

```orbit
<!-- Child.orbit -->
<template>
  <div>
    <slot :user="user" :is-active="isActive"></slot>
  </div>
</template>

<!-- Parent.orbit -->
<template>
  <Child>
    <template o-slot="slotProps">
      {{ slotProps.user.name }} is {{ slotProps.isActive ? 'active' : 'inactive' }}
    </template>
  </Child>
</template>
```

### Dynamic Components

Use the `<component>` element to switch between components dynamically:

```orbit
<template>
  <component :is="currentView">
    <!-- Component changes when currentView changes -->
  </component>
  
  <button @click="currentView = 'home'">Home</button>
  <button @click="currentView = 'about'">About</button>
  <button @click="currentView = 'contact'">Contact</button>
</template>
```

### Reusable Template Fragments

Define reusable template fragments with `o-template`:

```orbit
<template>
  <div>
    <o-template id="user-card">
      <div class="user-card">
        <img :src="user.avatar">
        <h3>{{ user.name }}</h3>
        <p>{{ user.bio }}</p>
      </div>
    </o-template>
    
    <o-template-instance template="user-card" 
                          o-for="user in users" 
                          :key="user.id"
                          :user="user">
    </o-template-instance>
  </div>
</template>
```

### Server-Side Rendering Hints

Provide hints for SSR optimization:

```orbit
<template>
  <!-- This will be rendered on the server -->
  <div o-ssr>
    <h1>Welcome to {{ title }}</h1>
  </div>
  
  <!-- This will not be rendered until client-side hydration -->
  <div o-client-only>
    <InteractiveComponent />
  </div>
</template>
```

## Edge Cases and Gotchas

### Multiple Root Elements

Unlike some frameworks, Orbit does allow multiple root elements in templates:

```orbit
<template>
  <header>Page Header</header>
  <main>Main Content</main>
  <footer>Page Footer</footer>
</template>
```

However, when using multiple root elements, be aware that certain features like component transitions might behave differently.

### Template Whitespace Handling

Whitespace in templates is generally preserved, which can sometimes lead to unexpected layouts. Use the following patterns to control whitespace:

```orbit
<template>
  <!-- Compact inline layout -->
  <span>{{ first }}</span><span>{{ second }}</span>
  
  <!-- Alternative compact layout -->
  <span>{{ first }}</span><!--
  --><span>{{ second }}</span>
</template>
```

### Expression Limitations

Template expressions should be simple. For complex logic, use methods or computed properties in your component:

```orbit
<template>
  <!-- Avoid complex expressions like this -->
  <div>{{ items.filter(i => i.isComplete).map(i => i.name).join(', ') }}</div>
  
  <!-- Better approach -->
  <div>{{ completedItemNames }}</div>
</template>

<code lang="rust">
  impl MyComponent {
      fn completed_item_names(&self) -> String {
          self.items
              .iter()
              .filter(|i| i.is_complete)
              .map(|i| i.name.clone())
              .collect::<Vec<_>>()
              .join(", ")
      }
  }
</code>
```

## Best Practices

1. **Keep templates focused**: Large templates are hard to maintain. Break complex UIs into smaller components.

2. **Use computed methods for complex logic**: Keep templates clean by placing complex logic in component methods.

3. **Properly key lists**: Always use `:key` with `o-for` to maintain component identity during updates.

4. **Be cautious with `v-html`**: Only use with trusted content to avoid XSS attacks.

5. **Use directive shorthands consistently**: Either use shorthands (`:prop`, `@event`) or full syntax (`o-bind:prop`, `o-on:event`) consistently throughout your codebase.

## Debugging Templates

When debugging templates, the following techniques are helpful:

1. **Use debug outputs**: Temporarily add `{{ JSON.stringify(someValue) }}` to check values.

2. **Component inspector**: Use the Orbit DevTools to inspect component structure.

3. **Template trace**: Enable template tracing in development:

```rust
// In main.rs
fn main() {
    OrbitApp::new()
        .with_debug(true)
        .with_template_trace(true)
        .mount("#app")
        .run();
}
```

## Template Syntax Reference

| Syntax | Description |
|--------|-------------|
| `{{ expression }}` | Text interpolation |
| `o-if="condition"` | Conditional rendering |
| `o-else-if="condition"` | Alternative condition |
| `o-else` | Fallback condition |
| `o-show="condition"` | Toggle element visibility |
| `o-for="item in items"` | List rendering |
| `o-bind:attr="value"` or `:attr="value"` | Attribute binding |
| `o-on:event="handler"` or `@event="handler"` | Event handling |
| `o-model="variable"` | Two-way binding |
| `o-ref="refName"` | Template reference |
| `<slot>` | Content projection |
| `o-slot:name` | Named slot content |
| `o-ssr` | Server rendering hint |
| `o-client-only` | Client only rendering |

## Further Reading

- [Component Model](/docs/core-concepts/component-model.md)
- [State Management](/docs/core-concepts/state-management.md)
- [Event Handling](/docs/core-concepts/event-handling.md)
- [Advanced Component Patterns](/docs/core-concepts/advanced-component-patterns.md)
