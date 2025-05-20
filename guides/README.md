# 📝 Guides

This section contains comprehensive guides and best practices for developing with the Orbit Framework.

## 📋 Table of Contents

- [Performance Optimization](#performance-optimization)
- [Accessibility](#accessibility)
- [Cross-Platform Development](#cross-platform-development)
- [State Management Patterns](#state-management-patterns)
- [Testing Strategies](#testing-strategies)
- [Styling Best Practices](#styling-best-practices)
- [Deployment](#deployment)

---

## Performance Optimization

### Component Optimization

Efficient components are the foundation of a performant application:

1. **Minimize State Updates**: Avoid unnecessary state changes that trigger re-renders.

```rust
// Avoid this pattern
fn increment_counter(&mut self) {
    self.state.count += 1;
    self.state.last_updated = Some(Instant::now()); // Unnecessary for every update
}

// Better approach
fn increment_counter(&mut self) {
    self.state.count += 1;
    // Only update timestamp on significant changes
    if self.state.count % 10 == 0 {
        self.state.last_updated = Some(Instant::now());
    }
}
```

2. **Implement `should_update()`**: Control when your component re-renders.

```rust
fn should_update(&self, next_props: &Props, next_state: &State) -> bool {
    // Only update if these specific properties changed
    self.props.user_id != next_props.user_id || 
    self.state.selected_item != next_state.selected_item
}
```

3. **Memoize Expensive Calculations**: Cache results of expensive operations.

```rust
<Component>
  <Props>
    data: Vec<DataPoint>
  </Props>

  <State>
    cached_result: Option<ProcessedData> = None
    last_data_hash: u64 = 0
  </State>

  <Script>
    fn get_processed_data(&mut self) -> &ProcessedData {
        let current_hash = hash(&self.props.data);
        
        if self.state.cached_result.is_none() || self.state.last_data_hash != current_hash {
            let processed = process_data(&self.props.data); // Expensive operation
            self.state.cached_result = Some(processed);
            self.state.last_data_hash = current_hash;
        }
        
        self.state.cached_result.as_ref().unwrap()
    }
  </Script>
</Component>
```

### Rendering Optimization

Optimize how your application renders:

1. **Virtualize Long Lists**: Only render visible items.

```rust
<Component>
  <Props>
    items: Vec<Item>
  </Props>

  <Template>
    <VirtualList
      items={self.props.items}
      height={500}
      item_height={50}
      render_item={|item| (
        <ItemRow item={item} />
      )}
    />
  </Template>
</Component>
```

2. **Lazy Load Components**: Load components only when needed.

```rust
<Template>
  <LazyLoad when={self.is_section_visible}>
    <ExpensiveComponent />
  </LazyLoad>
</Template>
```

3. **Optimize Images and Assets**: Use appropriate formats and sizes.

```rust
<Template>
  <Picture>
    <Source media="(max-width: 600px)" srcset="small.webp" type="image/webp" />
    <Source media="(max-width: 600px)" srcset="small.jpg" />
    <Source srcset="large.webp" type="image/webp" />
    <Image src="large.jpg" alt="Description" loading="lazy" />
  </Picture>
</Template>
```

### Build Optimization

Optimize your build for production:

1. **Code Splitting**: Split your code into smaller chunks.

```rust
// main.rs
fn main() {
    App::new()
        .with_code_splitting(true)
        .run();
}
```

2. **Tree Shaking**: Remove unused code.

```toml
# orbiton.config.toml
[build]
tree_shaking = true
```

3. **Asset Optimization**: Minimize asset sizes.

```toml
# orbiton.config.toml
[build.assets]
optimize = true
image_compression = true
```

---

## Accessibility

### Semantic Markup

Use the right HTML elements for the right purpose:

```rust
// Instead of this
<div class="button" on:click={self.submit}>Submit</div>

// Use this
<button on:click={self.submit}>Submit</button>
```

### ARIA Attributes

Enhance accessibility with ARIA when necessary:

```rust
<Template>
  <div 
    role="tablist"
    aria-label="Product Information"
  >
    <button 
      role="tab"
      id="tab-specs"
      aria-selected={self.state.active_tab == "specs"}
      aria-controls="panel-specs"
      on:click={|_| self.change_tab("specs")}
    >
      Specifications
    </button>
    <!-- More tabs -->
    
    <div 
      role="tabpanel"
      id="panel-specs"
      aria-labelledby="tab-specs"
      hidden={self.state.active_tab != "specs"}
    >
      <!-- Panel content -->
    </div>
  </div>
</Template>
```

### Keyboard Navigation

Ensure keyboard accessibility:

```rust
<Component>
  <State>
    focused_index: i32 = -1
    items: Vec<MenuItem> = vec![]
  </State>

  <Template>
    <ul
      role="menu"
      tabindex="0"
      on:keydown={self.handle_keyboard_navigation}
    >
      {self.state.items.iter().enumerate().map(|(index, item)| (
        <li 
          role="menuitem"
          tabindex="-1"
          class:focused={index as i32 == self.state.focused_index}
          ref={format!("item_{}", index)}
        >
          {item.label}
        </li>
      ))}
    </ul>
  </Template>

  <Script>
    fn handle_keyboard_navigation(&mut self, e: KeyboardEvent) {
        match e.key.as_str() {
            "ArrowDown" => {
                e.prevent_default();
                self.state.focused_index = min(
                    self.state.focused_index + 1, 
                    self.state.items.len() as i32 - 1
                );
                self.focus_current_item();
            },
            "ArrowUp" => {
                e.prevent_default();
                self.state.focused_index = max(self.state.focused_index - 1, 0);
                self.focus_current_item();
            },
            "Enter" => {
                if self.state.focused_index >= 0 {
                    self.select_item(self.state.focused_index as usize);
                }
            },
            _ => {}
        }
    }
    
    fn focus_current_item(&self) {
        if self.state.focused_index >= 0 {
            if let Some(element) = self.refs.get(&format!("item_{}", self.state.focused_index)) {
                element.focus();
            }
        }
    }
  </Script>
</Component>
```

### Screen Reader Support

Provide appropriate text alternatives:

```rust
<Template>
  <button on:click={self.close}>
    <Icon name="close" />
    <span class="sr-only">Close Dialog</span>
  </button>
</Template>

<Style>
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border-width: 0;
  }
</Style>
```

---

## Cross-Platform Development

### Platform Detection

Detect and adapt to different platforms:

```rust
<Script>
  fn mount(&mut self) {
    self.state.is_mobile = Platform::is_mobile();
    self.state.is_desktop = Platform::is_desktop();
    self.state.is_web = Platform::is_web();
    self.state.is_ios = Platform::is_ios();
    self.state.is_android = Platform::is_android();
  }
</Script>
```

### Adaptive Layouts

Design layouts that work across devices:

```rust
<Style>
  .container {
    /* Mobile-first approach */
    width: 100%;
    padding: 16px;
    
    /* Tablet */
    @media (min-width: 768px) {
      width: 720px;
      margin: 0 auto;
      padding: 24px;
    }
    
    /* Desktop */
    @media (min-width: 1024px) {
      width: 960px;
      padding: 32px;
    }
  }
</Style>
```

### Platform-Specific Code

Handle platform differences gracefully:

```rust
<Script>
  fn get_file_access(&self) -> Result<FileAccess, Error> {
    #[cfg(feature = "desktop")]
    {
      return FileAccess::native_file_dialog();
    }
    
    #[cfg(feature = "web")]
    {
      return FileAccess::web_file_input();
    }
    
    #[cfg(feature = "mobile")]
    {
      return FileAccess::mobile_document_picker();
    }
    
    Err(Error::PlatformNotSupported)
  }
</Script>
```

---

## State Management Patterns

### Component State

For component-specific state:

```rust
<Component>
  <State>
    count: i32 = 0
    loading: bool = false
    error: Option<String> = None
  </State>
</Component>
```

### Context API

For state that needs to be shared among a component tree:

```rust
// Provider Component
<Component>
  <Props>
    children: Children
  </Props>
  
  <State>
    user: Option<User> = None
    theme: Theme = Theme::Light
  </State>

  <Template>
    <CreateContext id="app_state" value={self.get_context_value()}>
      {self.props.children}
    </CreateContext>
  </Template>

  <Script>
    fn get_context_value(&self) -> AppContext {
      AppContext {
        user: self.state.user.clone(),
        theme: self.state.theme,
        login: Callback::new(|user| self.login(user)),
        logout: Callback::new(|| self.logout()),
        toggle_theme: Callback::new(|| self.toggle_theme()),
      }
    }
    
    fn login(&mut self, user: User) {
      self.state.user = Some(user);
    }
    
    fn logout(&mut self) {
      self.state.user = None;
    }
    
    fn toggle_theme(&mut self) {
      self.state.theme = match self.state.theme {
        Theme::Light => Theme::Dark,
        Theme::Dark => Theme::Light,
      };
    }
  </Script>
</Component>

// Consumer Component
<Component>
  <Template>
    <UseContext id="app_state" as="app">
      <div>
        {#if app.user.is_some() }
          <p>Welcome, {app.user.unwrap().name}!</p>
          <button on:click={|_| app.logout.emit()}>Logout</button>
        {#else}
          <button on:click={|_| self.show_login_form()}>Login</button>
        {/if}
        
        <button on:click={|_| app.toggle_theme.emit()}>
          Toggle {app.theme == Theme::Light ? "Dark" : "Light"} Mode
        </button>
      </div>
    </UseContext>
  </Template>
</Component>
```

### Global Store

For application-wide state management:

```rust
// store.rs
use orbit::state::{Store, Selector};

pub struct AppState {
    counter: i32,
    user: Option<User>,
    products: Vec<Product>,
}

impl Default for AppState {
    fn default() -> Self {
        Self {
            counter: 0,
            user: None,
            products: Vec::new(),
        }
    }
}

pub fn create_store() -> Store<AppState> {
    Store::new(AppState::default())
}

// Selectors
pub fn select_counter(state: &AppState) -> i32 {
    state.counter
}

pub fn select_user(state: &AppState) -> Option<User> {
    state.user.clone()
}

// Actions
pub fn increment(store: &Store<AppState>) {
    store.update(|state| {
        state.counter += 1;
    });
}

pub fn set_user(store: &Store<AppState>, user: User) {
    store.update(|state| {
        state.user = Some(user);
    });
}

// In your component
<Component>
  <Template>
    <div>
      <UseStore store={store} select={select_counter} as="counter">
        Counter: {counter}
        <button on:click={|_| increment(&store)}>Increment</button>
      </UseStore>
      
      <UseStore store={store} select={select_user} as="user">
        {#if user.is_some() }
          <p>Welcome, {user.unwrap().name}!</p>
        {#else}
          <button on:click={|_| self.login()}>Login</button>
        {/if}
      </UseStore>
    </div>
  </Template>

  <Script>
    fn login(&self) {
      // After authentication
      let user = User { name: "John Doe".to_string(), /* ... */ };
      set_user(&store, user);
    }
  </Script>
</Component>
```

---

## Testing Strategies

### Unit Testing Components

Test individual components:

```rust
#[cfg(test)]
mod tests {
    use orbit_test::prelude::*;
    use super::Counter;

    #[test]
    fn test_counter_increment() {
        // Arrange
        let mut test_runner = TestRunner::new();
        let counter = test_runner.render_component(Counter::new().with_prop("initial_count", 0));
        
        // Act
        counter.find_button("Increment").click();
        
        // Assert
        assert_eq!(counter.state().count, 1);
        assert_eq!(counter.find_text(".count").text(), "1");
    }
    
    #[test]
    fn test_counter_decrement() {
        // Arrange
        let mut test_runner = TestRunner::new();
        let counter = test_runner.render_component(Counter::new().with_prop("initial_count", 2));
        
        // Act
        counter.find_button("Decrement").click();
        
        // Assert
        assert_eq!(counter.state().count, 1);
        assert_eq!(counter.find_text(".count").text(), "1");
    }
}
```

### Integration Testing

Test component interactions:

```rust
#[cfg(test)]
mod tests {
    use orbit_test::prelude::*;
    use super::{App, TodoList, TodoForm};

    #[test]
    fn test_add_todo() {
        // Arrange
        let mut test_runner = TestRunner::new();
        let app = test_runner.render_component(App::new());
        
        // Act
        let form = app.find_component(TodoForm);
        form.find_input("new-todo").type_text("Buy milk");
        form.find_button("Add").click();
        
        // Assert
        let todo_list = app.find_component(TodoList);
        assert_eq!(todo_list.props().todos.len(), 1);
        assert_eq!(todo_list.props().todos[0], "Buy milk");
        assert!(app.find_text("Buy milk").exists());
    }
}
```

### End-to-End Testing

Test the entire application:

```rust
#[cfg(test)]
mod e2e_tests {
    use orbit_test::e2e::*;

    #[test]
    fn test_user_flow() {
        // Arrange
        let mut app = launch_app("http://localhost:3000");
        
        // Act - Login flow
        app.navigate_to("/login");
        app.fill_input("username", "testuser");
        app.fill_input("password", "password123");
        app.click_button("Login");
        
        // Assert - Dashboard loaded
        app.wait_for_navigation();
        assert_eq!(app.current_url(), "http://localhost:3000/dashboard");
        assert!(app.find_text("Welcome, testuser").exists());
        
        // Act - Create item
        app.click_button("New Item");
        app.fill_input("item-name", "Test Item");
        app.click_button("Save");
        
        // Assert - Item created
        app.wait_for_element("item-list");
        assert!(app.find_text("Test Item").exists());
    }
}
```

---

## Styling Best Practices

### Component-Scoped Styles

Keep styles encapsulated within components:

```rust
<Component>
  <Style>
    /* These styles only apply to this component */
    .button {
      padding: 8px 16px;
      border-radius: 4px;
      background-color: #3498db;
      color: white;
    }
    
    .button:hover {
      background-color: #2980b9;
    }
  </Style>
</Component>
```

### Theming

Create consistent themes:

```rust
// theme.rs
pub struct Theme {
    pub primary: Color,
    pub secondary: Color,
    pub background: Color,
    pub text: Color,
    pub spacing: Spacing,
    pub border_radius: BorderRadius,
}

impl Theme {
    pub fn light() -> Self {
        Self {
            primary: Color::rgb(52, 152, 219),
            secondary: Color::rgb(46, 204, 113),
            background: Color::rgb(255, 255, 255),
            text: Color::rgb(33, 33, 33),
            spacing: Spacing::default(),
            border_radius: BorderRadius::default(),
        }
    }
    
    pub fn dark() -> Self {
        Self {
            primary: Color::rgb(41, 128, 185),
            secondary: Color::rgb(39, 174, 96),
            background: Color::rgb(18, 18, 18),
            text: Color::rgb(238, 238, 238),
            spacing: Spacing::default(),
            border_radius: BorderRadius::default(),
        }
    }
}

// In your component
<Component>
  <Style>
    .card {
      background-color: $theme.background;
      color: $theme.text;
      border-radius: $theme.border_radius.medium;
      padding: $theme.spacing.medium;
    }
    
    .button {
      background-color: $theme.primary;
      color: white;
      padding: $theme.spacing.small $theme.spacing.medium;
      border-radius: $theme.border_radius.small;
    }
  </Style>
</Component>
```

### Responsive Design

Create adaptive user interfaces:

```rust
<Style>
  .layout {
    display: flex;
    flex-direction: column;
    
    /* Tablet and above */
    @media (min-width: 768px) {
      flex-direction: row;
    }
  }
  
  .sidebar {
    width: 100%;
    
    @media (min-width: 768px) {
      width: 250px;
      flex-shrink: 0;
    }
    
    @media (min-width: 1200px) {
      width: 300px;
    }
  }
  
  .content {
    flex: 1;
    padding: $theme.spacing.medium;
    
    @media (min-width: 768px) {
      padding: $theme.spacing.large;
    }
  }
</Style>
```

---

## Deployment

### Web Deployment

Deploy your Orbit web application:

1. **Build for Production**:

```bash
orbiton build --target web --optimize
```

2. **Static Site Deployment**:

```bash
# Deploy to Netlify
orbiton deploy --target netlify

# Deploy to Vercel
orbiton deploy --target vercel

# Deploy to GitHub Pages
orbiton deploy --target github-pages
```

3. **Configure Web Server**:

For manual deployment, ensure your web server configuration:
- Sets appropriate MIME types
- Configures cache headers
- Handles routing for SPA navigation

### Desktop Deployment

Package your desktop application:

1. **Build for Desktop**:

```bash
orbiton build --target desktop
```

2. **Create Installers**:

```bash
# Windows installer
orbiton package --target windows --installer

# macOS DMG
orbiton package --target macos --installer

# Linux packages
orbiton package --target linux --formats deb,rpm,AppImage
```

3. **Code Signing**:

```bash
# Sign Windows installer
orbiton sign --target windows --cert cert.pfx --password $CERT_PASSWORD

# Sign macOS app
orbiton sign --target macos --identity "Developer ID Application: Your Name"
```

### Mobile Deployment

Deploy to mobile app stores:

1. **Build for Mobile**:

```bash
orbiton build --target ios
orbiton build --target android
```

2. **Package for App Stores**:

```bash
# iOS App Store
orbiton package --target ios --app-store

# Google Play Store
orbiton package --target android --bundle
```

3. **Testing Deployments**:

```bash
# TestFlight (iOS)
orbiton deploy --target testflight --api-key $APPSTORE_API_KEY

# Firebase App Distribution (Android)
orbiton deploy --target firebase --app-id $FIREBASE_APP_ID
```

---

> Note: These guides are continuously updated as the framework evolves. Refer to the documentation for the most current best practices.
