# Building Your First Orbit Application

This tutorial will guide you through creating a complete Orbit application from scratch. By the end, you'll have a solid understanding of the Orbit Framework and be able to build your own applications.

## Prerequisites

- Basic knowledge of Rust
- Rust toolchain installed (version 1.70.0 or later)
- Cargo package manager
- Git (optional)

## Project Overview

We'll build a "Task Manager" application with the following features:
- Add, edit, and delete tasks
- Mark tasks as completed
- Filter tasks by status
- Persist tasks to local storage

## Step 1: Installation and Setup

First, let's install the Orbit CLI tool:

```bash
cargo install orbiton
```

Now, create a new Orbit project:

```bash
orbiton new task-manager
cd task-manager
```

This creates a basic project structure:

```
task-manager/
├── Cargo.toml
├── src/
│   ├── main.rs
│   └── components/
│       └── app.orbit
├── static/
│   ├── index.html
│   └── styles.css
└── .gitignore
```

## Step 2: Run the Default Application

Let's make sure everything is working by running the development server:

```bash
orbiton dev
```

This will start a development server at http://localhost:3000. Open your browser to see the default Orbit application.

## Step 3: Create the Task Model

Let's define our task data model. Create a new file `src/models/task.rs`:

```rust
use serde::{Deserialize, Serialize};
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct Task {
    pub id: u64,
    pub title: String,
    pub completed: bool,
    pub created_at: u64,
}

impl Task {
    pub fn new(title: String) -> Self {
        let start = SystemTime::now();
        let timestamp = start
            .duration_since(UNIX_EPOCH)
            .expect("Time went backwards")
            .as_secs();
            
        Self {
            id: timestamp,
            title,
            completed: false,
            created_at: timestamp,
        }
    }
    
    pub fn toggle_completed(&mut self) {
        self.completed = !self.completed;
    }
}

#[derive(Debug, Clone, Copy, PartialEq)]
pub enum TaskFilter {
    All,
    Active,
    Completed,
}

impl TaskFilter {
    pub fn matches(&self, task: &Task) -> bool {
        match self {
            TaskFilter::All => true,
            TaskFilter::Active => !task.completed,
            TaskFilter::Completed => task.completed,
        }
    }
    
    pub fn as_str(&self) -> &'static str {
        match self {
            TaskFilter::All => "All",
            TaskFilter::Active => "Active",
            TaskFilter::Completed => "Completed",
        }
    }
}
```

Now update `src/main.rs` to include this module:

```rust
mod models;
mod components;

use orbit::prelude::*;
use components::app::App;

fn main() {
    // Initialize the Orbit application with the App component
    orbit::run::<App>(orbit::AppConfig::default())
        .expect("Failed to start application");
}
```

And create the `src/models/mod.rs` file:

```rust
pub mod task;
```

## Step 4: Create a Task Input Component

Now, let's create a component for adding new tasks. Create a new file `src/components/task_input.orbit`:

```orbit
<template>
  <div class="task-input-container">
    <input
      type="text"
      class="task-input"
      placeholder="Add a new task..."
      :value="new_task_title"
      @input="handle_input"
      @keydown="handle_keydown"
    />
    <button class="add-task-btn" @click="add_task">Add Task</button>
  </div>
</template>

<style>
.task-input-container {
  display: flex;
  margin-bottom: 20px;
}

.task-input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

.add-task-btn {
  padding: 10px 15px;
  background-color: #0070f3;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
  font-size: 16px;
}

.add-task-btn:hover {
  background-color: #0051b3;
}
</style>

<code lang="rust">
use orbit::prelude::*;

pub struct TaskInput {
    new_task_title: String,
    on_add: Option<Callback<String>>,
}

pub struct TaskInputProps {
    pub on_add: Option<Callback<String>>,
}

impl Props for TaskInputProps {}

impl Component for TaskInput {
    type Props = TaskInputProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            new_task_title: String::new(),
            on_add: props.on_add,
        }
    }
}

impl TaskInput {
    fn handle_input(&mut self, event: InputEvent) {
        self.new_task_title = event.target_value();
    }
    
    fn handle_keydown(&mut self, event: KeyboardEvent) {
        if event.key() == "Enter" && !self.new_task_title.trim().is_empty() {
            self.add_task();
        }
    }
    
    fn add_task(&mut self) {
        let title = self.new_task_title.trim().to_string();
        if !title.is_empty() {
            if let Some(on_add) = &self.on_add {
                on_add.call(title);
                self.new_task_title = String::new();
            }
        }
    }
}
</script>
```

## Step 5: Create a Task Item Component

Next, let's create a component to display individual tasks. Create `src/components/task_item.orbit`:

```orbit
<template>
  <div class="task-item" :class="{ completed: task.completed }">
    <input
      type="checkbox"
      class="task-checkbox"
      :checked="task.completed"
      @change="toggle_completed"
    />
    <span class="task-title">{{ task.title }}</span>
    <button class="delete-btn" @click="delete_task">×</button>
  </div>
</template>

<style>
.task-item {
  display: flex;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid #eee;
}

.task-checkbox {
  margin-right: 10px;
}

.task-title {
  flex: 1;
  font-size: 16px;
}

.completed .task-title {
  text-decoration: line-through;
  color: #999;
}

.delete-btn {
  background: none;
  border: none;
  color: #ff0000;
  font-size: 20px;
  cursor: pointer;
  padding: 0 5px;
}

.delete-btn:hover {
  color: #cc0000;
}
</style>

<code lang="rust">
use orbit::prelude::*;
use crate::models::task::Task;

pub struct TaskItem {
    task: Task,
    on_toggle: Option<Callback<u64>>,
    on_delete: Option<Callback<u64>>,
}

pub struct TaskItemProps {
    pub task: Task,
    pub on_toggle: Option<Callback<u64>>,
    pub on_delete: Option<Callback<u64>>,
}

impl Props for TaskItemProps {}

impl Component for TaskItem {
    type Props = TaskItemProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            task: props.task,
            on_toggle: props.on_toggle,
            on_delete: props.on_delete,
        }
    }
}

impl TaskItem {
    fn toggle_completed(&mut self) {
        if let Some(on_toggle) = &self.on_toggle {
            on_toggle.call(self.task.id);
        }
    }
    
    fn delete_task(&mut self) {
        if let Some(on_delete) = &self.on_delete {
            on_delete.call(self.task.id);
        }
    }
}
</script>
```

## Step 6: Create a Task List Component

Now, let's create a component that lists all tasks. Create `src/components/task_list.orbit`:

```orbit
<template>
  <div class="task-list">
    <div v-if="tasks.is_empty()" class="empty-state">
      No tasks to display
    </div>
    <TaskItem
      v-for="task in filtered_tasks()"
      :key="task.id"
      :task="task.clone()"
      :on-toggle="create_callback(|id| self.handle_toggle(id))"
      :on-delete="create_callback(|id| self.handle_delete(id))"
    />
  </div>
</template>

<style>
.task-list {
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.empty-state {
  padding: 20px;
  text-align: center;
  color: #999;
}
</style>

<code lang="rust">
use orbit::prelude::*;
use crate::models::task::{Task, TaskFilter};
use crate::components::task_item::TaskItem;

pub struct TaskList {
    tasks: Vec<Task>,
    filter: TaskFilter,
    on_toggle: Option<Callback<u64>>,
    on_delete: Option<Callback<u64>>,
}

pub struct TaskListProps {
    pub tasks: Vec<Task>,
    pub filter: TaskFilter,
    pub on_toggle: Option<Callback<u64>>,
    pub on_delete: Option<Callback<u64>>,
}

impl Props for TaskListProps {}

impl Component for TaskList {
    type Props = TaskListProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            tasks: props.tasks,
            filter: props.filter,
            on_toggle: props.on_toggle,
            on_delete: props.on_delete,
        }
    }
}

impl TaskList {
    fn filtered_tasks(&self) -> Vec<&Task> {
        self.tasks
            .iter()
            .filter(|task| self.filter.matches(task))
            .collect()
    }
    
    fn handle_toggle(&mut self, id: u64) {
        if let Some(on_toggle) = &self.on_toggle {
            on_toggle.call(id);
        }
    }
    
    fn handle_delete(&mut self, id: u64) {
        if let Some(on_delete) = &self.on_delete {
            on_delete.call(id);
        }
    }
}
</script>
```

## Step 7: Create a Filter Component

Now, let's create a component for filtering tasks. Create `src/components/task_filter.orbit`:

```orbit
<template>
  <div class="filter-container">
    <span class="filter-label">Show:</span>
    <div class="filter-buttons">
      <button
        v-for="filter_option in [TaskFilter::All, TaskFilter::Active, TaskFilter::Completed]"
        :key="filter_option.as_str()"
        class="filter-btn"
        :class="{ active: filter_option == active_filter }"
        @click="set_filter(filter_option)"
      >
        {{ filter_option.as_str() }}
      </button>
    </div>
  </div>
</template>

<style>
.filter-container {
  display: flex;
  align-items: center;
  margin: 20px 0;
}

.filter-label {
  margin-right: 10px;
  font-weight: bold;
}

.filter-buttons {
  display: flex;
}

.filter-btn {
  background-color: #f5f5f5;
  border: 1px solid #ddd;
  padding: 8px 12px;
  margin-right: 5px;
  cursor: pointer;
}

.filter-btn.active {
  background-color: #0070f3;
  color: white;
  border-color: #0070f3;
}
</style>

<code lang="rust">
use orbit::prelude::*;
use crate::models::task::TaskFilter;

pub struct TaskFilterComponent {
    active_filter: TaskFilter,
    on_filter_change: Option<Callback<TaskFilter>>,
}

pub struct TaskFilterProps {
    pub active_filter: TaskFilter,
    pub on_filter_change: Option<Callback<TaskFilter>>,
}

impl Props for TaskFilterProps {}

impl Component for TaskFilterComponent {
    type Props = TaskFilterProps;
    
    fn new(props: Self::Props) -> Self {
        Self {
            active_filter: props.active_filter,
            on_filter_change: props.on_filter_change,
        }
    }
}

impl TaskFilterComponent {
    fn set_filter(&mut self, filter: TaskFilter) {
        if let Some(on_filter_change) = &self.on_filter_change {
            on_filter_change.call(filter);
        }
    }
}
</script>
```

## Step 8: Update the App Component

Now, let's update the main app component to use our new components. Replace the content of `src/components/app.orbit`:

```orbit
<template>
  <div class="app-container">
    <h1 class="app-title">Orbit Task Manager</h1>
    
    <TaskInput :on-add="create_callback(|title| self.add_task(title))" />
    
    <TaskFilterComponent
      :active-filter="filter"
      :on-filter-change="create_callback(|filter| self.set_filter(filter))"
    />
    
    <TaskList
      :tasks="tasks.clone()"
      :filter="filter"
      :on-toggle="create_callback(|id| self.toggle_task(id))"
      :on-delete="create_callback(|id| self.delete_task(id))"
    />
    
    <div class="task-stats">
      {{ active_count() }} remaining out of {{ tasks.len() }} tasks
    </div>
  </div>
</template>

<style>
.app-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.app-title {
  text-align: center;
  color: #0070f3;
  margin-bottom: 30px;
}

.task-stats {
  margin-top: 20px;
  text-align: right;
  color: #666;
  font-size: 14px;
}
</style>

<code lang="rust">
use orbit::prelude::*;
use crate::models::task::{Task, TaskFilter};
use crate::components::task_input::TaskInput;
use crate::components::task_list::TaskList;
use crate::components::task_filter::TaskFilterComponent;

pub struct App {
    tasks: Vec<Task>,
    filter: TaskFilter,
}

pub struct AppProps {}

impl Props for AppProps {}

impl Default for AppProps {
    fn default() -> Self {
        Self {}
    }
}

impl Component for App {
    type Props = AppProps;
    
    fn new(_props: Self::Props) -> Self {
        Self {
            tasks: Vec::new(),
            filter: TaskFilter::All,
        }
    }
    
    fn mounted(&mut self) {
        // Load tasks from localStorage in real app
        // For this tutorial, we'll add some sample tasks
        self.add_task("Learn Orbit UI Framework".to_string());
        self.add_task("Build a task manager app".to_string());
        self.add_task("Share it with the community".to_string());
    }
}

impl App {
    fn add_task(&mut self, title: String) {
        let task = Task::new(title);
        self.tasks.push(task);
    }
    
    fn toggle_task(&mut self, id: u64) {
        if let Some(task) = self.tasks.iter_mut().find(|t| t.id == id) {
            task.toggle_completed();
        }
    }
    
    fn delete_task(&mut self, id: u64) {
        self.tasks.retain(|task| task.id != id);
    }
    
    fn set_filter(&mut self, filter: TaskFilter) {
        self.filter = filter;
    }
    
    fn active_count(&self) -> usize {
        self.tasks.iter().filter(|task| !task.completed).count()
    }
}
</script>
```

## Step 9: Update the Module Files

Create a `src/components/mod.rs` file:

```rust
pub mod app;
pub mod task_input;
pub mod task_item;
pub mod task_list;
pub mod task_filter;
```

## Step 10: Add Global Styles

Update `static/styles.css` with some base styles:

```css
/* Reset and base styles */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  line-height: 1.6;
  background-color: #f5f7fa;
  color: #333;
}

button, input {
  font-family: inherit;
}

/* Focus styles for accessibility */
:focus {
  outline: 2px solid #0070f3;
  outline-offset: 2px;
}
```

## Step 11: Run the Application

Now, run the application again:

```bash
orbiton dev
```

Open your browser to http://localhost:3000 to see your task manager application.

## Step 12: Add Local Storage Persistence

Let's add persistence so tasks are saved between sessions. Update the `App` component:

```rust
impl Component for App {
    // ... existing code ...
    
    fn mounted(&mut self) {
        // Load tasks from localStorage
        if let Some(stored_tasks) = self.load_tasks() {
            self.tasks = stored_tasks;
        } else {
            // Add sample tasks for new users
            self.add_task("Learn Orbit UI Framework".to_string());
            self.add_task("Build a task manager app".to_string());
            self.add_task("Share it with the community".to_string());
        }
    }
    
    fn updated(&mut self) {
        // Save tasks when they change
        self.save_tasks();
    }
}

impl App {
    // ... existing methods ...
    
    fn save_tasks(&self) {
        if let Ok(json) = serde_json::to_string(&self.tasks) {
            orbit::storage::set_item("tasks", &json);
        }
    }
    
    fn load_tasks(&self) -> Option<Vec<Task>> {
        if let Some(json) = orbit::storage::get_item("tasks") {
            if let Ok(tasks) = serde_json::from_str(&json) {
                return Some(tasks);
            }
        }
        None
    }
}
```

Don't forget to add serde dependencies to your `Cargo.toml`:

```toml
[dependencies]
orbitrs = "0.1.0"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

## Step 13: Build for Production

When you're ready to deploy your application, build it for production:

```bash
orbiton build
```

This creates optimized builds in the `dist/` directory.

## Final Project Structure

Your completed project should have this structure:

```
task-manager/
├── Cargo.toml
├── src/
│   ├── main.rs
│   ├── models/
│   │   ├── mod.rs
│   │   └── task.rs
│   └── components/
│       ├── mod.rs
│       ├── app.orbit
│       ├── task_input.orbit
│       ├── task_item.orbit
│       ├── task_list.orbit
│       └── task_filter.orbit
├── static/
│   ├── index.html
│   └── styles.css
└── .gitignore
```

## Next Steps

Now that you've built your first Orbit application, here are some ways to enhance it:

1. Add task editing functionality
2. Implement task due dates
3. Add task categories or tags
4. Create a task detail view
5. Add animations for improved user experience
6. Implement keyboard shortcuts
7. Add drag-and-drop for reordering tasks

## Conclusion

Congratulations! You've built a functional task manager application with the Orbit UI Framework. This tutorial covered:

- Setting up an Orbit project
- Creating and composing components
- Handling events and state
- Implementing filters and conditional rendering
- Adding persistence with local storage

As you continue learning Orbit, explore the documentation for more advanced features and patterns.

Happy coding!
