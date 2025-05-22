# Troubleshooting Guide

This guide helps you diagnose and resolve common issues when working with the Orbit Framework.

## Table of Contents

- [Development Environment Issues](#development-environment-issues)
- [Build Errors](#build-errors)
- [Runtime Errors](#runtime-errors)
- [Rendering Issues](#rendering-issues)
- [Performance Problems](#performance-problems)
- [Cross-Platform Issues](#cross-platform-issues)
- [Common Error Messages](#common-error-messages)
- [Getting Help](#getting-help)

## Development Environment Issues

### Orbiton CLI not found after installation

**Symptoms:**
- `Command not found: orbiton` when trying to use the CLI

**Possible causes:**
- Cargo bin directory not in PATH
- Installation failed

**Solutions:**
1. Ensure Cargo's bin directory is in your PATH:
   ```bash
   echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

2. Reinstall Orbiton:
   ```bash
   cargo install orbiton --force
   ```

3. Check installation location and make sure it's in your PATH:
   ```bash
   which orbiton
   # If not found, try
   find ~/.cargo -name orbiton
   ```

### Project creation fails

**Symptoms:**
- Error when running `orbiton new my-project`

**Possible causes:**
- Missing dependencies
- Permission issues
- Network problems when fetching templates

**Solutions:**
1. Check for missing dependencies:
   ```bash
   rustc --version
   cargo --version
   # Make sure both are installed and up-to-date
   ```

2. Check folder permissions:
   ```bash
   # Try creating in a different location
   orbiton new my-project --path ~/Documents
   ```

3. Try with verbose logging:
   ```bash
   ORBIT_LOG_LEVEL=debug orbiton new my-project
   ```

## Build Errors

### Component compilation errors

**Symptoms:**
- Errors when building `.orbit` components

**Possible causes:**
- Syntax errors in templates
- Missing imports
- Type mismatches

**Solutions:**
1. Use the analyzer to find issues:
   ```bash
   orbiton analyze --fix
   ```

2. Check component syntax:
   - Ensure template has correct HTML-like syntax
   - Verify that all Rust expressions are valid
   - Check for missing closing tags

3. Validate imports and dependencies:
   - Make sure all components are properly imported
   - Check type compatibility between components

### Dependency resolution failures

**Symptoms:**
- `cargo build` fails with dependency errors

**Possible causes:**
- Version conflicts
- Missing features
- Incompatible dependencies

**Solutions:**
1. Update dependencies:
   ```bash
   cargo update
   ```

2. Check Cargo.toml for version conflicts

3. Ensure required features are enabled:
   ```toml
   [dependencies]
   orbitrs = { version = "0.1.0", features = ["skia", "wgpu"] }
   ```

## Runtime Errors

### Component initialization failures

**Symptoms:**
- App crashes during startup
- Components fail to mount

**Possible causes:**
- Initialization errors in component lifecycle
- Props validation failures
- Missing required dependencies

**Solutions:**
1. Check component initialization logic:
   - Ensure `new()` method doesn't panic
   - Add error handling around initialization logic

2. Validate props:
   - Make sure required props are provided
   - Check for type mismatches

3. Add debug logging:
   ```rust
   fn mounted(&mut self) {
       println!("Component mounted with props: {:?}", self.props);
   }
   ```

### State update issues

**Symptoms:**
- UI not updating when state changes
- Unexpected rendering behavior

**Possible causes:**
- Missing state updates
- Side effects not properly handled
- Circular update dependencies

**Solutions:**
1. Verify state updates are triggering renders:
   ```rust
   fn update_count(&mut self) {
       println!("Updating count from {} to {}", self.count, self.count + 1);
       self.count += 1;
       println!("Count updated, should trigger render");
   }
   ```

2. Check for proper event handling:
   - Ensure events are properly connected
   - Verify callbacks are being executed

3. Look for circular dependencies in your update logic

## Rendering Issues

### Visual glitches

**Symptoms:**
- Elements misaligned
- Flickering
- Missing content

**Possible causes:**
- Renderer compatibility issues
- CSS styling problems
- Layout calculation errors

**Solutions:**
1. Try switching renderers:
   ```bash
   orbiton dev --renderer skia
   # Then try
   orbiton dev --renderer wgpu
   ```

2. Check for CSS issues:
   - Inspect the generated CSS
   - Look for conflicting styles
   - Ensure parent containers have proper dimensions

3. Add debug visualization:
   ```css
   /* Add to your component style */
   .debug-layout {
     border: 1px solid red;
   }
   ```

### Text rendering problems

**Symptoms:**
- Incorrect font rendering
- Missing text
- Text alignment issues

**Possible causes:**
- Font loading failures
- Text measurement issues
- Internationalization problems

**Solutions:**
1. Verify font availability:
   - Ensure fonts are properly loaded
   - Try using system fonts temporarily

2. Check text measurement:
   - Use explicit dimensions where needed
   - Add padding to accommodate text variation

3. For international text:
   - Ensure proper language support
   - Check for bidirectional text handling

## Performance Problems

### Slow rendering

**Symptoms:**
- UI feels sluggish
- Low FPS
- Delayed responses

**Possible causes:**
- Too many components rendering
- Inefficient rendering paths
- Resource-intensive operations on the main thread

**Solutions:**
1. Use performance profiling:
   ```bash
   orbiton dev --profile
   ```

2. Optimize render cycles:
   - Implement memo components for expensive renders
   - Use virtualization for long lists
   - Defer non-critical updates

3. Move expensive operations off the main thread:
   ```rust
   fn process_data(&mut self) {
       // Move to a worker thread
       std::thread::spawn(move || {
           // Process data
           // Then dispatch results back to UI
       });
   }
   ```

### Memory leaks

**Symptoms:**
- Increasing memory usage over time
- Performance degradation with prolonged use

**Possible causes:**
- Uncleaned resources
- Unmanaged subscriptions
- Circular references

**Solutions:**
1. Ensure proper cleanup in unmounted:
   ```rust
   fn unmounted(&mut self) {
       // Clean up resources
       self.subscriptions.clear();
       self.cached_data = None;
   }
   ```

2. Check for retained references:
   - Watch for components holding references to each other
   - Ensure callbacks are properly cleaned up

3. Use the memory profiler:
   ```bash
   orbiton dev --memory-profile
   ```

## Cross-Platform Issues

### Web-specific problems

**Symptoms:**
- Works on desktop but not on web
- WASM-specific errors

**Possible causes:**
- Browser compatibility issues
- WASM limitations
- Web API access problems

**Solutions:**
1. Check browser console for errors

2. Use platform-specific code paths:
   ```rust
   #[cfg(target_arch = "wasm32")]
   fn initialize_storage(&mut self) {
       // Web-specific implementation
   }
   
   #[cfg(not(target_arch = "wasm32"))]
   fn initialize_storage(&mut self) {
       // Desktop implementation
   }
   ```

3. Verify WASM compatibility:
   - Some Rust features aren't available in WASM
   - Check for threading limitations

### Desktop-specific issues

**Symptoms:**
- Works on web but not on desktop
- Native platform errors

**Possible causes:**
- System dependencies missing
- Window management issues
- Filesystem access problems

**Solutions:**
1. Check system dependencies:
   ```bash
   # For Linux systems, check for missing libraries
   ldd /path/to/your/executable
   ```

2. Verify permissions:
   - Ensure proper access to required resources
   - Check for sandboxing limitations

3. Try platform-specific debugging:
   ```bash
   orbiton dev --desktop-debug
   ```

## Common Error Messages

### "Component not found: XYZ"

**Cause:** The component isn't properly registered or imported.

**Solution:**
1. Check import statements:
   ```rust
   use crate::components::xyz::XYZ;
   ```
2. Ensure the component is properly exported from its module
3. Verify naming consistency (case sensitivity matters)

### "Property XYZ is required but not provided"

**Cause:** A required prop wasn't provided to a component.

**Solution:**
1. Check component usage:
   ```rust
   // Make sure all required props are provided
   <MyComponent required_prop="value" />
   ```
2. Make the prop optional in the component definition:
   ```rust
   pub struct MyComponentProps {
       pub required_prop: Option<String>,
   }
   ```

### "Failed to initialize renderer"

**Cause:** The selected renderer couldn't be initialized.

**Solution:**
1. Check hardware compatibility
2. Try alternative renderer:
   ```bash
   orbiton dev --renderer skia
   ```
3. Update graphics drivers
4. Verify required dependencies are installed

## Getting Help

If you've tried the solutions above and still have issues:

1. **Check Documentation:**
   - Review the [API Reference](./api/README.md)
   - Read the [Architecture Guide](./architecture.md)

2. **Search Existing Issues:**
   - Check [GitHub Issues](https://github.com/orbitrs/orbitrs/issues)
   - Look for similar problems in the community forums

3. **Ask for Help:**
   - Post in [GitHub Discussions](https://github.com/orbitrs/orbitrs/discussions)
   - Join the community Discord
   - Provide detailed information about your environment and the issue

4. **Report a Bug:**
   - If you've found a reproducible issue, report it on GitHub
   - Include environment details, steps to reproduce, and expected behavior

### Providing Good Bug Reports

When reporting issues, include:

- Orbit Framework version
- Rust version
- Operating system
- Browser (for web issues)
- Steps to reproduce
- Expected vs. actual behavior
- Error messages and stack traces
- Minimal reproducible example if possible
