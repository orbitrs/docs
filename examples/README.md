# 🌟 Examples

This section contains a variety of examples to help you learn how to use the Orbit Framework effectively. Each example demonstrates specific features and patterns.

## 🚀 Quick Start Examples

### 1. Hello World

The simplest possible Orbit application:

```rust
// src/app.orbit
<Component>
  <Template>
    <div>Hello, Orbit!</div>
  </Template>
</Component>

// src/main.rs
use orbit::prelude::*;

fn main() {
    App::new()
        .with_component::<app::App>()
        .run();
}
```

### 2. Counter

A basic counter that demonstrates state management:

```rust
// src/components/counter.orbit
<Component>
  <State>
    count: i32 = 0
  </State>

  <Template>
    <div>
      <button on:click={self.decrement}>-</button>
      <span>{self.state.count}</span>
      <button on:click={self.increment}>+</button>
    </div>
  </Template>

  <Script>
    fn increment(&mut self) {
        self.state.count += 1;
    }
    
    fn decrement(&mut self) {
        self.state.count -= 1;
    }
  </Script>
</Component>
```

### 3. Todo List

A simple todo list application:

```rust
// src/components/todo_list.orbit
<Component>
  <State>
    todos: Vec<String> = Vec::new()
    new_todo: String = String::new()
  </State>

  <Template>
    <div>
      <h1>Todo List</h1>
      
      <div>
        <input 
          value={self.state.new_todo}
          on:input={self.update_new_todo}
          placeholder="What needs to be done?"
        />
        <button on:click={self.add_todo}>Add</button>
      </div>
      
      <ul>
        {self.state.todos.iter().enumerate().map(|(index, todo)| (
          <li>
            {todo}
            <button on:click={move |_| self.remove_todo(index)}>✓</button>
          </li>
        ))}
      </ul>
    </div>
  </Template>

  <Script>
    fn update_new_todo(&mut self, e: InputEvent) {
        self.state.new_todo = e.value;
    }
    
    fn add_todo(&mut self) {
        if !self.state.new_todo.trim().is_empty() {
            self.state.todos.push(self.state.new_todo.clone());
            self.state.new_todo = String::new();
        }
    }
    
    fn remove_todo(&mut self, index: usize) {
        self.state.todos.remove(index);
    }
  </Script>
</Component>
```

## 🧩 Component Patterns

### 1. Component Composition

```rust
// src/components/user_card.orbit
<Component>
  <Props>
    user: User
  </Props>

  <Template>
    <div class="user-card">
      <Avatar src={self.props.user.avatar_url} size="medium" />
      <div class="user-info">
        <h3>{self.props.user.name}</h3>
        <p>{self.props.user.email}</p>
      </div>
    </div>
  </Template>

  <Style>
    .user-card {
      display: flex;
      align-items: center;
      gap: 16px;
      padding: 16px;
      border-radius: 8px;
      background-color: #f5f5f5;
    }
    
    .user-info {
      display: flex;
      flex-direction: column;
    }
    
    .user-info h3 {
      margin: 0;
      font-size: 18px;
    }
    
    .user-info p {
      margin: 4px 0 0 0;
      color: #666;
    }
  </Style>
</Component>

// src/components/user_list.orbit
<Component>
  <Props>
    users: Vec<User>
  </Props>

  <Template>
    <div class="user-list">
      {self.props.users.iter().map(|user| (
        <UserCard user={user.clone()} />
      ))}
    </div>
  </Template>

  <Style>
    .user-list {
      display: flex;
      flex-direction: column;
      gap: 16px;
    }
  </Style>
</Component>
```

### 2. Context API

```rust
// src/context/theme.orbit
<Component>
  <Props>
    children: Children
    dark_mode: bool = false
  </Props>

  <Script>
    fn get_theme_context(&self) -> ThemeContext {
      ThemeContext {
        dark_mode: self.props.dark_mode,
        primary_color: if self.props.dark_mode { "#61dafb" } else { "#0066cc" },
        background: if self.props.dark_mode { "#121212" } else { "#ffffff" },
        text_color: if self.props.dark_mode { "#ffffff" } else { "#121212" },
      }
    }
  </Script>

  <Template>
    <CreateContext id="theme" value={self.get_theme_context()}>
      {self.props.children}
    </CreateContext>
  </Template>
</Component>

// src/components/themed_button.orbit
<Component>
  <Props>
    label: String
    on_click: Callback<()>
  </Props>

  <Template>
    <UseContext id="theme" as="theme">
      <button 
        class="button"
        style="background-color: {theme.primary_color}; color: {theme.background};"
        on:click={|_| self.props.on_click.emit(())}
      >
        {self.props.label}
      </button>
    </UseContext>
  </Template>

  <Style>
    .button {
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      font-weight: bold;
      cursor: pointer;
    }
  </Style>
</Component>
```

## 🌐 Cross-Platform Examples

### 1. Adaptive Layout

```rust
// src/components/adaptive_dashboard.orbit
<Component>
  <State>
    platform: Platform = Platform::detect()
  </State>

  <Template>
    <div class="dashboard">
      {match self.state.platform {
        Platform::Desktop => (
          <DesktopLayout>
            <DashboardContent />
          </DesktopLayout>
        ),
        Platform::Mobile => (
          <MobileLayout>
            <DashboardContent />
          </MobileLayout>
        ),
        Platform::Web => (
          <ResponsiveLayout>
            <DashboardContent />
          </ResponsiveLayout>
        ),
      }}
    </div>
  </Template>

  <Script>
    fn mount(&mut self) {
      // Subscribe to platform changes
      self.subscribe_to_platform_changes(|platform| {
        self.state.platform = platform;
      });
    }
  </Script>
</Component>
```

### 2. Platform-Specific Features

```rust
// src/components/file_uploader.orbit
<Component>
  <Props>
    on_file_selected: Callback<FileData>
  </Props>

  <Template>
    <div class="uploader">
      {#if Platform::is_web() }
        <input 
          type="file"
          on:change={self.handle_web_file_selection}
        />
      {#else if Platform::is_desktop() }
        <button on:click={self.handle_desktop_file_selection}>
          Select File
        </button>
      {#else}
        <button on:click={self.handle_mobile_file_selection}>
          Choose from Gallery
        </button>
      {/if}
    </div>
  </Template>

  <Script>
    fn handle_web_file_selection(&mut self, e: InputEvent) {
      // Web-specific file handling
      let file = e.files.get(0).unwrap();
      self.props.on_file_selected.emit(file.into());
    }
    
    fn handle_desktop_file_selection(&mut self) {
      // Desktop-specific file dialog
      if let Some(path) = orbit::platform::desktop::open_file_dialog() {
        let file_data = orbit::platform::desktop::read_file(&path).unwrap();
        self.props.on_file_selected.emit(file_data);
      }
    }
    
    fn handle_mobile_file_selection(&mut self) {
      // Mobile-specific image picker
      orbit::platform::mobile::pick_image(|result| {
        if let Some(image) = result {
          self.props.on_file_selected.emit(image.into());
        }
      });
    }
  </Script>
</Component>
```

## 🎨 Advanced UI Examples

### 1. Animations

```rust
// src/components/animated_card.orbit
<Component>
  <Props>
    title: String
    description: String
  </Props>

  <State>
    is_expanded: bool = false
  </State>

  <Template>
    <div 
      class="card"
      class:expanded={self.state.is_expanded}
      on:click={|_| self.toggle_expand()}
    >
      <h3>{self.props.title}</h3>
      
      <Transition name="expand">
        {#if self.state.is_expanded }
          <p>{self.props.description}</p>
        {/if}
      </Transition>
    </div>
  </Template>

  <Style>
    .card {
      padding: 16px;
      border-radius: 8px;
      background-color: #f5f5f5;
      cursor: pointer;
      transition: all 0.3s ease;
    }
    
    .card.expanded {
      background-color: #e0e0e0;
    }
    
    /* Transition styles */
    .expand-enter-active, .expand-leave-active {
      transition: max-height 0.3s ease, opacity 0.3s ease;
      overflow: hidden;
    }
    
    .expand-enter-from, .expand-leave-to {
      max-height: 0;
      opacity: 0;
    }
    
    .expand-enter-to, .expand-leave-from {
      max-height: 200px;
      opacity: 1;
    }
  </Style>

  <Script>
    fn toggle_expand(&mut self) {
      self.state.is_expanded = !self.state.is_expanded;
    }
  </Script>
</Component>
```

### 2. Drag and Drop

```rust
// src/components/drag_drop_list.orbit
<Component>
  <Props>
    items: Vec<String>
    on_reorder: Callback<Vec<String>>
  </Props>

  <State>
    dragging_index: Option<usize> = None
    items_copy: Vec<String> = self.props.items.clone()
  </State>

  <Template>
    <div class="drag-drop-list">
      {self.state.items_copy.iter().enumerate().map(|(index, item)| (
        <div 
          class="item"
          class:dragging={Some(index) == self.state.dragging_index}
          draggable="true"
          on:dragstart={move |_| self.start_drag(index)}
          on:dragover={move |e| self.handle_drag_over(e, index)}
          on:drop={move |_| self.handle_drop(index)}
          on:dragend={|_| self.end_drag()}
        >
          {item}
        </div>
      ))}
    </div>
  </Template>

  <Style>
    .drag-drop-list {
      width: 100%;
    }
    
    .item {
      padding: 12px;
      margin-bottom: 8px;
      background-color: #f5f5f5;
      border-radius: 4px;
      cursor: move;
      transition: transform 0.1s, box-shadow 0.1s;
    }
    
    .item.dragging {
      opacity: 0.5;
      transform: scale(0.98);
    }
  </Style>

  <Script>
    fn start_drag(&mut self, index: usize) {
      self.state.dragging_index = Some(index);
    }
    
    fn handle_drag_over(&mut self, e: DragEvent, _index: usize) {
      e.prevent_default();
    }
    
    fn handle_drop(&mut self, target_index: usize) {
      if let Some(source_index) = self.state.dragging_index {
        let mut new_items = self.state.items_copy.clone();
        let item = new_items.remove(source_index);
        new_items.insert(target_index, item);
        
        self.state.items_copy = new_items.clone();
        self.props.on_reorder.emit(new_items);
      }
    }
    
    fn end_drag(&mut self) {
      self.state.dragging_index = None;
    }
    
    fn update(&mut self) {
      if self.props.items != self.state.items_copy && self.state.dragging_index.is_none() {
        self.state.items_copy = self.props.items.clone();
      }
    }
  </Script>
</Component>
```

---

> Note: These examples represent the expected API and syntax which may evolve as development progresses. Check the repository for the most up-to-date examples.
