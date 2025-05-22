# API Documentation Template

## Component Name

**Package:** `orbitkit::components::component_name`
**Version:** 0.1.0
**Status:** Stable/Beta/Experimental

Brief description of the component's purpose and main use cases.

## Import

```rust
use orbitkit::components::ComponentName;
```

## Examples

### Basic Usage

```orbit
<template>
  <ComponentName prop1="value1" :prop2="dynamicValue">
    Content
  </ComponentName>
</template>

<script>
use orbitkit::components::ComponentName;

pub struct MyComponent {
    dynamic_value: String,
}

impl Component for MyComponent {
    // implementation details
}
</script>
```

### Advanced Usage

```orbit
<template>
  <ComponentName
    :prop1="computedValue"
    @event="handleEvent">
    <template #named-slot>
      <div>Custom content for the named slot</div>
    </template>
  </ComponentName>
</template>
```

## Props

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `prop1` | `String` | ✅ | - | Description of the property |
| `prop2` | `i32` | ❌ | `0` | Description with possible values and constraints |
| `prop3` | `ComponentType` | ❌ | `ComponentType::Default` | Description with possible enum values |

## Events

| Name | Payload Type | Description |
|------|--------------|-------------|
| `change` | `ChangeEvent` | Triggered when the component's value changes |
| `focus` | `FocusEvent` | Triggered when the component receives focus |

## Slots

| Name | Props | Description |
|------|-------|-------------|
| `default` | - | Default content slot |
| `header` | `{ title: String }` | Custom header with a title prop passed to the slot |

## Methods

| Name | Signature | Description |
|------|-----------|-------------|
| `reset` | `fn reset(&mut self)` | Resets the component to its initial state |
| `focus` | `fn focus(&self)` | Programmatically focuses the component |

## Style API

| CSS Variable | Default | Description |
|--------------|---------|-------------|
| `--component-color` | `#0070f3` | Primary color of the component |
| `--component-background` | `#ffffff` | Background color of the component |

### CSS Classes

| Class | Description |
|-------|-------------|
| `.component-active` | Applied when the component is in active state |
| `.component-disabled` | Applied when the component is disabled |

## Accessibility

This component follows the [WAI-ARIA Button Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/button/). It implements the following ARIA attributes:

- `role="button"` when behaving as a button
- `aria-pressed` to indicate toggle state
- `aria-disabled` when in disabled state

### Keyboard Navigation

- `Space` or `Enter`: Activate the component
- `Tab`: Focus the component
- `Escape`: Dismiss or cancel (where applicable)

## Browser Compatibility

| Platform | Support Level |
|----------|---------------|
| Modern browsers (Chrome, Firefox, Safari, Edge) | Full |
| IE11 | Partial (specify limitations) |
| WebKit for iOS | Full |

## Performance Considerations

- This component uses virtualization for large datasets
- Avoid using more than X instances on a single page
- Consider using the lightweight variant for performance-critical applications

## Related Components

- [`SimilarComponent`](./similar-component.md) - Alternative with fewer features but better performance
- [`ExtendedComponent`](./extended-component.md) - Enhanced version with additional capabilities

## Changelog

### v0.1.0
- Initial release

### v0.0.2
- Fixed focus handling bug
- Added new `size` prop
