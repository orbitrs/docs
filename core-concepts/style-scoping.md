# Style Scoping in Orbit

This document explains how CSS style scoping works in the Orbit Framework and the various approaches available for styling components.

## Overview

Style scoping is a technique that prevents CSS rules from one component from affecting other components, avoiding style conflicts and making components more modular. Orbit's style scoping automatically ensures that CSS rules defined in a component are only applied to elements within that component.

## How Style Scoping Works

When you include a `<style>` section in an `.orbit` file, the CSS rules are automatically scoped to the component by default. This is done through a combination of:

1. **Unique Component ID** - Each component instance gets a unique identifier
2. **CSS Prefixing** - CSS selectors are modified to include the component ID
3. **Rule Isolation** - Rules can't leak out to affect other components

### Example of Scoped Styles

```orbit
<template>
  <div class="container">
    <h1 class="title">Hello, {{ name }}!</h1>
  </div>
</template>

<style>
  /* These styles only apply to elements in this component */
  .container {
    padding: 20px;
    border: 1px solid #ccc;
  }
  
  .title {
    color: #0070f3;
    font-size: 24px;
  }
</style>
```

Behind the scenes, Orbit transforms the CSS to something like:

```css
.container[data-orbit-component="c-28fj3s9"] {
  padding: 20px;
  border: 1px solid #ccc;
}

.title[data-orbit-component="c-28fj3s9"] {
  color: #0070f3;
  font-size: 24px;
}
```

The HTML elements also receive the appropriate attribute:

```html
<div class="container" data-orbit-component="c-28fj3s9">
  <h1 class="title" data-orbit-component="c-28fj3s9">Hello, User!</h1>
</div>
```

## Opting Out of Style Scoping

There are cases where you might want styles to apply globally (e.g., for common themes or reset styles). You can opt out of style scoping using the `scoped` attribute:

```orbit
<style scoped="false">
  /* These styles will apply globally */
  body {
    font-family: 'Roboto', sans-serif;
    margin: 0;
    padding: 0;
  }
  
  .global-button {
    background-color: #0070f3;
    color: white;
    border-radius: 4px;
  }
</style>
```

## Advanced Style Scoping Techniques

### Deep Selectors

Sometimes you need to target child components from a parent component. You can use the `/deep/` or `>>>` selector for this:

```css
.parent /deep/ .child {
  /* This will affect .child elements even in child components */
  color: red;
}

/* Alternatively */
.parent >>> .child {
  margin-top: 10px;
}
```

### Hybrid Approach

You can have multiple `<style>` sections in a single component, some scoped and some not:

```orbit
<style>
  /* Scoped styles (default) */
  .container {
    padding: 20px;
  }
</style>

<style scoped="false">
  /* Global styles */
  .global-header {
    background-color: #f0f0f0;
  }
</style>
```

### Styling Slotted Content

When using slots to receive content from parent components, you might need to style that content:

```css
/* Target elements slotted into the default slot */
::slotted(*) {
  margin: 5px;
}

/* Target specific elements in a named slot */
::slotted(button) {
  padding: 10px;
}

/* Target elements in a specific named slot */
::slotted(.header-content) {
  font-weight: bold;
}
```

## CSS Variables and Theming

Style scoping works well with CSS variables for creating themeable components:

```orbit
<style>
  .themed-component {
    color: var(--text-color, #333);
    background-color: var(--bg-color, white);
    border: 1px solid var(--border-color, #ccc);
  }
</style>

<code lang="rust">
  // You can set CSS variables from your component code
  use orbit::prelude::*;
  
  impl Component for ThemedComponent {
    // ...other methods...
    
    fn mounted(&mut self) {
      // Set CSS variables based on props or state
      self.set_css_variable("--text-color", &self.props.theme.text_color);
      self.set_css_variable("--bg-color", &self.props.theme.background);
    }
  }
</code>
```

## Performance Considerations

Style scoping adds a small overhead to the CSS processing and rendering pipeline. For performance-critical applications, consider these approaches:

1. **Minimize Scoped Styles** - Move common styles to global stylesheets
2. **Use CSS Variables** - Define variables globally, use them in scoped components
3. **Optimize Selectors** - Simple selectors are faster than complex ones
4. **Style Sheet Splitting** - Extract frequently changing styles into separate style blocks

## Best Practices

1. **Default to Scoped Styles** - Use scoped styles by default for component modularity
2. **Global Styles for Common Elements** - Use unscoped styles for global themes, resets, and utilities
3. **Avoid Deep Selectors** - Use them sparingly as they reduce the benefits of scoping
4. **Component-Based Design** - Design components with internal style consistency
5. **Use Variables for Theming** - Pass themes through CSS variables rather than overriding styles

## Troubleshooting

### Styles Not Being Applied

If styles aren't being applied as expected, check:

1. **Selector Specificity** - Scoped selectors have increased specificity
2. **Component Boundaries** - Styles don't cross component boundaries unless using deep selectors
3. **Style Precedence** - Later styles override earlier ones
4. **Runtime Issues** - Check the developer console for CSS errors

### Style Leakage

If styles are affecting other components unexpectedly:

1. **Check `scoped` Attribute** - Ensure you haven't disabled scoping unintentionally
2. **Avoid Element-Only Selectors** - Using `div {}` is more likely to leak than `.my-div {}`
3. **Check Global Styles** - They might be overriding your component styles

## Browser Support

Orbit's style scoping works in all modern browsers. The implementation uses standard CSS attributes and classes rather than Shadow DOM for maximum compatibility.
