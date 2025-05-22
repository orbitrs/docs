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
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;
use orbitrs::context::*;
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
use orbitrs::prelude::*;
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
use orbitrs::prelude::*;
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
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;
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
use orbitrs::prelude::*;
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
use orbitrs::prelude::*;
use orbitrs::hooks::*;

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
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;
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

## 🔄 Related Topics

- [Component Model](./component-model.md) - Learn about the basic component model
- [State Management](./state-management.md) - Understand how to manage and update component state
- [Event Handling](./event-handling.md) - Learn about Orbit's event system
- [Rendering Architecture](./rendering-architecture.md) - Dive into the rendering process
