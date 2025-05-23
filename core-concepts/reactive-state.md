# Reactive State Management in Orbit

This document provides an overview of the reactive state management system introduced in the Orbit framework. This system provides a fine-grained reactivity model similar to those found in modern frameworks like Solid.js and Vue.js.

## Core Concepts

The reactive state system is built around three main concepts:

1. **Signals** - Reactive values that track when they're read and notify listeners when they change
2. **Computed Values** - Derived values that automatically update when their dependencies change
3. **Effects** - Side effects that run when reactive values they depend on change

## Using Signals

Signals are the basic building blocks of the reactive system. They wrap a value and provide methods to get and set it, tracking dependencies and notifying subscribers when the value changes.

```rust
use orbit::prelude::*;

// Create a signal with an initial value
let count = create_signal(0);

// Read the current value
println!("Count: {}", count.get());

// Update the value
count.set(5);

// Update the value using a function
count.update(|&n| n + 1); // Now 6
```

## Computed Values

Computed values are derived from other reactive values. They automatically update when their dependencies change.

```rust
use orbit::prelude::*;

let count = create_signal(0);

// Create a computed value that depends on count
let double_count = create_computed(move || count.get() * 2);

println!("Count: {}, Double: {}", count.get(), double_count.get()); // 0, 0

// Update the count
count.set(5);

println!("Count: {}, Double: {}", count.get(), double_count.get()); // 5, 10
```

## Effects

Effects are side effects that automatically run when their dependencies change.

```rust
use orbit::prelude::*;

let count = create_signal(0);

// Create an effect that runs when count changes
let _effect = create_effect(move || {
    println!("Count changed to {}", count.get());
});

// This will trigger the effect
count.set(5);
```

## Integration with Components

The reactive system is fully integrated with the component model. You can use signals and computed values in your components.

```rust
use orbit::prelude::*;

struct Counter {
    context: Context,
    props: CounterProps,
    count: Rc<Signal<i32>>,
    double_count: Rc<Computed<i32>>,
}

impl Component for Counter {
    type Props = CounterProps;

    fn create(props: Self::Props, context: Context) -> Self {
        // Create a reactive signal for the count
        let count = create_signal(props.initial);
        
        // Create a computed value that depends on count
        let count_clone = count.clone();
        let double_count = create_computed(move || count_clone.get() * 2);
        
        Self {
            context,
            props,
            count,
            double_count,
        }
    }
    
    // Implement other methods...
    
    fn render(&self) -> Result<Vec<Node>, ComponentError> {
        // Use the reactive values in render
        println!("Count: {}, Double: {}", self.count.get(), self.double_count.get());
        
        // Return nodes...
        Ok(vec![])
    }
}
```

## Thread-Safe Signals

For use cases where you need to share reactive state across threads, you can use `ThreadSafeSignal`:

```rust
use orbit::prelude::*;
use std::thread;

let count = ThreadSafeSignal::new(0);
let count_clone = count.clone();

let handle = thread::spawn(move || {
    // Read the count in another thread
    let value = count_clone.get().unwrap();
    println!("Count in thread: {}", value);
    
    // Update the count
    count_clone.set(5).unwrap();
});

// Wait for the thread to complete
handle.join().unwrap();

// Read the updated value
println!("Count after thread: {}", count.get().unwrap());
```

## Benefits of the Reactive System

1. **Fine-grained Updates**: Only components that depend on changed state are updated
2. **Automatic Dependency Tracking**: No need to manually specify dependencies
3. **Predictable Data Flow**: Clear flow of data from signals to computed values to effects
4. **Efficient Rendering**: Minimizes unnecessary re-renders

## When to Use

The reactive state system is ideal for:

- Components with complex state that changes frequently
- When you need derived values that automatically update
- When you want to minimize re-renders
- For creating reactive libraries and utilities

For simpler use cases, you can still use the basic `State` and `StateContainer` API.

## Next Steps

- Check out the `examples/component_lifecycle.rs` file for a complete example
- Read the API documentation for `Signal`, `Computed`, and `Effect`
- Explore integrating the reactive system with the renderer

## Upgrading from the Legacy State System

If you're currently using the legacy `State` and `StateContainer` API, you can gradually migrate to the new reactive system. Both systems can be used side by side in the same application.
