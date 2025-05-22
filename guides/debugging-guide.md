# Debugging Guide for Orbit Applications

This guide provides comprehensive strategies and tools for debugging Orbit applications across different environments, from development to production.

## Overview

Debugging is an essential part of the development process. This guide covers:

1. Development-time debugging
2. Browser-based debugging for web applications
3. Native application debugging
4. Production debugging and troubleshooting
5. Common issues and their solutions

## Development Environment Setup

### IDE Configuration

For optimal debugging experience, configure your IDE properly:

#### VS Code

1. Install the "Rust Analyzer" extension
2. Install the "CodeLLDB" extension for breakpoint debugging
3. Create a launch configuration in `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug Orbit Application",
            "cargo": {
                "args": ["build", "--bin=my-orbit-app", "--package=my-orbit-app"],
                "filter": {
                    "name": "my-orbit-app",
                    "kind": "bin"
                }
            },
            "args": [],
            "cwd": "${workspaceFolder}"
        },
        {
            "type": "chrome",
            "request": "launch",
            "name": "Debug Web (Chrome)",
            "url": "http://localhost:3000",
            "webRoot": "${workspaceFolder}/dist"
        }
    ]
}
```

#### IntelliJ/CLion

1. Install the "Rust" plugin
2. Configure a Run/Debug configuration:
   - Select "Cargo" configuration type
   - Command: `run`
   - Working directory: your project root

### Logging Setup

Configure proper logging for your application:

```rust
use log::{debug, error, info, warn};
use env_logger;

fn main() {
    env_logger::init();
    
    info!("Application starting up");
    debug!("Debug information");
    
    // Application code
}
```

Add to your `Cargo.toml`:

```toml
[dependencies]
log = "0.4"
env_logger = "0.10"
```

Run with verbose logging:

```bash
RUST_LOG=debug cargo run
```

## Orbit Development Server Debugging

The Orbiton development server includes several debugging features:

### Hot Module Reloading (HMR)

HMR preserves state between reloads during development:

```bash
orbiton dev --hot-reload
```

### Inspector Mode

Enable the Orbit Inspector to examine component structure:

```bash
orbiton dev --inspector
```

The inspector provides:
- Component tree visualization
- State inspection
- Event monitoring
- Performance metrics

### Dev Server Logs

Access detailed logs from the dev server:

```bash
orbiton dev --log-level debug
```

## Console Debugging

### Debug Macros

Use Orbit's debugging macros for efficient console output:

```rust
use orbit::debug;

fn my_function() {
    let user = get_user();
    debug!(user);  // Prints: [DEBUG] user = User { id: 1, name: "John" }
    
    // Advanced formatting
    debug!(target: "auth", "User authenticated: {}", user.name);
}
```

### Conditional Debugging

```rust
#[cfg(debug_assertions)]
fn debug_only_function() {
    // This code only runs in debug builds
}

fn main() {
    #[cfg(debug_assertions)]
    println!("Running in debug mode");
    
    // Regular code
}
```

## Browser Debugging (Web Applications)

For Orbit web applications:

### Web Console

Use `console.log` calls via Orbit's web interop:

```rust
use orbit::web::console;

fn handle_click() {
    console::log("Button clicked");
    console::table(vec!["data1", "data2", "data3"]); // Tabular data
    console::error("Something went wrong!");
}
```

### Chrome/Firefox DevTools

1. Open DevTools (F12)
2. Key sections for Orbit apps:
   - Console: View logs and errors
   - Network: Monitor API calls
   - Elements: Inspect rendered DOM
   - Sources: Set breakpoints in compiled WebAssembly

### WebAssembly Debugging

For debugging the WebAssembly code:

1. Enable source maps in your build:
   ```bash
   orbiton build --debug --sourcemaps
   ```

2. In Chrome DevTools, navigate to Sources > Filesystem
3. Add your project folder
4. Now you can set breakpoints directly in your Rust source code

### Using the Orbit Web Inspector

The Orbit Web Inspector provides real-time insights:

```bash
orbiton dev --web-inspector
```

Access at `http://localhost:3000/_orbit/inspector`

## Native Application Debugging

For desktop or mobile applications:

### GDB/LLDB

Debug native Orbit applications with LLDB:

```bash
# Build with debug symbols
cargo build

# Debug with LLDB
lldb target/debug/my-orbit-app
```

Common LLDB commands:
- `b src/main.rs:45` - Set breakpoint at line 45
- `run` - Start program
- `n` - Next line
- `s` - Step into function
- `p variable_name` - Print variable
- `bt` - Backtrace

### Platform-Specific Tools

#### Windows

Use Visual Studio debugger:
1. Open project in Visual Studio
2. Set breakpoints
3. Run with F5

#### macOS

Use Xcode instruments:
```bash
xcrun instruments -t Time\ Profiler -D output.trace ./target/debug/my-orbit-app
```

#### Linux

Use `perf` for performance analysis:
```bash
perf record --call-graph dwarf ./target/debug/my-orbit-app
perf report
```

## Component Debugging

### Component State Inspection

Monitor component state changes:

```rust
impl Component for MyComponent {
    // ...
    
    fn updated(&mut self) {
        debug!("Component updated, new state: {:?}", self);
    }
    
    // ...
}
```

### Event Debugging

Track event propagation:

```rust
fn handle_event(&mut self, event: Event) {
    debug!("Event received: {:?}", event);
    
    // Handle the event
}
```

### Orbit Debug Mode

Enable Orbit's component debug mode:

```rust
orbit::enable_debug_mode();
```

This provides:
- Component lifecycle logging
- State mutation tracking
- Event propagation visualization
- Performance metrics

## State Management Debugging

### Inspecting State Store

For centralized state stores:

```rust
use orbit::store::{Store, Observer};

struct DebugObserver;

impl Observer for DebugObserver {
    fn notify(&self, state: &AppState, action: &Action) {
        debug!("Action dispatched: {:?}", action);
        debug!("New state: {:?}", state);
    }
}

fn main() {
    let store = Store::new(initial_state);
    store.add_observer(Box::new(DebugObserver));
    
    // App initialization
}
```

### Time-Travel Debugging

Enable time-travel debugging for development:

```rust
use orbit::debug::TimeTravel;

fn main() {
    let mut time_travel = TimeTravel::new();
    time_travel.enable();
    
    // App code
}
```

Access the time-travel UI at `http://localhost:3000/_orbit/timetravel`

## Network Debugging

For debugging API requests and responses:

```rust
use orbit::network::RequestTracer;

async fn fetch_data() {
    let tracer = RequestTracer::new();
    
    let response = orbit::http::get("https://api.example.com/data")
        .trace(&tracer)
        .send()
        .await?;
        
    debug!("Response: {:?}", response);
}
```

### Mock API Responses

During development, use mock responses:

```rust
use orbit::testing::MockApi;

fn main() {
    let mock_api = MockApi::new();
    
    // Mock a GET request
    mock_api.mock(
        "GET", 
        "https://api.example.com/users",
        200,
        r#"[{"id": 1, "name": "Test User"}]"#
    );
    
    // App initialization with mock API
    orbit::init_with_api(mock_api);
}
```

## Performance Debugging

### Component Profiling

Identify slow rendering components:

```bash
orbiton build --profile
orbiton run --profile
```

View profile data at `http://localhost:3000/_orbit/profile`

### Memory Analysis

Track memory usage:

```rust
use orbit::debug::MemTracker;

fn main() {
    let tracker = MemTracker::new();
    tracker.enable();
    
    // App code that might leak memory
    
    let report = tracker.generate_report();
    debug!("{}", report);
}
```

## Common Issues and Solutions

### Reactivity Issues

**Issue**: Component not updating when state changes

**Debugging steps**:
1. Verify the state mutation is detected by adding debug prints
2. Check if you're mutating a copy instead of the original state
3. Ensure the component is properly connected to the state
4. Check that your component implements `should_update()` correctly

**Solution Example**:
```rust
// Instead of this (no update)
let mut temp = self.items;
temp.push(new_item);

// Do this
self.items.push(new_item);
```

### Event Handling Issues

**Issue**: Event handlers not firing

**Debugging steps**:
1. Add console logs before and after event dispatch
2. Check event naming (e.g., `click` vs `onclick`)
3. Verify event bubbling isn't stopped prematurely
4. Ensure handlers are properly bound

**Solution Example**:
```rust
// Instead of this (incorrect binding)
<button @click="self.handle_click">Click</button>

// Do this
<button @click="handle_click">Click</button>
```

### Rendering Issues

**Issue**: Components render incorrectly

**Debugging steps**:
1. Check the component hierarchy with the Orbit Inspector
2. Verify CSS/styles are applied correctly
3. Inspect component props and state
4. Check for conditional rendering logic errors

**Solution Example**:
```rust
// Instead of this (might cause flicker)
if self.is_visible {
    return html!(<div>Content</div>);
} else {
    return html!(<div class="hidden">Content</div>);
}

// Do this
return html!(<div class={if self.is_visible { "" } else { "hidden" }}>Content</div>);
```

### WebAssembly-Specific Issues

**Issue**: Unexpected behavior in WebAssembly runtime

**Debugging steps**:
1. Check for browser-specific compatibility issues
2. Verify memory allocation and management
3. Look for JavaScript interop issues
4. Test on multiple browsers

**Solution Example**:
```rust
// Instead of assuming browser capabilities
#[cfg(target_arch = "wasm32")]
fn check_browser_support() -> bool {
    // Browser feature detection
}
```

## Production Debugging

### Error Monitoring

Integrate with error monitoring services:

```rust
use orbit::monitoring::Sentry;

fn main() {
    Sentry::init("your-sentry-dsn");
    
    // Set user context for better error reporting
    Sentry::set_user(Some(User {
        id: current_user.id,
        email: current_user.email,
    }));
    
    // App code
}
```

### Logging in Production

Configure appropriate log levels for production:

```rust
use orbit::logger;

fn main() {
    // Only errors and warnings in production
    #[cfg(not(debug_assertions))]
    logger::init(LogLevel::Warn);
    
    // Full debugging in development
    #[cfg(debug_assertions)]
    logger::init(LogLevel::Debug);
    
    // App code
}
```

### Remote Debugging

For production issues that can't be reproduced locally:

1. Enable session recording (privacy considerations required):
   ```rust
   use orbit::debug::SessionRecorder;
   
   fn main() {
       #[cfg(feature = "support-tools")]
       {
           let recorder = SessionRecorder::new();
           recorder.start();
       }
       
       // App code
   }
   ```

2. Implement secure debug tunneling:
   ```bash
   orbiton debug --connect production-app-id
   ```

## Debugging Tools Reference

| Tool | Purpose | Command/Usage |
|------|---------|---------------|
| Orbit Inspector | Visual component debugging | `orbiton dev --inspector` |
| State Time-Travel | Track state changes over time | `orbiton dev --time-travel` |
| Performance Profiler | Identify bottlenecks | `orbiton dev --profile` |
| Rust Analyzer | Static code analysis | IDE integration |
| LLDB | Native debugging | `lldb target/debug/my-app` |
| Chrome DevTools | Web debugging | F12 in Chrome browser |
| Memory Tracker | Find memory leaks | `orbit::debug::MemTracker` |
| Network Monitor | Track API requests | `orbit::network::RequestTracer` |

## Conclusion

Effective debugging is a critical skill for developing Orbit applications. By understanding the available tools and techniques, you can quickly identify and fix issues in your applications.

## Related Resources

- [Component Model](../core-concepts/component-model.md)
- [State Management](../core-concepts/state-management.md)
- [Event Handling](../core-concepts/event-handling.md)
- [Testing Strategies](./testing-strategies.md)
- [Performance Optimization](./performance-optimization.md)
