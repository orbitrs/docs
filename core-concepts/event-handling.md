# 🎯 Event Handling in Orbit

## 📝 Overview

Event handling is a core aspect of interactive UI applications. The Orbit Framework provides a powerful and flexible event system that supports various interaction types across different platforms while maintaining a consistent developer experience.

## 🔄 Event Flow Architecture

Orbit's event system follows a bubbling and capturing model similar to the DOM, but with optimizations specific to Rust and cross-platform environments.

### Event Phases

1. **Capture Phase** - Events travel from the root component down to the target component
2. **Target Phase** - Event reaches the component where the event originated
3. **Bubble Phase** - Events bubble up from the target component to the root

```
     ┌─────────────────┐
     │     Root        │
     │   Component     │ ◄─── 3. Bubble Phase (up)
     └────────┬────────┘
              │
     ┌────────▼────────┐
     │     Parent      │
     │    Component    │
     └────────┬────────┘
              │
┌─────────────▼─────────────┐
│        Target             │
│       Component           │ ◄─── 2. Target Phase
└─────────────┬─────────────┘
              │
              │
              │
    1. Capture Phase (down)
```

## 📋 Event Types

Orbit supports several categories of events:

### User Input Events

- **Pointer Events**: `click`, `dblclick`, `pointerdown`, `pointerup`, `pointermove`, `pointerenter`, `pointerleave`
- **Keyboard Events**: `keydown`, `keyup`, `keypress`
- **Touch Events**: `touchstart`, `touchmove`, `touchend`, `touchcancel`
- **Form Events**: `change`, `input`, `submit`, `focus`, `blur`

### Component Lifecycle Events

- **Initialization Events**: `mounted`, `initialized`
- **Update Events**: `beforeupdate`, `updated`
- **Destruction Events**: `beforedestroy`, `destroyed`

### Custom Events

Define your own events to communicate between components:

```rust
use orbit::events::{Event, EventData};

// Define custom event data
struct ItemSelectedEvent {
    id: String,
    value: i32,
}

// Implement EventData for your custom data
impl EventData for ItemSelectedEvent {}

// Create and dispatch the event
let event = Event::new("itemselected", ItemSelectedEvent { 
    id: "item-1".to_string(), 
    value: 42 
});
component.dispatch_event(event);
```

## 🧩 Event Handling in Components

### In `.orbit` Files

```orbit
<template>
  <button @click="increment">
    Count: {{ count }}
  </button>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct Counter {
    count: State<i32>,
}

impl Counter {
    pub fn new() -> Self {
        Self {
            count: State::new(0),
        }
    }
    
    // Event handler method
    pub fn increment(&mut self, _event: &ClickEvent) {
        self.count.set(*self.count.get() + 1);
    }
}
</script>
```

### Using Event Modifiers

Orbit provides event modifiers to customize event behavior:

```orbit
<template>
  <!-- Stop propagation -->
  <button @click.stop="handleClick">Click Me</button>
  
  <!-- Prevent default behavior -->
  <form @submit.prevent="handleSubmit">...</form>
  
  <!-- Only trigger once -->
  <div @mouseover.once="showTooltip">...</div>
  
  <!-- Key modifiers -->
  <input @keydown.enter="submit">
  
  <!-- Combined modifiers -->
  <div @click.stop.prevent="doSomething">...</div>
</template>
```

## 🔄 Event Delegation

Orbit uses event delegation for efficient event handling, particularly for lists and repeated items. Instead of attaching listeners to each item, attach a single listener to a parent element:

```orbit
<template>
  <ul @click="handleItemClick">
    <li v-for="item in items" data-id="{{ item.id }}">
      {{ item.name }}
    </li>
  </ul>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct ItemList {
    items: State<Vec<Item>>,
}

impl ItemList {
    pub fn handle_item_click(&self, event: &ClickEvent) {
        // Get the item ID from the clicked element
        if let Some(target) = event.target() {
            if let Some(id) = target.get_attribute("data-id") {
                println!("Clicked item with ID: {}", id);
            }
        }
    }
}
</script>
```

## 🌐 Cross-Platform Event Handling

Orbit abstracts platform-specific event systems into a unified API. This means you can use the same event handling code regardless of whether your application runs on:

- Web (via WebAssembly)
- Desktop (Windows, macOS, Linux)
- Mobile (iOS, Android)
- Embedded systems

Behind the scenes, Orbit maps these events to the appropriate platform-specific events:

| Orbit Event   | Web                | Desktop          | Mobile           |
|---------------|--------------------|-----------------:|------------------|
| `click`       | `click`            | `mouseup`        | `touchend`       |
| `pointerdown` | `pointerdown`      | `mousedown`      | `touchstart`     |
| `change`      | `change`           | Custom impl      | Custom impl      |

## 🧪 Event Testing

Orbit provides utilities for testing event handlers:

```rust
use orbit::testing::*;
use orbit::events::*;

#[test]
fn test_click_handler() {
    // Create a test component
    let mut component = TestComponent::<Counter>::new();
    
    // Assert initial state
    assert_eq!(*component.get_state::<i32>("count"), 0);
    
    // Simulate click event
    component.dispatch_event(Event::new("click", ClickEvent::default()));
    
    // Assert state after event
    assert_eq!(*component.get_state::<i32>("count"), 1);
}
```

## 🔒 Best Practices

1. **Delegate events** whenever possible for better performance
2. **Use appropriate event types** - don't use click events when you need hover events
3. **Be cautious with event.stopPropagation()** - it might prevent other necessary event handlers from executing
4. **Keep handlers small and focused** - extract complex logic into separate methods
5. **Consider accessibility** - ensure keyboard events accompany mouse/touch events for accessibility

## 🧠 Advanced Event Patterns

### Event Bus Pattern

For complex applications, an event bus can facilitate communication between distant components without prop drilling:

```rust
use orbit::prelude::*;
use orbit::events::*;

// Global event bus
pub struct EventBus {
    listeners: RefCell<HashMap<String, Vec<Box<dyn Fn(&Event)>>>>,
}

impl EventBus {
    pub fn new() -> Self {
        Self {
            listeners: RefCell::new(HashMap::new()),
        }
    }
    
    pub fn on<F>(&self, event_name: &str, callback: F) -> EventSubscription
    where
        F: Fn(&Event) + 'static,
    {
        let event_name = event_name.to_string();
        let listener_id = generate_unique_id();
        
        self.listeners.borrow_mut()
            .entry(event_name.clone())
            .or_insert_with(Vec::new)
            .push(Box::new(callback));
            
        EventSubscription {
            event_bus: self,
            event_name,
            listener_id,
        }
    }
    
    pub fn emit(&self, event: Event) {
        if let Some(listeners) = self.listeners.borrow().get(&event.name) {
            for listener in listeners {
                listener(&event);
            }
        }
    }
    
    pub fn off(&self, event_name: &str, listener_id: &str) {
        // Implementation to remove a specific listener
    }
}

// Subscription object for cleanup
pub struct EventSubscription<'a> {
    event_bus: &'a EventBus,
    event_name: String,
    listener_id: String,
}

impl<'a> Drop for EventSubscription<'a> {
    fn drop(&mut self) {
        self.event_bus.off(&self.event_name, &self.listener_id);
    }
}

// Usage in components
#[component]
pub struct AppRoot {
    event_bus: Rc<EventBus>,
}

impl AppRoot {
    pub fn new() -> Self {
        Self {
            event_bus: Rc::new(EventBus::new()),
        }
    }
    
    pub fn provide_event_bus(&self) -> Rc<EventBus> {
        self.event_bus.clone()
    }
}

#[component]
pub struct HeaderComponent {
    event_bus: Rc<EventBus>,
    _subscription: Option<EventSubscription<'static>>,  // Hold subscription for lifetime management
}

impl HeaderComponent {
    pub fn new(event_bus: Rc<EventBus>) -> Self {
        Self {
            event_bus,
            _subscription: None,
        }
    }
    
    #[lifecycle(mounted)]
    pub fn subscribe(&mut self) {
        let event_bus = self.event_bus.clone();
        self._subscription = Some(self.event_bus.on("user:login", move |event| {
            // Handle login event
            if let Some(data) = event.data::<UserLoginEvent>() {
                println!("User logged in: {}", data.username);
            }
        }));
    }
    
    pub fn trigger_logout(&self) {
        self.event_bus.emit(Event::new("user:logout", UserLogoutEvent {}));
    }
}
```

### Debounced and Throttled Events

For performance-sensitive events like resize or scroll, implement debouncing or throttling:

```rust
use orbit::prelude::*;
use orbit::utils::timing::*;

#[component]
pub struct SearchInput {
    query: State<String>,
    debounced_search: Debouncer<String>,
}

impl SearchInput {
    pub fn new() -> Self {
        Self {
            query: State::new(String::new()),
            // Create a debouncer that waits 300ms after the last update
            debounced_search: Debouncer::new(Duration::from_millis(300)),
        }
    }
    
    pub fn update_query(&self, event: &InputEvent) {
        let new_query = event.target_value();
        self.query.set(new_query.clone());
        
        // Trigger debounced search
        self.debounced_search.update(new_query, |query| {
            // This will only run once the user stops typing for 300ms
            perform_search(&query);
        });
    }
}

// For scroll or resize events, use throttling
#[component]
pub struct InfiniteScroller {
    throttled_scroll: Throttler,
}

impl InfiniteScroller {
    pub fn new() -> Self {
        Self {
            // Create a throttler that runs at most once every 100ms
            throttled_scroll: Throttler::new(Duration::from_millis(100)),
        }
    }
    
    pub fn handle_scroll(&self, event: &ScrollEvent) {
        // This will run at most once every 100ms
        self.throttled_scroll.run(|| {
            let scroll_position = event.scroll_top();
            let container_height = event.target_height();
            let content_height = event.scrollable_height();
            
            // Check if we need to load more content
            if content_height - (scroll_position + container_height) < 200.0 {
                self.load_more_content();
            }
        });
    }
    
    fn load_more_content(&self) {
        // Implementation to load more content
    }
}
```

### Custom Event Creation and Propagation

Create and dispatch custom events for component communication:

```rust
use orbit::prelude::*;
use orbit::events::*;

// Define custom event data
pub struct ItemAddedEvent {
    pub id: String,
    pub name: String,
    pub timestamp: u64,
}

// Implement the EventData trait
impl EventData for ItemAddedEvent {}

#[component]
pub struct ItemCreator {
    name: State<String>,
    item_added: EventEmitter<ItemAddedEvent>,
}

impl ItemCreator {
    pub fn new() -> Self {
        Self {
            name: State::new(String::new()),
            item_added: EventEmitter::new(),
        }
    }
    
    pub fn update_name(&self, event: &InputEvent) {
        self.name.set(event.target_value());
    }
    
    pub fn add_item(&self, _: &ClickEvent) {
        let name = self.name.get().clone();
        if !name.is_empty() {
            // Create the custom event data
            let event_data = ItemAddedEvent {
                id: generate_unique_id(),
                name,
                timestamp: get_current_timestamp(),
            };
            
            // Emit the custom event
            self.item_added.emit(event_data);
            
            // Clear the input
            self.name.set(String::new());
        }
    }
}

// Parent component listening for the custom event
#[component]
pub struct ItemManager {
    items: State<Vec<Item>>,
}

impl ItemManager {
    pub fn new() -> Self {
        Self {
            items: State::new(Vec::new()),
        }
    }
    
    pub fn handle_item_added(&self, event: &ItemAddedEvent) {
        // Add the new item to the list
        self.items.update(|items| {
            items.push(Item {
                id: event.id.clone(),
                name: event.name.clone(),
                created_at: event.timestamp,
            });
        });
        
        // Additional logic like saving to storage, etc.
    }
}

impl Component for ItemManager {
    fn render(&self) -> Node {
        html! {
            <div class="item-manager">
                <h2>Item Manager</h2>
                
                <ItemCreator @item_added="handle_item_added" />
                
                <ul class="items-list">
                    {
                        self.items.get().iter().map(|item| {
                            html! {
                                <li key={item.id.clone()}>
                                    {item.name.clone()}
                                </li>
                            }
                        }).collect::<Vec<_>>()
                    }
                </ul>
            </div>
        }
    }
}
```

### Asynchronous Event Processing

Handle events asynchronously when they trigger operations that may take time:

```rust
use orbit::prelude::*;
use orbit::async_utils::*;

#[component]
pub struct AsyncDataLoader {
    is_loading: State<bool>,
    data: State<Option<Data>>,
    error: State<Option<String>>,
}

impl AsyncDataLoader {
    pub fn new() -> Self {
        Self {
            is_loading: State::new(false),
            data: State::new(None),
            error: State::new(None),
        }
    }
    
    pub async fn load_data(&self, event: &ClickEvent) {
        // Prevent multiple concurrent requests
        if *self.is_loading.get() {
            return;
        }
        
        // Reset error state and set loading
        self.error.set(None);
        self.is_loading.set(true);
        
        // Start async operation
        match fetch_data().await {
            Ok(data) => {
                self.data.set(Some(data));
            },
            Err(err) => {
                self.error.set(Some(err.to_string()));
            }
        }
        
        // End loading state
        self.is_loading.set(false);
    }
}

impl Component for AsyncDataLoader {
    fn render(&self) -> Node {
        html! {
            <div class="data-loader">
                <button 
                    @click="load_data"
                    :disabled="is_loading"
                >
                    {if *self.is_loading.get() { "Loading..." } else { "Load Data" }}
                </button>
                
                {
                    if let Some(error) = &*self.error.get() {
                        html! { <div class="error">{error}</div> }
                    } else if let Some(data) = &*self.data.get() {
                        html! { <DataDisplay :data={data.clone()} /> }
                    } else {
                        html! { <div class="empty-state">No data loaded yet</div> }
                    }
                }
            </div>
        }
    }
}
```

## 🚧 Edge Cases and Troubleshooting

### Event Propagation Quirks

Events bubble up the component tree, but there are scenarios where event propagation can be unexpected:

```rust
// Problem: Inner click handler stops propagation, 
// preventing outer handler from running
html! {
    <div @click="outerClickHandler">
        <button @click.stop="innerClickHandler">
            Click Me
        </button>
    </div>
}

// Solution: Use capture phase to ensure outer handler runs
html! {
    <div @click.capture="outerClickHandler">
        <button @click.stop="innerClickHandler">
            Click Me
        </button>
    </div>
}
```

### Mobile-Specific Event Handling

Mobile devices have unique event behaviors that require special handling:

```rust
#[component]
pub struct TouchFriendlyComponent {
    touch_timer: State<Option<u32>>,
    active: State<bool>,
}

impl TouchFriendlyComponent {
    pub fn new() -> Self {
        Self {
            touch_timer: State::new(None),
            active: State::new(false),
        }
    }
    
    // Prevent "ghost clicks" that can occur after touchend events
    pub fn handle_touch_start(&self, event: &TouchEvent) {
        self.active.set(true);
        
        // Store touch coordinates for later comparison
        if let Some(touch) = event.touches().item(0) {
            // Store coordinates in component state
        }
    }
    
    pub fn handle_touch_end(&self, _event: &TouchEvent) {
        self.active.set(false);
        
        // Set a timer to prevent immediate click events
        let timer_id = set_timeout(|| {
            // Clear the touch timer after 300ms
            self.touch_timer.set(None);
        }, 300);
        
        self.touch_timer.set(Some(timer_id));
    }
    
    pub fn handle_click(&self, event: &ClickEvent) {
        // If there was a recent touch event, ignore this click
        // to prevent duplicate handling
        if self.touch_timer.get().is_some() {
            event.prevent_default();
            return;
        }
        
        // Normal click handling
    }
}

impl Component for TouchFriendlyComponent {
    fn render(&self) -> Node {
        html! {
            <div 
                class="touch-friendly {{ active ? 'active' : '' }}"
                @touchstart="handle_touch_start"
                @touchend="handle_touch_end"
                @click="handle_click"
            >
                Touch or Click Me
            </div>
        }
    }
}
```

### Cleaning Up Event Listeners

Properly clean up event listeners, especially those attached to the window or document:

```rust
#[component]
pub struct GlobalEventListener {
    click_listener: Option<EventListener>,
    resize_listener: Option<EventListener>,
}

impl GlobalEventListener {
    pub fn new() -> Self {
        Self {
            click_listener: None,
            resize_listener: None,
        }
    }
    
    #[lifecycle(mounted)]
    pub fn setup_listeners(&mut self) {
        // Create document click listener
        let click_handler = EventHandler::new(|event: &ClickEvent| {
            // Handle document clicks
            println!("Document clicked at: ({}, {})", event.client_x(), event.client_y());
        });
        
        self.click_listener = Some(document_add_event_listener("click", click_handler));
        
        // Create window resize listener
        let resize_handler = EventHandler::new(|_: &ResizeEvent| {
            // Handle window resize
            println!("Window resized to: {}x{}", window_width(), window_height());
        });
        
        self.resize_listener = Some(window_add_event_listener("resize", resize_handler));
    }
    
    #[lifecycle(before_destroy)]
    pub fn cleanup_listeners(&mut self) {
        // Clean up document listener
        if let Some(listener) = self.click_listener.take() {
            listener.remove();
        }
        
        // Clean up window listener
        if let Some(listener) = self.resize_listener.take() {
            listener.remove();
        }
    }
}
```

### Event Performance Optimization

For high-frequency events like scroll, mousemove, or resize, use specialized techniques:

```rust
#[component]
pub struct PerformanceOptimizedComponent {
    last_run_time: State<f64>,
    frame_request_id: State<Option<u32>>,
    mouse_position: State<(f64, f64)>,
}

impl PerformanceOptimizedComponent {
    pub fn new() -> Self {
        Self {
            last_run_time: State::new(0.0),
            frame_request_id: State::new(None),
            mouse_position: State::new((0.0, 0.0)),
        }
    }
    
    // Use passive event listeners for touch and wheel events
    #[lifecycle(mounted)]
    pub fn setup_passive_listeners(&self) {
        let options = EventListenerOptions {
            passive: true,
            capture: false,
        };
        
        add_event_listener_with_options("wheel", handle_wheel, options);
        add_event_listener_with_options("touchmove", handle_touch_move, options);
    }
    
    // Use requestAnimationFrame for smooth animations with mouse movement
    pub fn handle_mouse_move(&self, event: &MouseEvent) {
        // Store the position but don't render yet
        let x = event.client_x();
        let y = event.client_y();
        self.mouse_position.set((x, y));
        
        // If we don't have an animation frame requested, request one
        if self.frame_request_id.get().is_none() {
            let frame_id = request_animation_frame(|| {
                self.update_ui_on_animation_frame();
            });
            self.frame_request_id.set(Some(frame_id));
        }
    }
    
    pub fn update_ui_on_animation_frame(&self) {
        // Clear the frame request ID
        self.frame_request_id.set(None);
        
        // Update UI based on the mouse position
        let (x, y) = *self.mouse_position.get();
        
        // Update UI elements based on position...
        
        // Track when we last updated
        self.last_run_time.set(now());
    }
}
```

## 🔄 Related Topics

- [State Management](./state-management.md) - Learn how to manage and update component state
- [Component Model](./component-model.md) - Understand how components work in Orbit
- [Rendering Architecture](./rendering-architecture.md) - Learn about the rendering process
- [Advanced Component Patterns](./advanced-component-patterns.md) - Explore advanced component patterns
