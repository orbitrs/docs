# Orbit Framework FAQ

This document answers frequently asked questions about the Orbit UI Framework.

## General Questions

### What is Orbit?

Orbit is a Rust-first, cross-platform UI framework that enables building Web, Native, and Embedded applications from a unified, single-source component format (`.orbit`). It combines declarative markup with Rust logic for high-performance, flexible, and maintainable UI development.

### How does Orbit compare to other UI frameworks?

| Framework | Similarities | Differences |
|-----------|--------------|------------|
| React | Component-based, Virtual DOM | Rust instead of JavaScript, single-file components, native/embedded support |
| Vue | Single-file components, reactivity | Rust instead of JavaScript, stronger typing, native rendering |
| Flutter | Cross-platform, single codebase | Rust instead of Dart, more native integration, web-first option |
| Tauri | Rust backend, web frontend | Unified component model, no JavaScript requirement |
| gtk-rs | Rust bindings, native UI | Web support, unified component model, modern architecture |

### Is Orbit production-ready?

Orbit is currently in active development (v0.1.x) and working toward its first stable release. While early adopters are using it for experimental projects, we recommend waiting for the v0.5.0 release for non-critical applications and v1.0.0 for production use.

### Who is behind Orbit?

Orbit is an open-source project maintained by a team of Rust developers and UI experts. The project welcomes contributions from the community and is not controlled by any single company.

## Technical Questions

### Why use Rust for UI development?

Rust offers several advantages for UI development:
- Memory safety without garbage collection
- Predictable performance
- Strong type system
- Excellent cross-compilation support
- Growing ecosystem for graphics and UI
- Can target both high-level and low-level platforms

### How does the hybrid rendering approach work?

Orbit uses two complementary rendering engines:
1. **Skia** - for standard UI components, text, and 2D graphics
2. **WGPU** - for hardware-accelerated 3D graphics and custom shaders

The framework automatically selects the appropriate renderer based on the content or allows manual selection. A compositor manages the coordination between renderers when both are needed in the same view.

### Can I use Orbit with existing Rust code?

Yes, Orbit is designed to integrate with existing Rust codebases. You can:
- Import and use Rust libraries in your Orbit components
- Embed Orbit UI components in existing Rust applications
- Gradually migrate parts of your application to Orbit

### Does Orbit support native UI controls?

Orbit primarily uses its own rendering engines rather than native UI controls. This approach ensures consistent appearance and behavior across platforms while still allowing platform-specific optimizations. For certain components (like file pickers), Orbit provides bindings to native dialogs for better integration.

## Component Model

### What is the `.orbit` file format?

The `.orbit` file format is a single-file component format that combines:
- HTML-like markup in the `<template>` section
- CSS styling in the `<style>` section
- Rust code in the `<code lang="rust">` section

This unified format allows you to keep related UI code together while maintaining separation of concerns.

### How do I handle state in Orbit components?

Orbit provides several approaches to state management:
1. **Component State** - Local state defined within the component
2. **Props** - Data passed from parent to child components
3. **Context API** - Shared state accessible by component subtrees
4. **Stores** - Application-wide state management

For detailed information, see the [State Management guide](./core-concepts/state-management.md).

### How does component composition work?

Components can be composed by importing and using them in your templates:

```orbit
<template>
  <div>
    <Header title="My App" />
    <Sidebar :items="menuItems" />
    <MainContent :data="contentData" />
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;
use crate::components::{Header, Sidebar, MainContent};

// Component implementation
</code>
```

## Platform Support

### Which platforms does Orbit support?

Orbit currently supports:
- **Web** - Via WebAssembly (WASM)
- **Desktop** - Windows, macOS, and Linux
- **Embedded** - Experimental support for certain embedded platforms

Mobile support (iOS and Android) is planned for future releases.

### How do I handle platform-specific code?

Orbit provides several approaches to platform-specific code:

1. **Feature flags** - Use Rust's conditional compilation:
   ```rust
   #[cfg(target_arch = "wasm32")]
   fn web_specific_function() { /* ... */ }
   
   #[cfg(not(target_arch = "wasm32"))]
   fn native_specific_function() { /* ... */ }
   ```

2. **Platform API** - Use Orbit's platform detection:
   ```rust
   if platform::is_web() {
       // Web-specific code
   } else if platform::is_desktop() {
       // Desktop-specific code
   }
   ```

3. **Adaptive Components** - Create components that adapt to their platform:
   ```rust
   fn render_button(&self) -> String {
       if platform::is_desktop() {
           // Render native-styled button
       } else {
           // Render web-styled button
       }
   }
   ```

### Does Orbit support progressive web apps (PWAs)?

Yes, Orbit applications can be configured as PWAs when targeting the web platform. The `orbiton` CLI provides tools for generating the necessary manifest files, service workers, and offline support.

## Development Experience

### What tools do I need to develop Orbit applications?

To develop with Orbit, you need:
- Rust toolchain (1.70.0 or newer)
- Cargo package manager
- Orbiton CLI (`cargo install orbiton`)
- (Optional) IDE with Rust support (VS Code with rust-analyzer recommended)

### Does Orbit support hot reloading?

Yes, the `orbiton dev` command starts a development server with hot reloading. When you make changes to your `.orbit` files or Rust code, the application will automatically rebuild and refresh.

### How do I debug Orbit applications?

Debugging depends on your target platform:

- **Web**: Use browser DevTools for inspection and debugging
- **Desktop**: Use standard Rust debugging tools (GDB, LLDB)
- **Both**: Orbit provides built-in logging and error reporting

Additional debugging tools are planned for future releases.

## Performance

### How does Orbit perform compared to other frameworks?

Orbit is designed for high performance across platforms:
- **Compilation**: Rust's ahead-of-time compilation provides fast startup times
- **Rendering**: Skia and WGPU provide hardware-accelerated rendering
- **Memory**: Rust's ownership model prevents memory leaks and reduces GC pauses
- **Updates**: Virtual DOM and fine-grained reactivity minimize re-renders

Detailed benchmarks will be published as the framework matures.

### How do I optimize Orbit applications?

Key optimization strategies include:
- Use memoization for stable components
- Implement virtualization for long lists
- Split large components into focused smaller ones
- Use appropriate renderers for different content types
- Optimize assets (images, fonts, etc.)

For detailed performance guidance, see the [Performance Optimization guide](./guides/performance-optimization.md).

## Common Issues

### Component updates aren't triggering re-renders

**Possible causes**:
- State mutation isn't detected (use proper state update methods)
- Component implements custom `should_update` logic preventing updates
- Event handlers aren't properly connected

**Solutions**:
- Ensure you're using state update methods that trigger reactivity
- Check component update logic
- Verify event handler connections

### Error: "Component not found"

**Possible causes**:
- Component not properly imported
- Component not registered in the module system
- Typo in component name

**Solutions**:
- Check import statements
- Ensure components are properly exported from their modules
- Verify component naming (case sensitivity matters)

### Styling not applying as expected

**Possible causes**:
- CSS specificity issues
- Style scoping conflicts
- Missing style dependencies

**Solutions**:
- Check CSS specificity and use more specific selectors if needed
- Use scoped styles to prevent conflicts
- Ensure all style dependencies are properly imported

For more troubleshooting help, see the [Troubleshooting guide](./guides/troubleshooting.md).

## Contributing

### How can I contribute to Orbit?

There are many ways to contribute:
- Submit bug reports and feature requests
- Improve documentation
- Fix issues and implement features
- Share examples and tutorials
- Help with testing across platforms

See the [Contributing guide](../CONTRIBUTING.md) for detailed instructions.

### Where can I discuss Orbit development?

- GitHub Discussions: [github.com/orbitrs/orbitrs/discussions](https://github.com/orbitrs/orbitrs/discussions)
- Discord: Coming soon
- GitHub Issues: [github.com/orbitrs/orbitrs/issues](https://github.com/orbitrs/orbitrs/issues)

## Learning Resources

### Where can I learn more about Orbit?

- Official documentation (you are here)
- [Example applications](./examples/README.md)
- [GitHub repository](https://github.com/orbitrs/orbitrs)
- Community tutorials (coming soon)
- Video courses (planned for future)

### Are there any sample applications?

Yes, check out the [Examples section](./examples/README.md) for sample applications demonstrating various Orbit features and patterns.

### Where can I find component examples?

The [OrbitKit documentation](./api/orbitkit-components.md) includes examples for all standard components. Additionally, the [Examples directory](./examples/README.md) contains more complex component patterns and usage examples.
