# 🧩 Advanced Component Patterns

## 📝 Overview

This document explores advanced component patterns in the Orbit Framework that help you build more complex, reusable, and maintainable UI components. These patterns build on the foundation laid out in the [Component Model](./component-model.md) documentation.

## 🔄 Composition Patterns

### Component Composition

Component composition is a fundamental technique for building complex UIs from simpler building blocks. Orbit supports various composition patterns:

#### Containment

Use when one component needs to wrap or contain other components:

```orbit
<!-- Layout component that uses slot to render child components -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">Default Header</slot>
    </div>
    <div class="card-body">
      <slot>Default Content</slot>
    </div>
    <div class="card-footer">
      <slot name="footer">Default Footer</slot>
    </div>
  </div>
</template>

<!-- Usage -->
<Card>
  <template #header>
    <h2>My Card Title</h2>
  </template>
  
  <p>This is the main content</p>
  
  <template #footer>
    <button>Submit</button>
  </template>
</Card>
```

#### Specialization

Create specialized versions of more generic components:

```orbit
<!-- Base Button component -->
<template>
  <button 
    class="btn {{ variant }}" 
    :disabled="disabled"
    @click="handleClick"
  >
    <slot></slot>
  </button>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct Button {
    variant: Prop<String>,
    disabled: Prop<bool>,
    on_click: EventEmitter<ClickEvent>,
}

impl Button {
    pub fn new() -> Self {
        Self {
            variant: Prop::with_default("primary".to_string()),
            disabled: Prop::with_default(false),
            on_click: EventEmitter::new(),
        }
    }
    
    pub fn handle_click(&self, event: &ClickEvent) {
        self.on_click.emit(event);
    }
}
</code>

<!-- Creating a specialized component -->
<template>
  <Button variant="danger" @click="handleDelete">
    <Icon name="trash" />
    Delete Item
  </Button>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct DeleteButton {
    on_click: EventEmitter<ClickEvent>,
}

impl DeleteButton {
    pub fn new() -> Self {
        Self {
            on_click: EventEmitter::new(),
        }
    }
    
    pub fn handle_delete(&self, event: &ClickEvent) {
        // Confirmation logic
        if confirm("Are you sure?") {
            self.on_click.emit(event);
        }
    }
}
</script>
```

## 🧠 Higher-Order Components (HOCs)

Higher-Order Components are functions that take a component and return a new component with additional functionality. This pattern is useful for cross-cutting concerns like authentication, logging, or data fetching.

```rust
use orbit::prelude::*;

// HOC for adding authentication to a component
pub fn with_authentication<T: Component>(component: T) -> impl Component {
    struct AuthenticatedComponent<C: Component> {
        inner: C,
        is_authenticated: State<bool>,
    }
    
    impl<C: Component> Component for AuthenticatedComponent<C> {
        // Implementation details...
        
        fn render(&self) -> Node {
            if *self.is_authenticated.get() {
                self.inner.render()
            } else {
                html! {
                    <div class="auth-required">
                        <h2>Authentication Required</h2>
                        <LoginForm @login="handle_login" />
                    </div>
                }
            }
        }
    }
    
    AuthenticatedComponent {
        inner: component,
        is_authenticated: State::new(false),
    }
}

// Usage
let protected_dashboard = with_authentication(Dashboard::new());
```

## 🏭 Component Factories

Component factories are functions or objects that create components dynamically based on input parameters:

```rust
use orbit::prelude::*;

// Component factory function
fn create_dialog(title: String, content: String, is_closable: bool) -> impl Component {
    struct Dialog {
        title: State<String>,
        content: State<String>,
        is_closable: State<bool>,
        is_open: State<bool>,
    }
    
    // Implementation details...
    
    Dialog {
        title: State::new(title),
        content: State::new(content),
        is_closable: State::new(is_closable),
        is_open: State::new(true),
    }
}

// Usage
let confirm_dialog = create_dialog(
    "Confirm Action".to_string(),
    "Are you sure you want to proceed?".to_string(),
    true
);
```

## 💉 Dependency Injection

Orbit supports a form of dependency injection via its context system, allowing components to access shared services without prop drilling:

```orbit
<template>
  <div>{{ theme.current }}</div>
</template>

<code lang="rust">
use orbit::prelude::*;
use orbit::context::*;
use app::services::ThemeService;

#[component]
pub struct ThemeDisplay {
    // Inject theme service from context
    #[context]
    theme: UseContext<ThemeService>,
}

impl ThemeDisplay {
    pub fn new() -> Self {
        Self {
            theme: UseContext::new(),
        }
    }
}
</code>

<!-- Providing context to child components -->
<template>
  <ContextProvider :value="theme_service">
    <App />
  </ContextProvider>
</template>

<code lang="rust">
use orbit::prelude::*;
use app::services::ThemeService;

#[component]
pub struct Root {
    theme_service: State<ThemeService>,
}

impl Root {
    pub fn new() -> Self {
        Self {
            theme_service: State::new(ThemeService::new()),
        }
    }
}
</script>
```

## 🔄 Render Props

The render props pattern allows components to customize what they render based on the current state:

```orbit
<template>
  <div class="data-loader">
    <template v-if="loading">
      <slot name="loading">
        <Spinner />
      </slot>
    </template>
    <template v-else-if="error">
      <slot name="error" :error="error">
        <ErrorDisplay :message="error.message" />
      </slot>
    </template>
    <template v-else>
      <slot :data="data">
        <!-- Default rendering if no slot provided -->
        <DataTable :items="data" />
      </slot>
    </template>
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;
use app::api::fetch_data;

#[component]
pub struct DataLoader<T> {
    url: Prop<String>,
    loading: State<bool>,
    data: State<Option<T>>,
    error: State<Option<Error>>,
}

impl<T: DeserializeOwned + 'static> DataLoader<T> {
    pub fn new() -> Self {
        Self {
            url: Prop::new(),
            loading: State::new(true),
            data: State::new(None),
            error: State::new(None),
        }
    }
    
    #[lifecycle(mounted)]
    pub async fn load_data(&self) {
        self.loading.set(true);
        
        match fetch_data::<T>(self.url.get()).await {
            Ok(data) => {
                self.data.set(Some(data));
                self.error.set(None);
            }
            Err(err) => {
                self.error.set(Some(err));
                self.data.set(None);
            }
        }
        
        self.loading.set(false);
    }
}
</script>

<!-- Usage with custom rendering -->
<DataLoader url="https://api.example.com/users">
  <template #loading>
    <CustomLoader message="Fetching users..." />
  </template>
  
  <template #error="{ error }">
    <Alert type="error">Failed to load users: {{ error.message }}</Alert>
  </template>
  
  <template #default="{ data }">
    <UserList :users="data" />
  </template>
</DataLoader>
```

## 🔍 Compound Components

Compound components are a set of components that work together to provide a cohesive user interface:

```orbit
<!-- Tab component (parent) -->
<template>
  <div class="tabs">
    <div class="tab-headers">
      <slot name="headers"></slot>
    </div>
    <div class="tab-content">
      <slot></slot>
    </div>
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct Tabs {
    active_tab: State<String>,
    // Context to share with child tab components
    context: Context<TabContext>,
}

pub struct TabContext {
    active_tab: String,
    set_active_tab: Box<dyn Fn(String)>,
}

impl Tabs {
    pub fn new() -> Self {
        let active_tab = State::new("".to_string());
        
        let context = Context::new(TabContext {
            active_tab: active_tab.get().clone(),
            set_active_tab: Box::new(move |tab| {
                active_tab.set(tab);
            }),
        });
        
        Self {
            active_tab,
            context,
        }
    }
}
</script>

<!-- TabHeader component -->
<template>
  <button 
    class="tab-header {{ active ? 'active' : '' }}" 
    @click="activate"
  >
    <slot></slot>
  </button>
</template>

<code lang="rust">
use orbit::prelude::*;
use super::TabContext;

#[component]
pub struct TabHeader {
    id: Prop<String>,
    #[context]
    tab_context: UseContext<TabContext>,
}

impl TabHeader {
    pub fn new() -> Self {
        Self {
            id: Prop::new(),
            tab_context: UseContext::new(),
        }
    }
    
    pub fn active(&self) -> bool {
        self.tab_context.active_tab == *self.id.get()
    }
    
    pub fn activate(&self, _: &ClickEvent) {
        (self.tab_context.set_active_tab)(self.id.get().clone());
    }
}
</script>

<!-- TabPanel component -->
<template>
  <div class="tab-panel" v-if="active">
    <slot></slot>
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;
use super::TabContext;

#[component]
pub struct TabPanel {
    id: Prop<String>,
    #[context]
    tab_context: UseContext<TabContext>,
}

impl TabPanel {
    pub fn new() -> Self {
        Self {
            id: Prop::new(),
            tab_context: UseContext::new(),
        }
    }
    
    pub fn active(&self) -> bool {
        self.tab_context.active_tab == *self.id.get()
    }
}
</script>

<!-- Usage of compound components -->
<Tabs>
  <template #headers>
    <TabHeader id="tab1">Users</TabHeader>
    <TabHeader id="tab2">Products</TabHeader>
    <TabHeader id="tab3">Settings</TabHeader>
  </template>
  
  <TabPanel id="tab1">
    <UserList :users="users" />
  </TabPanel>
  
  <TabPanel id="tab2">
    <ProductCatalog :products="products" />
  </TabPanel>
  
  <TabPanel id="tab3">
    <SettingsPanel />
  </TabPanel>
</Tabs>
```

## 🎨 Customization via Hooks

Hooks allow for reusing stateful logic across components:

```rust
use orbit::prelude::*;
use orbit::hooks::*;

// Custom hook for handling form state
pub fn use_form<T: Default + Clone>(initial_values: Option<T>) -> FormHook<T> {
    let values = State::new(initial_values.unwrap_or_default());
    let errors = State::new(HashMap::new());
    let is_submitting = State::new(false);
    
    FormHook {
        values,
        errors,
        is_submitting,
        set_field: Box::new(move |field, value| {
            // Implementation details
        }),
        validate: Box::new(move || {
            // Implementation details
            true
        }),
        submit: Box::new(move |callback| {
            // Implementation details
        }),
    }
}

// Using the hook in a component
#[component]
pub struct LoginForm {
    form: Hook<FormHook<LoginData>>,
    on_login: EventEmitter<LoginEvent>,
}

impl LoginForm {
    pub fn new() -> Self {
        Self {
            form: Hook::new(use_form(Some(LoginData::default()))),
            on_login: EventEmitter::new(),
        }
    }
    
    pub fn handle_submit(&self, _: &SubmitEvent) {
        self.form.submit(|data| {
            self.on_login.emit(&LoginEvent { data });
        });
    }
}
```

## ♿ Accessibility-First Component Design

Designing accessible components requires a strategic approach that embeds accessibility directly into your component architecture. Here are patterns to follow:

### Composite Role Patterns

When building complex components from simpler ones, manage ARIA relationships properly:

```rust
use orbit::prelude::*;
use orbit::a11y::*;

#[component]
pub struct AccessibleMenu {
    id: Prop<String>,
    expanded: State<bool>,
    items: Prop<Vec<MenuItem>>,
    selected_index: State<Option<usize>>,
}

impl AccessibleMenu {
    pub fn new() -> Self {
        Self {
            id: Prop::with_default(generate_unique_id()),
            expanded: State::new(false),
            items: Prop::new(),
            selected_index: State::new(None),
        }
    }
    
    pub fn toggle(&self, _: &ClickEvent) {
        self.expanded.update(|expanded| !expanded);
    }
    
    pub fn select_item(&self, index: usize) {
        self.selected_index.set(Some(index));
        self.expanded.set(false);
    }
    
    pub fn get_menu_id(&self) -> String {
        format!("{}-menu", self.id.get())
    }
    
    pub fn get_button_id(&self) -> String {
        format!("{}-button", self.id.get())
    }
}

impl Component for AccessibleMenu {
    fn render(&self) -> Node {
        html! {
            <div class="menu-container">
                <button 
                    id={self.get_button_id()}
                    class="menu-button"
                    aria-haspopup="true"
                    aria-controls={self.get_menu_id()}
                    aria-expanded={self.expanded.get().to_string()}
                    @click="toggle"
                >
                    <slot name="trigger">Menu</slot>
                </button>
                
                <ul 
                    id={self.get_menu_id()}
                    class="menu-items {{ expanded ? 'visible' : 'hidden' }}" 
                    role="menu"
                    aria-labelledby={self.get_button_id()}
                >
                    {
                        self.items.get().iter().enumerate().map(|(i, item)| {
                            let is_selected = self.selected_index.get().map_or(false, |index| index == i);
                            
                            html! {
                                <li 
                                    role="menuitem" 
                                    class="menu-item {{ is_selected ? 'selected' : '' }}"
                                    tabindex={if *self.expanded.get() { "0" } else { "-1" }}
                                    @click={move |_| self.select_item(i)}
                                >
                                    {item.label.clone()}
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

### Focus Management Patterns

Implement consistent focus management across components:

```rust
use orbit::prelude::*;
use orbit::a11y::*;

// Reusable focus trap hook
pub fn use_focus_trap(root_ref: NodeRef) -> FocusTrapHook {
    let trap_active = State::new(false);
    
    FocusTrapHook {
        trap_active,
        activate: Box::new(move || {
            if let Some(root) = root_ref.get() {
                // Implementation details: Store previous focus, set up key listeners
                trap_active.set(true);
                focus_first_focusable(&root);
            }
        }),
        deactivate: Box::new(move || {
            // Implementation details: Restore focus, clean up listeners
            trap_active.set(false);
        }),
    }
}

// Modal component with focus trap
#[component]
pub struct AccessibleModal {
    open: Prop<bool>,
    on_close: EventEmitter<()>,
    root_ref: NodeRef,
    focus_trap: FocusTrapHook,
}

impl AccessibleModal {
    pub fn new() -> Self {
        let root_ref = NodeRef::new();
        
        Self {
            open: Prop::with_default(false),
            on_close: EventEmitter::new(),
            root_ref,
            focus_trap: use_focus_trap(root_ref),
        }
    }
    
    #[lifecycle(updated)]
    pub fn handle_open_change(&self, prev_props: &Self::Props) {
        // Activate/deactivate focus trap when open state changes
        if *self.open.get() != *prev_props.open.get() {
            if *self.open.get() {
                (self.focus_trap.activate)();
            } else {
                (self.focus_trap.deactivate)();
            }
        }
    }
    
    pub fn close(&self) {
        self.on_close.emit(());
    }
}

impl Component for AccessibleModal {
    fn render(&self) -> Node {
        if !*self.open.get() {
            return html! { <></> };
        }
        
        html! {
            <>
                <div class="modal-backdrop" @click="close"></div>
                <div 
                    class="modal" 
                    role="dialog" 
                    aria-modal="true"
                    ref={self.root_ref.clone()}
                >
                    <div class="modal-content">
                        <div class="modal-header">
                            <slot name="header"></slot>
                            <button 
                                class="close-button" 
                                aria-label="Close modal"
                                @click="close"
                            >
                                ×
                            </button>
                        </div>
                        <div class="modal-body">
                            <slot></slot>
                        </div>
                        <div class="modal-footer">
                            <slot name="footer"></slot>
                        </div>
                    </div>
                </div>
            </>
        }
    }
}
```

### Announcement Pattern

Create accessible, non-intrusive status announcements:

```rust
use orbit::prelude::*;
use orbit::a11y::*;

// Global announcement service
pub struct AnnouncementService {
    message: State<String>,
    politeness: State<Politeness>,
}

#[derive(Clone)]
pub enum Politeness {
    Polite,
    Assertive,
}

impl AnnouncementService {
    pub fn new() -> Self {
        Self {
            message: State::new(String::new()),
            politeness: State::new(Politeness::Polite),
        }
    }
    
    pub fn announce(&self, message: String, politeness: Politeness) {
        self.message.set(message);
        self.politeness.set(politeness);
    }
}

// Live region component that renders announcements
#[component]
pub struct LiveAnnouncer {
    #[context]
    announcement_service: UseContext<AnnouncementService>,
}

impl LiveAnnouncer {
    pub fn new() -> Self {
        Self {
            announcement_service: UseContext::new(),
        }
    }
    
    pub fn politeness_attr(&self) -> &'static str {
        match *self.announcement_service.politeness.get() {
            Politeness::Polite => "polite",
            Politeness::Assertive => "assertive",
        }
    }
}

impl Component for LiveAnnouncer {
    fn render(&self) -> Node {
        html! {
            <div 
                class="sr-only" 
                role="status"
                aria-live={self.politeness_attr()}
            >
                {self.announcement_service.message.get().clone()}
            </div>
        }
    }
}

// Usage
#[component]
pub struct App {
    announcement_service: AnnouncementService,
}

impl App {
    pub fn new() -> Self {
        Self {
            announcement_service: AnnouncementService::new(),
        }
    }
    
    pub fn handle_form_submit(&self, _: &FormEvent) {
        // Process form...
        
        // Announce success
        self.announcement_service.announce(
            "Form submitted successfully".to_string(),
            Politeness::Polite,
        );
    }
}

impl Component for App {
    fn render(&self) -> Node {
        html! {
            <ContextProvider :value={&self.announcement_service}>
                <LiveAnnouncer />
                
                <form @submit="handle_form_submit">
                    <!-- Form fields -->
                    <button type="submit">Submit</button>
                </form>
            </ContextProvider>
        }
    }
}
```

## ⚡ Performance Optimizations for Components

### Memoization Pattern

Avoid unnecessary re-renders by memoizing expensive computations:

```rust
use orbit::prelude::*;
use orbit::memo::*;

#[component]
pub struct DataTable {
    data: Prop<Vec<DataRow>>,
    filter: Prop<String>,
    sort_field: Prop<String>,
    sort_direction: Prop<SortDirection>,
    
    // Memoized computed value
    #[memo(data, filter, sort_field, sort_direction)]
    processed_data: Memo<Vec<DataRow>>,
}

impl DataTable {
    pub fn new() -> Self {
        Self {
            data: Prop::new(),
            filter: Prop::with_default(String::new()),
            sort_field: Prop::with_default(String::new()),
            sort_direction: Prop::with_default(SortDirection::Asc),
            processed_data: Memo::new(|props| {
                // This expensive calculation only runs when deps change
                let mut result = props.data.clone();
                
                // Apply filtering
                if !props.filter.is_empty() {
                    result = result.into_iter()
                        .filter(|row| row.matches_filter(&props.filter))
                        .collect();
                }
                
                // Apply sorting
                if !props.sort_field.is_empty() {
                    result.sort_by(|a, b| {
                        let cmp = a.compare_by_field(b, &props.sort_field);
                        match props.sort_direction {
                            SortDirection::Asc => cmp,
                            SortDirection::Desc => cmp.reverse(),
                        }
                    });
                }
                
                result
            }),
        }
    }
}
```

### Virtual Scrolling Pattern

Render only visible items for large lists:

```rust
use orbit::prelude::*;
use orbit::scroll::*;

#[component]
pub struct VirtualList<T: Clone + 'static> {
    items: Prop<Vec<T>>,
    item_height: Prop<f32>,
    container_height: Prop<f32>,
    scroll_position: State<f32>,
    visible_range: Computed<Range<usize>>,
    renderer: Prop<Box<dyn Fn(&T, usize) -> Node>>,
}

impl<T: Clone + 'static> VirtualList<T> {
    pub fn new() -> Self {
        Self {
            items: Prop::new(),
            item_height: Prop::with_default(40.0),
            container_height: Prop::with_default(400.0),
            scroll_position: State::new(0.0),
            visible_range: Computed::new(|self_| {
                let item_height = *self_.item_height.get();
                let container_height = *self_.container_height.get();
                let scroll_position = *self_.scroll_position.get();
                let items_count = self_.items.get().len();
                
                // Calculate visible range with buffer
                let start_index = (scroll_position / item_height).floor() as usize;
                let visible_count = (container_height / item_height).ceil() as usize + 1; // +1 for partially visible
                let end_index = (start_index + visible_count).min(items_count);
                
                start_index..end_index
            }),
            renderer: Prop::new(),
        }
    }
    
    pub fn handle_scroll(&self, event: &ScrollEvent) {
        self.scroll_position.set(event.scroll_top());
    }
    
    pub fn total_height(&self) -> f32 {
        *self.item_height.get() * self.items.get().len() as f32
    }
    
    pub fn offset_for_index(&self, index: usize) -> f32 {
        *self.item_height.get() * index as f32
    }
}

impl<T: Clone + 'static> Component for VirtualList<T> {
    fn render(&self) -> Node {
        let total_height = self.total_height();
        let visible_range = self.visible_range.get().clone();
        let renderer = self.renderer.get();
        
        html! {
            <div 
                class="virtual-list-container" 
                style="height: {{ container_height }}px; overflow-y: auto;"
                @scroll="handle_scroll"
            >
                <div class="virtual-list-content" style="height: {{ total_height }}px; position: relative;">
                    {
                        self.items.get()[visible_range].iter().enumerate().map(|(i, item)| {
                            let absolute_index = visible_range.start + i;
                            let offset = self.offset_for_index(absolute_index);
                            
                            html! {
                                <div 
                                    class="virtual-list-item" 
                                    style="position: absolute; top: {{ offset }}px; width: 100%;"
                                >
                                    {renderer(item, absolute_index)}
                                </div>
                            }
                        }).collect::<Vec<_>>()
                    }
                </div>
            </div>
        }
    }
}

// Usage
fn render_list() -> Node {
    let items = (0..10000).map(|i| format!("Item {}", i)).collect::<Vec<_>>();
    
    html! {
        <VirtualList 
            :items={items} 
            :item_height={50.0} 
            :container_height={400.0}
            :renderer={Box::new(|item: &String, index: usize| {
                html! { <div class="list-item">{{ index }}: {{ item }}</div> }
            })}
        />
    }
}
```

### Incremental Rendering Pattern

Break up heavy rendering work into smaller chunks:

```rust
use orbit::prelude::*;
use orbit::schedule::*;

#[component]
pub struct IncrementalRenderer<T: Clone + 'static> {
    items: Prop<Vec<T>>,
    chunk_size: Prop<usize>,
    rendered_count: State<usize>,
    renderer: Prop<Box<dyn Fn(&T, usize) -> Node>>,
}

impl<T: Clone + 'static> IncrementalRenderer<T> {
    pub fn new() -> Self {
        Self {
            items: Prop::new(),
            chunk_size: Prop::with_default(20),
            rendered_count: State::new(0),
            renderer: Prop::new(),
        }
    }
    
    #[lifecycle(mounted)]
    pub fn start_rendering(&self) {
        // Initial render of first chunk
        self.render_next_chunk();
    }
    
    pub fn render_next_chunk(&self) {
        let items_len = self.items.get().len();
        let current_count = *self.rendered_count.get();
        let chunk_size = *self.chunk_size.get();
        
        if current_count < items_len {
            let new_count = (current_count + chunk_size).min(items_len);
            self.rendered_count.set(new_count);
            
            // If more items to render, schedule next chunk in next frame
            if new_count < items_len {
                schedule_animation_frame(Box::new(move || {
                    self.render_next_chunk();
                }));
            }
        }
    }
}

impl<T: Clone + 'static> Component for IncrementalRenderer<T> {
    fn render(&self) -> Node {
        let rendered_count = *self.rendered_count.get();
        let renderer = self.renderer.get();
        
        html! {
            <div class="incremental-renderer">
                {
                    self.items.get()[0..rendered_count].iter().enumerate().map(|(i, item)| {
                        renderer(item, i)
                    }).collect::<Vec<_>>()
                }
                
                {
                    if rendered_count < self.items.get().len() {
                        html! {
                            <div class="loading-indicator">
                                Loading more items...
                                <Spinner size="small" />
                            </div>
                        }
                    } else {
                        html! { <></> }
                    }
                }
            </div>
        }
    }
}

// Usage
fn render_heavy_list() -> Node {
    let items = (0..5000).map(|i| {
        ComplexDataItem::new(/* complex data */)
    }).collect::<Vec<_>>();
    
    html! {
        <IncrementalRenderer 
            :items={items} 
            :chunk_size={50}
            :renderer={Box::new(|item: &ComplexDataItem, index: usize| {
                html! { <ComplexItemCard :item={item.clone()} :index={index} /> }
            })}
        />
    }
}
```

## 🔄 Async Component Patterns

### Suspense Pattern

Declaratively handle loading states for async data:

```rust
use orbit::prelude::*;
use orbit::suspense::*;

// Resource loader that tracks loading state
struct DataResource<T: 'static> {
    data: State<Option<T>>,
    error: State<Option<String>>,
    is_loading: State<bool>,
}

impl<T: DeserializeOwned + 'static> DataResource<T> {
    pub fn new(url: String) -> Self {
        let res = Self {
            data: State::new(None),
            error: State::new(None),
            is_loading: State::new(true),
        };
        
        // Start loading immediately
        let res_clone = res.clone();
        spawn(async move {
            match fetch_json::<T>(&url).await {
                Ok(data) => {
                    res_clone.data.set(Some(data));
                    res_clone.is_loading.set(false);
                }
                Err(err) => {
                    res_clone.error.set(Some(err.to_string()));
                    res_clone.is_loading.set(false);
                }
            }
        });
        
        res
    }
}

// Suspense component
#[component]
pub struct Suspense {
    fallback: Prop<Node>,
    suspense_resources: State<Vec<SuspenseResource>>,
}

impl Suspense {
    pub fn new() -> Self {
        Self {
            fallback: Prop::new(),
            suspense_resources: State::new(vec![]),
        }
    }
    
    pub fn register_resource(&self, resource: SuspenseResource) {
        self.suspense_resources.update(|resources| {
            resources.push(resource);
        });
    }
    
    pub fn is_loading(&self) -> bool {
        self.suspense_resources.get().iter().any(|resource| resource.is_loading())
    }
}

impl Component for Suspense {
    fn render(&self) -> Node {
        if self.is_loading() {
            self.fallback.get().clone()
        } else {
            html! { <slot></slot> }
        }
    }
}

// Usage
#[component]
pub struct UserProfile {
    user_id: Prop<String>,
    user_resource: DataResource<User>,
    posts_resource: DataResource<Vec<Post>>,
}

impl UserProfile {
    pub fn new() -> Self {
        let user_id = Prop::new();
        
        Self {
            user_id: user_id.clone(),
            user_resource: DataResource::new(format!("/api/users/{}", user_id.get())),
            posts_resource: DataResource::new(format!("/api/users/{}/posts", user_id.get())),
        }
    }
}

impl Component for UserProfile {
    fn render(&self) -> Node {
        html! {
            <Suspense fallback={html! { <LoadingSpinner /> }}>
                <SuspenseResource :resource={self.user_resource.clone()} />
                <SuspenseResource :resource={self.posts_resource.clone()} />
                
                <div class="user-profile">
                    <h1>{self.user_resource.data.get().unwrap().name}</h1>
                    <div class="bio">{self.user_resource.data.get().unwrap().bio}</div>
                    
                    <h2>Posts</h2>
                    <div class="posts">
                        {
                            self.posts_resource.data.get().unwrap().iter().map(|post| {
                                html! { <PostCard :post={post.clone()} /> }
                            }).collect::<Vec<_>>()
                        }
                    </div>
                </div>
            </Suspense>
        }
    }
}
```

## 🧪 Test-Friendly Component Patterns

### Testable Component Pattern

Design components with testing in mind:

```rust
use orbit::prelude::*;
use orbit::test_utils::*;

// Component designed for testability
#[component]
pub struct SearchForm {
    query: State<String>,
    results: State<Vec<SearchResult>>,
    is_loading: State<bool>,
    // Dependency injection for better testing
    search_service: Prop<Box<dyn SearchService>>,
    // Test IDs for better test targeting
    test_id: Prop<String>,
}

impl SearchForm {
    pub fn new() -> Self {
        Self {
            query: State::new(String::new()),
            results: State::new(vec![]),
            is_loading: State::new(false),
            search_service: Prop::with_default(Box::new(RealSearchService::new())),
            test_id: Prop::with_default("search-form".to_string()),
        }
    }
    
    pub fn update_query(&self, event: &InputEvent) {
        self.query.set(event.target_value());
    }
    
    pub async fn perform_search(&self, _: &FormEvent) {
        let query = self.query.get().clone();
        if query.is_empty() {
            return;
        }
        
        self.is_loading.set(true);
        
        match self.search_service.search(&query).await {
            Ok(results) => {
                self.results.set(results);
            }
            Err(_) => {
                self.results.set(vec![]);
            }
        }
        
        self.is_loading.set(false);
    }
}

impl Component for SearchForm {
    fn render(&self) -> Node {
        html! {
            <div class="search-container" data-testid={self.test_id.get().clone()}>
                <form 
                    class="search-form" 
                    @submit="perform_search"
                    data-testid={format!("{}-form", self.test_id.get())}
                >
                    <input 
                        type="text" 
                        :value={*self.query.get()}
                        @input="update_query"
                        placeholder="Search..."
                        data-testid={format!("{}-input", self.test_id.get())}
                    />
                    <button 
                        type="submit"
                        data-testid={format!("{}-button", self.test_id.get())}
                    >
                        Search
                    </button>
                </form>
                
                <div 
                    class="search-results {{ is_loading ? 'loading' : '' }}"
                    data-testid={format!("{}-results", self.test_id.get())}
                >
                    {
                        if *self.is_loading.get() {
                            html! { <Spinner data-testid={format!("{}-spinner", self.test_id.get())} /> }
                        } else if self.results.get().is_empty() {
                            html! { <div class="no-results" data-testid={format!("{}-no-results", self.test_id.get())}>No results found</div> }
                        } else {
                            self.results.get().iter().map(|result| {
                                html! { <SearchResultItem :result={result.clone()} /> }
                            }).collect::<Vec<_>>()
                        }
                    }
                </div>
            </div>
        }
    }
}

// Example test
#[test]
fn test_search_form() {
    // Create a mock search service
    let mock_service = Box::new(MockSearchService::new(
        vec![
            SearchResult::new("Result 1", "Description 1"),
            SearchResult::new("Result 2", "Description 2"),
        ],
    ));
    
    // Render component with mock
    let component = TestRenderer::render(html! {
        <SearchForm :search_service={mock_service} />
    });
    
    // Interact with component
    let input = component.find_by_test_id("search-form-input");
    input.set_value("test query");
    
    let button = component.find_by_test_id("search-form-button");
    button.click();
    
    // Wait for async operation
    component.wait_for_suspense();
    
    // Verify results
    let results = component.find_by_test_id("search-form-results");
    assert!(results.text_content().contains("Result 1"));
    assert!(results.text_content().contains("Result 2"));
}
```

## 🚀 Real-World Component Integration Example

Putting it all together with a complex example that uses multiple patterns:

```rust
use orbit::prelude::*;
use app::components::*;
use app::context::*;
use app::state::*;

// Main application component
#[component]
pub struct AnalyticsApplication {
    store: Store<AppState, AppAction>,
    auth: AuthState,
}

impl AnalyticsApplication {
    pub fn new() -> Self {
        Self {
            store: Store::new(AppState::default()),
            auth: AuthState::new(),
        }
    }
}

impl Component for AnalyticsApplication {
    fn render(&self) -> Node {
        html! {
            // Error boundary at the app level
            <ErrorBoundary>
                // Global context providers
                <AuthProvider :state={self.auth.clone()}>
                <ThemeProvider>
                <StoreProvider :store={self.store.clone()}>
                    
                    <AppLayout>
                        <template #header>
                            <AppHeader />
                        </template>
                        
                        <template #sidebar>
                            <ConsumeContext<AuthContext>>
                                {|auth_context| {
                                    if auth_context.is_authenticated() {
                                        html! { <NavigationSidebar /> }
                                    } else {
                                        html! { <GuestSidebar /> }
                                    }
                                }}
                            </ConsumeContext>
                        </template>
                        
                        <template #main>
                            // Lazy load routes based on authentication
                            <ConsumeContext<AuthContext>>
                                {|auth_context| {
                                    if auth_context.is_authenticated() {
                                        html! {
                                            <Router>
                                                <Route path="/" component={Dashboard} />
                                                <LazyRoute path="/reports" module="reports_module" />
                                                <LazyRoute path="/settings" module="settings_module" />
                                                <Route path="*" component={NotFound} />
                                            </Router>
                                        }
                                    } else {
                                        html! {
                                            <Router>
                                                <Route path="/" component={Welcome} />
                                                <Route path="/login" component={Login} />
                                                <Route path="/register" component={Register} />
                                                <Route path="*" component={RedirectToLogin} />
                                            </Router>
                                        }
                                    }
                                }}
                            </ConsumeContext>
                        </template>
                        
                        <template #footer>
                            <AppFooter />
                        </template>
                    </AppLayout>
                    
                </StoreProvider>
                </ThemeProvider>
                </AuthProvider>
            </ErrorBoundary>
        }
    }
}
```

## 🔄 Related Topics

- [Component Model](./component-model.md) - Learn about the basic component model
- [State Management](./state-management.md) - Understand how to manage and update component state
- [Event Handling](./event-handling.md) - Learn about Orbit's event system
- [Rendering Architecture](./rendering-architecture.md) - Dive into the rendering process
- [Performance Optimization](../guides/performance-optimization.md) - Techniques for optimizing component performance

## 📊 Performance Benchmarking Components

Understanding your component's performance characteristics is crucial for optimization. Here's a pattern for creating self-benchmarking components:

```rust
use orbit::prelude::*;
use orbit::debug::*;

#[component]
pub struct BenchmarkedComponent {
    render_times: State<Vec<f64>>,
    update_times: State<Vec<f64>>,
    benchmark_enabled: Prop<bool>,
    children: Children,
}

impl BenchmarkedComponent {
    pub fn new() -> Self {
        Self {
            render_times: State::new(vec![]),
            update_times: State::new(vec![]),
            benchmark_enabled: Prop::with_default(false),
            children: Children::new(),
        }
    }
    
    pub fn average_render_time(&self) -> f64 {
        let times = self.render_times.get();
        if times.is_empty() {
            0.0
        } else {
            times.iter().sum::<f64>() / times.len() as f64
        }
    }
    
    pub fn max_render_time(&self) -> f64 {
        self.render_times.get().iter().fold(0.0, |a, &b| a.max(b))
    }
    
    #[lifecycle(before_render)]
    pub fn start_render_timing(&self) {
        if *self.benchmark_enabled.get() {
            debug_mark_begin("component_render");
        }
    }
    
    #[lifecycle(after_render)]
    pub fn end_render_timing(&self) {
        if *self.benchmark_enabled.get() {
            let duration = debug_mark_end("component_render");
            self.render_times.update(|times| {
                times.push(duration);
                if times.len() > 100 {
                    times.remove(0);
                }
            });
        }
    }
}

impl Component for BenchmarkedComponent {
    fn render(&self) -> Node {
        if *self.benchmark_enabled.get() {
            html! {
                <div class="benchmarked-component">
                    <div class="benchmark-stats" style="position: absolute; top: 0; right: 0; background: rgba(0,0,0,0.7); color: white; padding: 5px; font-size: 12px; z-index: 9999;">
                        <div>Avg render: {format!("{:.2}ms", self.average_render_time())}</div>
                        <div>Max render: {format!("{:.2}ms", self.max_render_time())}</div>
                    </div>
                    <div class="component-content">
                        {self.children.render()}
                    </div>
                </div>
            }
        } else {
            self.children.render()
        }
    }
}

// Usage
fn app() -> Node {
    html! {
        <BenchmarkedComponent :benchmark_enabled={cfg!(debug_assertions)}>
            <ComplexComponent />
        </BenchmarkedComponent>
    }
}
```

## 📱 Responsive Component Patterns

Create truly adaptive components that respond to various device capabilities:

```rust
use orbit::prelude::*;
use orbit::responsive::*;

// Breakpoint system
pub struct Breakpoints {
    pub xs: f32,
    pub sm: f32,
    pub md: f32,
    pub lg: f32,
    pub xl: f32,
}

impl Default for Breakpoints {
    fn default() -> Self {
        Self {
            xs: 0.0,
            sm: 640.0,
            md: 768.0,
            lg: 1024.0,
            xl: 1280.0,
        }
    }
}

// Media query hook
pub fn use_media_query(query: String) -> UseMediaQueryHook {
    let matches = State::new(false);
    
    // Set up media query listener
    orbit::effects::use_effect(move || {
        let media_query = window().match_media(&query).unwrap();
        matches.set(media_query.matches());
        
        let matches_clone = matches.clone();
        let listener = EventListener::new(&media_query, "change", move |event| {
            let media_event = event.dyn_ref::<MediaQueryListEvent>().unwrap();
            matches_clone.set(media_event.matches());
        });
        
        // Cleanup function
        move || {
            drop(listener);
        }
    });
    
    UseMediaQueryHook { matches }
}

// Responsive component
#[component]
pub struct ResponsiveLayout {
    breakpoints: Prop<Breakpoints>,
    is_mobile: Computed<bool>,
    is_tablet: Computed<bool>,
    is_desktop: Computed<bool>,
}

impl ResponsiveLayout {
    pub fn new() -> Self {
        let breakpoints = Prop::with_default(Breakpoints::default());
        
        // Create responsive hooks
        let mobile_query = use_media_query(format!("(max-width: {}px)", breakpoints.get().sm));
        let tablet_query = use_media_query(format!("(min-width: {}px) and (max-width: {}px)", 
                                                 breakpoints.get().sm, breakpoints.get().lg));
        let desktop_query = use_media_query(format!("(min-width: {}px)", breakpoints.get().lg));
        
        Self {
            breakpoints,
            is_mobile: Computed::from(mobile_query.matches),
            is_tablet: Computed::from(tablet_query.matches),
            is_desktop: Computed::from(desktop_query.matches),
        }
    }
}

impl Component for ResponsiveLayout {
    fn render(&self) -> Node {
        html! {
            <div class="responsive-layout {{ is_mobile ? 'mobile' : is_tablet ? 'tablet' : 'desktop' }}">
                {
                    if *self.is_mobile.get() {
                        html! { <slot name="mobile"><slot></slot></slot> }
                    } else if *self.is_tablet.get() {
                        html! { <slot name="tablet"><slot></slot></slot> }
                    } else {
                        html! { <slot name="desktop"><slot></slot></slot> }
                    }
                }
            </div>
        }
    }
}

// Usage
fn app() -> Node {
    html! {
        <ResponsiveLayout>
            <template #mobile>
                <MobileNavigation />
                <ContentStack />
            </template>
            
            <template #tablet>
                <TabletSidebar />
                <MainContent />
            </template>
            
            <template #desktop>
                <DesktopHeader />
                <SidebarLayout>
                    <Navigation />
                    <Content />
                </SidebarLayout>
            </template>
        </ResponsiveLayout>
    }
}
```
