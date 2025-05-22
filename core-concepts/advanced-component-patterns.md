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

## 🔧 Performance Optimizations

### Memoization

Prevent unnecessary re-renders with memoization:

```rust
use orbit::prelude::*;

#[component]
pub struct ExpensiveComponent {
    data: Prop<Vec<Item>>,
    filtered_data: Computed<Vec<Item>>,
}

impl ExpensiveComponent {
    pub fn new() -> Self {
        let data = Prop::new();
        
        // Computed value that only recalculates when data changes
        let filtered_data = Computed::new(move || {
            data.get().iter()
                .filter(|item| item.is_active)
                .cloned()
                .collect()
        });
        
        Self {
            data,
            filtered_data,
        }
    }
}
```

### Component Keying

Use keys to help Orbit optimize rendering of lists:

```orbit
<template>
  <ul>
    <!-- Using the unique ID as a key helps Orbit efficiently update only changed items -->
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>
```

## 🏗️ Dynamic Components

Create components that render different child components based on props:

```orbit
<template>
  <component :is="current_view">
    <slot></slot>
  </component>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct DynamicView {
    current_view: Prop<Box<dyn Component>>,
}

impl DynamicView {
    pub fn new() -> Self {
        Self {
            current_view: Prop::new(),
        }
    }
}
</script>

<!-- Usage -->
<template>
  <div>
    <button @click="switch_view('list')">List View</button>
    <button @click="switch_view('grid')">Grid View</button>
    <button @click="switch_view('table')">Table View</button>
    
    <DynamicView :current_view="current_view_component">
      <UserData :users="users" />
    </DynamicView>
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;
use app::views::{ListView, GridView, TableView};

#[component]
pub struct ViewSwitcher {
    current_view: State<String>,
    current_view_component: Computed<Box<dyn Component>>,
}

impl ViewSwitcher {
    pub fn new() -> Self {
        let current_view = State::new("list".to_string());
        
        let current_view_component = Computed::new(move || {
            match current_view.get().as_str() {
                "list" => Box::new(ListView::new()) as Box<dyn Component>,
                "grid" => Box::new(GridView::new()) as Box<dyn Component>,
                "table" => Box::new(TableView::new()) as Box<dyn Component>,
                _ => Box::new(ListView::new()) as Box<dyn Component>,
            }
        });
        
        Self {
            current_view,
            current_view_component,
        }
    }
    
    pub fn switch_view(&self, view: &str) {
        self.current_view.set(view.to_string());
    }
}
</script>
```

## 🛡️ Error Boundaries

Error Boundaries are components that catch JavaScript errors in their child component tree and display a fallback UI instead of crashing the entire application.

```rust
use orbit::prelude::*;

#[component]
pub struct ErrorBoundary {
    has_error: State<bool>,
    error_info: State<Option<ErrorInfo>>,
    children: Children,
    fallback: Option<Box<dyn Fn(&ErrorInfo) -> Node>>,
}

impl ErrorBoundary {
    pub fn new() -> Self {
        Self {
            has_error: State::new(false),
            error_info: State::new(None),
            children: Children::new(),
            fallback: None,
        }
    }
    
    // Called when an error occurs in a descendant component
    pub fn catch_error(&self, error: Error, component_stack: String) {
        self.has_error.set(true);
        self.error_info.set(Some(ErrorInfo {
            error,
            component_stack,
        }));
    }
    
    // Reset the error state to try rendering children again
    pub fn reset(&self) {
        self.has_error.set(false);
        self.error_info.set(None);
    }
}

impl Component for ErrorBoundary {
    fn render(&self) -> Node {
        if *self.has_error.get() {
            if let Some(fallback) = &self.fallback {
                return fallback(self.error_info.get().as_ref().unwrap());
            } else {
                return html! {
                    <div class="error-boundary">
                        <h2>Something went wrong.</h2>
                        <button @click="reset">Try again</button>
                    </div>
                };
            }
        }
        
        // Render children normally if no error
        self.children.render()
    }
}
```

Usage example:

```orbit
<template>
  <ErrorBoundary>
    <template #fallback="{ error }">
      <div class="error-container">
        <h2>Component Error</h2>
        <p>{{ error.message }}</p>
        <button @click="retry">Retry</button>
      </div>
    </template>
    
    <UserProfile :id="user_id" />
  </ErrorBoundary>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct UserPage {
    user_id: Prop<String>,
    
    pub fn retry(&self) {
        // Reset the error boundary and try again
        self.$refs.error_boundary.reset();
    }
}
</code>
```

## 🌐 Context API

The Context API provides a way to share values between components without having to explicitly pass props through every level of the component tree.

```rust
use orbit::prelude::*;
use orbit::context::{ProvideContext, ConsumeContext};

// Define your context type
pub struct ThemeContext {
    pub theme: String,
    pub toggle_theme: Box<dyn Fn()>,
}

// Provider component
#[component]
pub struct ThemeProvider {
    theme: State<String>,
    children: Children,
}

impl ThemeProvider {
    pub fn new() -> Self {
        Self {
            theme: State::new("light".to_string()),
            children: Children::new(),
        }
    }
    
    pub fn toggle_theme(&self) {
        let current = self.theme.get();
        if current == "light" {
            self.theme.set("dark".to_string());
        } else {
            self.theme.set("light".to_string());
        }
    }
}

impl Component for ThemeProvider {
    fn render(&self) -> Node {
        let theme_context = ThemeContext {
            theme: self.theme.get().clone(),
            toggle_theme: Box::new(move || self.toggle_theme()),
        };
        
        html! {
            <ProvideContext value={theme_context}>
                {self.children.render()}
            </ProvideContext>
        }
    }
}

// Consumer component
#[component]
pub struct ThemedButton {
    label: Prop<String>,
    on_click: EventEmitter<ClickEvent>,
}

impl ThemedButton {
    pub fn new() -> Self {
        Self {
            label: Prop::with_default("Click me".to_string()),
            on_click: EventEmitter::new(),
        }
    }
}

impl Component for ThemedButton {
    fn render(&self) -> Node {
        html! {
            <ConsumeContext<ThemeContext>>
                {|context| {
                    let theme_class = format!("button-{}", context.theme);
                    
                    html! {
                        <button 
                            class={theme_class} 
                            @click="handle_click">
                            {self.label.get()}
                        </button>
                    }
                }}
            </ConsumeContext>
        }
    }
    
    fn handle_click(&self, event: &ClickEvent) {
        self.on_click.emit(event);
    }
}
```

Usage example:

```orbit
<template>
  <ThemeProvider>
    <div class="app">
      <ThemedHeader title="My App" />
      <ThemedButton label="Toggle Theme" @click="handle_toggle" />
      <ThemedContent>
        <!-- App content -->
      </ThemedContent>
    </div>
  </ThemeProvider>
</template>

<code lang="rust">
use orbit::prelude::*;
use orbit::context::ConsumeContext;
use app::theme::ThemeContext;

#[component]
pub struct App {}

impl App {
    fn handle_toggle(&self) {
        // The actual toggle is handled by the ThemeProvider
    }
}
</code>
```

## 🧩 State Machines in Components

Using state machines to manage complex component behavior:

```rust
use orbit::prelude::*;
use state_machine::{StateMachine, Transition, State};

#[derive(Clone, PartialEq)]
enum FormState {
    Editing,
    Submitting,
    Success,
    Error,
}

#[derive(Clone)]
enum FormEvent {
    Submit,
    Reset,
    ApiSuccess,
    ApiFailure,
}

#[component]
pub struct ComplexForm {
    machine: StateMachine<FormState, FormEvent>,
    form_data: State<FormData>,
    error: State<Option<String>>,
}

impl ComplexForm {
    pub fn new() -> Self {
        // Define the state machine transitions
        let mut machine = StateMachine::new(FormState::Editing);
        
        machine
            .add_transition(FormState::Editing, FormEvent::Submit, FormState::Submitting)
            .add_transition(FormState::Submitting, FormEvent::ApiSuccess, FormState::Success)
            .add_transition(FormState::Submitting, FormEvent::ApiFailure, FormState::Error)
            .add_transition(FormState::Error, FormEvent::Reset, FormState::Editing)
            .add_transition(FormState::Success, FormEvent::Reset, FormState::Editing);
        
        Self {
            machine,
            form_data: State::new(FormData::default()),
            error: State::new(None),
        }
    }
    
    pub fn submit(&self) {
        self.machine.send(FormEvent::Submit);
        
        // Start API submission
        orbit::spawn(async move {
            match submit_form_data(self.form_data.get()).await {
                Ok(_) => {
                    self.machine.send(FormEvent::ApiSuccess);
                },
                Err(e) => {
                    self.error.set(Some(e.to_string()));
                    self.machine.send(FormEvent::ApiFailure);
                }
            }
        });
    }
    
    pub fn reset(&self) {
        self.form_data.set(FormData::default());
        self.error.set(None);
        self.machine.send(FormEvent::Reset);
    }
}

impl Component for ComplexForm {
    fn render(&self) -> Node {
        match self.machine.current_state() {
            FormState::Editing => html! {
                <form @submit="handle_submit">
                    <!-- Form fields -->
                    <button type="submit">Submit</button>
                </form>
            },
            FormState::Submitting => html! {
                <div class="loading">
                    <Spinner />
                    <p>Submitting your data...</p>
                </div>
            },
            FormState::Success => html! {
                <div class="success">
                    <Icon name="check-circle" />
                    <p>Form submitted successfully!</p>
                    <button @click="reset">Start Over</button>
                </div>
            },
            FormState::Error => html! {
                <div class="error">
                    <Icon name="alert-triangle" />
                    <p>Error: {self.error.get().clone().unwrap_or_default()}</p>
                    <button @click="reset">Try Again</button>
                </div>
            },
        }
    }
}
```

## 🌟 Lazy Loading Components

Implementing lazy loading for better performance:

```rust
use orbit::prelude::*;
use orbit::lazy::{LazyComponent, load_component};

#[component]
pub struct LazyLoadedPage {
    is_loaded: State<bool>,
    component: State<Option<Box<dyn Component>>>,
}

impl LazyLoadedPage {
    pub fn new() -> Self {
        Self {
            is_loaded: State::new(false),
            component: State::new(None),
        }
    }
    
    pub fn mounted(&self) {
        // Load the component only when this component is mounted
        orbit::spawn(async move {
            let component = load_component("analytics_dashboard").await;
            self.component.set(Some(component));
            self.is_loaded.set(true);
        });
    }
}

impl Component for LazyLoadedPage {
    fn render(&self) -> Node {
        if *self.is_loaded.get() {
            if let Some(component) = &*self.component.get() {
                return component.render();
            }
        }
        
        html! {
            <div class="loading">
                <Spinner />
                <p>Loading dashboard...</p>
            </div>
        }
    }
}
```

Usage in routing:

```rust
use orbit::prelude::*;
use orbit::router::{Router, Route};
use orbit::lazy::LazyRoute;

fn setup_router() -> Router {
    Router::new()
        .route("/", Route::component(HomePage::new))
        .route("/about", Route::component(AboutPage::new))
        // Lazy loaded routes
        .route("/dashboard", LazyRoute::new("dashboard_module"))
        .route("/reports", LazyRoute::new("reports_module"))
        .route("/settings", LazyRoute::new("settings_module"))
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
