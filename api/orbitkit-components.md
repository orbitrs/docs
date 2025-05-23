# `orbitkit::components` API Reference

This document provides a comprehensive reference for the component library included in the OrbitKit package.

## Overview

`orbitkit::components` is a collection of pre-built, reusable UI components designed to accelerate application development with the Orbit Framework. These components follow accessibility best practices, support theming, and work across all Orbit-supported platforms.

## Core Components

### Button

A clickable button component with various styles and states.

```rust
pub struct Button {
    // Internal fields
}

#[derive(Properties)]
pub struct ButtonProps {
    /// Button variant: "primary", "secondary", "outlined", "danger", "text"
    pub variant: Option<String>,
    /// Whether the button is disabled
    pub disabled: Option<bool>,
    /// Size of the button: "small", "medium", "large"
    pub size: Option<String>,
    /// Whether the button should fill its container width
    pub full_width: Option<bool>,
    /// Optional icon name to display
    pub icon: Option<String>,
    /// Icon position ("left" or "right")
    pub icon_position: Option<String>,
    /// Callback for click events
    pub on_click: EventEmitter<ClickEvent>,
    /// Children elements
    pub children: Children,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::Button;

#[component]
pub fn MyForm() -> impl View {
    let handle_submit = use_callback(|_| {
        println!("Button clicked!");
    });

    html! {
        <div>
            <Button 
                variant="primary" 
                on:click={handle_submit}
            >
                Submit Form
            </Button>
            
            <Button 
                variant="outlined"
                disabled={true}
            >
                Disabled Button
            </Button>
        </div>
    }
}
```

#### Accessibility

- Uses appropriate ARIA attributes based on variant and state
- Keyboard navigable
- Visual focus indicators
- Screen reader support with proper role and state announcements

### Card

A container component for displaying content in a card format.

```rust
pub struct Card {
    // Internal fields
}

#[derive(Properties)]
pub struct CardProps {
    /// Whether the card has a shadow
    pub elevated: Option<bool>,
    /// Whether the card has a border
    pub bordered: Option<bool>,
    /// Padding level: "none", "small", "medium", "large"
    pub padding: Option<String>,
    /// Optional header content
    pub header: Option<Children>,
    /// Optional footer content
    pub footer: Option<Children>,
    /// Main content
    pub children: Children,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::Card;

#[component]
pub fn ProfileCard() -> impl View {
    html! {
        <Card 
            elevated={true}
            padding="medium"
            header={html! {
                <h2>User Profile</h2>
            }}
            footer={html! {
                <Button variant="text">Edit Profile</Button>
            }}
        >
            <div class="profile-content">
                <Avatar src="user.jpg" />
                <p>John Doe</p>
                <p>Software Engineer</p>
            </div>
        </Card>
    }
}
```

#### Accessibility

- Semantic structure with appropriate heading levels
- Keyboard navigable content
- Proper focus management for interactive elements

### Input

A text input component with various configurations.

```rust
pub struct Input {
    // Internal fields
}

#[derive(Properties)]
pub struct InputProps {
    /// Input type: "text", "password", "email", "number", etc.
    pub input_type: Option<String>,
    /// Input placeholder
    pub placeholder: Option<String>,
    /// Input value
    pub value: Option<String>,
    /// Whether the input is disabled
    pub disabled: Option<bool>,
    /// Whether the input is read-only
    pub read_only: Option<bool>,
    /// Input label
    pub label: Option<String>,
    /// Helper text below the input
    pub helper_text: Option<String>,
    /// Error message (displays in error state)
    pub error: Option<String>,
    /// Input size: "small", "medium", "large"
    pub size: Option<String>,
    /// Whether the input is required
    pub required: Option<bool>,
    /// Maximum character length
    pub max_length: Option<u32>,
    /// Input change callback
    pub on_change: EventEmitter<InputEvent>,
    /// Input focus callback
    pub on_focus: EventEmitter<FocusEvent>,
    /// Input blur callback
    pub on_blur: EventEmitter<BlurEvent>,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::Input;

#[component]
pub fn LoginForm() -> impl View {
    let username = use_state(|| String::new());
    let password = use_state(|| String::new());
    
    let handle_username_change = {
        let username = username.clone();
        use_callback(move |e: InputEvent| {
            username.set(e.value.clone());
        })
    };
    
    let handle_password_change = {
        let password = password.clone();
        use_callback(move |e: InputEvent| {
            password.set(e.value.clone());
        })
    };

    html! {
        <form>
            <Input
                label="Username"
                input_type="text"
                placeholder="Enter your username"
                value={(*username).clone()}
                required={true}
                on:change={handle_username_change}
            />
            
            <Input
                label="Password"
                input_type="password"
                placeholder="Enter your password"
                value={(*password).clone()}
                required={true}
                on:change={handle_password_change}
            />
        </form>
    }
}
```

#### Accessibility

- Properly associated labels using `id` and `for` attributes
- ARIA attributes for error states and validation
- Clear focus states
- Screen reader announcements for validation errors

### Layout

A set of layout components for structuring UI.

#### Stack

```rust
pub struct Stack {
    // Internal fields
}

#[derive(Properties)]
pub struct StackProps {
    /// Direction: "horizontal" or "vertical"
    pub direction: Option<String>,
    /// Spacing between items: "none", "xsmall", "small", "medium", "large", "xlarge"
    pub spacing: Option<String>,
    /// How items align in the cross axis: "start", "center", "end", "stretch"
    pub align: Option<String>,
    /// How items align in the main axis: "start", "center", "end", "space-between", "space-around"
    pub justify: Option<String>,
    /// Whether to wrap items
    pub wrap: Option<bool>,
    /// Child elements
    pub children: Children,
}
```

#### Grid

```rust
pub struct Grid {
    // Internal fields
}

#[derive(Properties)]
pub struct GridProps {
    /// Number of columns
    pub columns: Option<u32>,
    /// Gap between items
    pub gap: Option<String>,
    /// Responsive column configuration
    pub responsive: Option<HashMap<String, u32>>,
    /// Child elements
    pub children: Children,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::layout::{Stack, Grid};

#[component]
pub fn PageLayout() -> impl View {
    html! {
        <Stack direction="vertical" spacing="large">
            <header>
                <h1>My Application</h1>
            </header>
            
            <Grid columns={3} gap="medium">
                <Card>Content 1</Card>
                <Card>Content 2</Card>
                <Card>Content 3</Card>
            </Grid>
            
            <Stack direction="horizontal" spacing="medium" justify="center">
                <Button variant="secondary">Cancel</Button>
                <Button variant="primary">Submit</Button>
            </Stack>
        </Stack>
    }
}
```

### Avatar

A component for displaying user images or initials.

```rust
pub struct Avatar {
    // Internal fields
}

#[derive(Properties)]
pub struct AvatarProps {
    /// Image source URL
    pub src: Option<String>,
    /// User name (for generating initials and accessibility)
    pub name: Option<String>,
    /// Size: "small", "medium", "large", or pixel value
    pub size: Option<String>,
    /// Shape: "circle" or "square"
    pub shape: Option<String>,
    /// Whether to show a status indicator
    pub status: Option<String>,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::Avatar;

#[component]
pub fn UserInfo() -> impl View {
    html! {
        <div class="user-info">
            <Avatar 
                src="https://example.com/user.jpg"
                name="John Doe"
                size="large"
                status="online"
            />
            <h3>John Doe</h3>
        </div>
    }
}
```

### Modal

A dialog component for displaying content that requires user attention.

```rust
pub struct Modal {
    // Internal fields
}

#[derive(Properties)]
pub struct ModalProps {
    /// Whether the modal is visible
    pub visible: bool,
    /// Modal title
    pub title: Option<String>,
    /// Whether to show the close button
    pub closable: Option<bool>,
    /// Width of the modal: "small", "medium", "large", or custom size
    pub width: Option<String>,
    /// Callback when closing the modal
    pub on_close: EventEmitter<()>,
    /// Modal content
    pub children: Children,
    /// Optional footer content
    pub footer: Option<Children>,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::Modal;

#[component]
pub fn App() -> impl View {
    let modal_visible = use_state(|| false);
    
    let open_modal = {
        let modal_visible = modal_visible.clone();
        use_callback(move |_| {
            modal_visible.set(true);
        })
    };
    
    let close_modal = {
        let modal_visible = modal_visible.clone();
        use_callback(move |_| {
            modal_visible.set(false);
        })
    };

    html! {
        <div>
            <Button on:click={open_modal}>Open Modal</Button>
            
            <Modal
                visible={*modal_visible}
                title="Important Information"
                on:close={close_modal}
                footer={html! {
                    <>
                        <Button variant="secondary" on:click={close_modal}>Cancel</Button>
                        <Button variant="primary">Confirm</Button>
                    </>
                }}
            >
                <p>This is the modal content.</p>
            </Modal>
        </div>
    }
}
```

#### Accessibility

- Focus trap within the modal when open
- ARIA attributes for modal role and labeling
- Focus returns to trigger element on close
- Keyboard navigation and Escape key closing
- Screen reader announcements for modal opening/closing

### Tabs

A component for organizing content into tabbed sections.

```rust
pub struct Tabs {
    // Internal fields
}

#[derive(Properties)]
pub struct TabsProps {
    /// Active tab key
    pub active_key: String,
    /// Callback when tab changes
    pub on_change: EventEmitter<String>,
    /// Tab orientation: "horizontal" or "vertical"
    pub orientation: Option<String>,
    /// Tab style: "line", "card", "button"
    pub variant: Option<String>,
    /// Tab items
    pub items: Vec<TabItem>,
}

#[derive(Clone)]
pub struct TabItem {
    /// Unique key for the tab
    pub key: String,
    /// Tab label
    pub label: String,
    /// Tab icon (optional)
    pub icon: Option<String>,
    /// Whether tab is disabled
    pub disabled: Option<bool>,
    /// Tab content
    pub content: Children,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::tabs::{Tabs, TabItem};

#[component]
pub fn ProfileTabs() -> impl View {
    let active_tab = use_state(|| "info".to_string());
    
    let handle_tab_change = {
        let active_tab = active_tab.clone();
        use_callback(move |key: String| {
            active_tab.set(key);
        })
    };
    
    let tabs = vec![
        TabItem {
            key: "info".to_string(),
            label: "Information".to_string(),
            icon: Some("info".to_string()),
            disabled: None,
            content: html! {
                <div>Personal information content</div>
            },
        },
        TabItem {
            key: "settings".to_string(),
            label: "Settings".to_string(),
            icon: Some("settings".to_string()),
            disabled: None,
            content: html! {
                <div>Settings content</div>
            },
        },
    ];

    html! {
        <Tabs
            active_key={(*active_tab).clone()}
            on:change={handle_tab_change}
            variant="card"
            items={tabs}
        />
    }
}
```

#### Accessibility

- ARIA tabs role with proper tab relationships
- Keyboard navigation between tabs
- Focus management
- State announcements for screen readers

### Table

A component for displaying tabular data.

```rust
pub struct Table {
    // Internal fields
}

#[derive(Properties)]
pub struct TableProps {
    /// Table columns configuration
    pub columns: Vec<Column>,
    /// Table data
    pub data: Vec<HashMap<String, Value>>,
    /// Whether the table has borders
    pub bordered: Option<bool>,
    /// Whether rows are striped
    pub striped: Option<bool>,
    /// Whether the table is compact
    pub compact: Option<bool>,
    /// Whether rows are hoverable
    pub hoverable: Option<bool>,
    /// Row selection configuration
    pub selection: Option<SelectionConfig>,
    /// Row click handler
    pub on_row_click: Option<EventEmitter<RowClickEvent>>,
}

#[derive(Clone)]
pub struct Column {
    /// Column key (corresponds to data field)
    pub key: String,
    /// Column title
    pub title: String,
    /// Whether the column is sortable
    pub sortable: Option<bool>,
    /// Column width
    pub width: Option<String>,
    /// Custom renderer for the column
    pub render: Option<Box<dyn Fn(Value, HashMap<String, Value>) -> Node>>,
}
```

#### Usage

```rust
use orbit::prelude::*;
use orbitkit::components::table::{Table, Column};
use serde_json::Value;

#[component]
pub fn UsersTable() -> impl View {
    let columns = vec![
        Column {
            key: "name".to_string(),
            title: "Name".to_string(),
            sortable: Some(true),
            width: None,
            render: None,
        },
        Column {
            key: "email".to_string(),
            title: "Email".to_string(),
            sortable: Some(true),
            width: None,
            render: None,
        },
        Column {
            key: "role".to_string(),
            title: "Role".to_string(),
            sortable: None,
            width: Some("100px".to_string()),
            render: Some(Box::new(|value, _| {
                let role = value.as_str().unwrap_or("User");
                let color = match role {
                    "Admin" => "red",
                    "Moderator" => "blue",
                    _ => "gray",
                };
                
                html! {
                    <span style={format!("color: {}", color)}>{role}</span>
                }
            })),
        },
    ];
    
    let data = vec![
        HashMap::from([
            ("name".to_string(), Value::String("John Doe".to_string())),
            ("email".to_string(), Value::String("john@example.com".to_string())),
            ("role".to_string(), Value::String("Admin".to_string())),
        ]),
        HashMap::from([
            ("name".to_string(), Value::String("Jane Smith".to_string())),
            ("email".to_string(), Value::String("jane@example.com".to_string())),
            ("role".to_string(), Value::String("User".to_string())),
        ]),
    ];

    html! {
        <Table
            columns={columns}
            data={data}
            bordered={true}
            striped={true}
            hoverable={true}
        />
    }
}
```

#### Accessibility

- Proper table semantics with headers and cells
- ARIA attributes for sorting and selection
- Keyboard navigation for interactive tables
- Screen reader announcements for state changes

## Form Components

### Checkbox

```rust
pub struct Checkbox {
    // Internal fields
}

#[derive(Properties)]
pub struct CheckboxProps {
    /// Whether the checkbox is checked
    pub checked: bool,
    /// Checkbox label
    pub label: String,
    /// Whether the checkbox is disabled
    pub disabled: Option<bool>,
    /// Checkbox change callback
    pub on_change: EventEmitter<bool>,
}
```

### RadioGroup

```rust
pub struct RadioGroup {
    // Internal fields
}

#[derive(Properties)]
pub struct RadioGroupProps {
    /// Selected value
    pub value: String,
    /// Group name
    pub name: String,
    /// Options
    pub options: Vec<RadioOption>,
    /// Whether the group is disabled
    pub disabled: Option<bool>,
    /// Direction: "horizontal" or "vertical"
    pub direction: Option<String>,
    /// Change callback
    pub on_change: EventEmitter<String>,
}

pub struct RadioOption {
    /// Option value
    pub value: String,
    /// Option label
    pub label: String,
    /// Whether this option is disabled
    pub disabled: Option<bool>,
}
```

### Select

```rust
pub struct Select {
    // Internal fields
}

#[derive(Properties)]
pub struct SelectProps {
    /// Selected value
    pub value: Option<String>,
    /// Select options
    pub options: Vec<SelectOption>,
    /// Select placeholder
    pub placeholder: Option<String>,
    /// Whether the select is disabled
    pub disabled: Option<bool>,
    /// Select label
    pub label: Option<String>,
    /// Helper text
    pub helper_text: Option<String>,
    /// Error message
    pub error: Option<String>,
    /// Whether multiple selection is allowed
    pub multiple: Option<bool>,
    /// Maximum items to display when multiple
    pub max_tag_count: Option<u32>,
    /// Change callback
    pub on_change: EventEmitter<SelectEvent>,
}

pub struct SelectOption {
    /// Option value
    pub value: String,
    /// Option label
    pub label: String,
    /// Whether this option is disabled
    pub disabled: Option<bool>,
}
```

## Navigation Components

### Menu

```rust
pub struct Menu {
    // Internal fields
}

#[derive(Properties)]
pub struct MenuProps {
    /// Selected key
    pub selected_key: Option<String>,
    /// Menu items
    pub items: Vec<MenuItem>,
    /// Menu mode: "vertical", "horizontal", "inline"
    pub mode: Option<String>,
    /// Whether the menu is collapsed (for vertical mode)
    pub collapsed: Option<bool>,
    /// Theme: "light" or "dark"
    pub theme: Option<String>,
    /// Selection callback
    pub on_select: EventEmitter<String>,
}

pub struct MenuItem {
    /// Item key
    pub key: String,
    /// Item label
    pub label: String,
    /// Item icon
    pub icon: Option<String>,
    /// Whether the item is disabled
    pub disabled: Option<bool>,
    /// Child items for submenu
    pub children: Option<Vec<MenuItem>>,
}
```

### Pagination

```rust
pub struct Pagination {
    // Internal fields
}

#[derive(Properties)]
pub struct PaginationProps {
    /// Current page
    pub current: u32,
    /// Total items
    pub total: u32,
    /// Items per page
    pub page_size: u32,
    /// Available page sizes
    pub page_size_options: Option<Vec<u32>>,
    /// Whether to show the page size changer
    pub show_size_changer: Option<bool>,
    /// Whether to show quick jumper
    pub show_quick_jumper: Option<bool>,
    /// Page change callback
    pub on_change: EventEmitter<PaginationEvent>,
}
```

## Feedback Components

### Alert

```rust
pub struct Alert {
    // Internal fields
}

#[derive(Properties)]
pub struct AlertProps {
    /// Alert type: "info", "success", "warning", "error"
    pub alert_type: String,
    /// Alert message
    pub message: String,
    /// Alert description (optional)
    pub description: Option<String>,
    /// Whether the alert is closable
    pub closable: Option<bool>,
    /// Whether the alert shows an icon
    pub show_icon: Option<bool>,
    /// Alert action component
    pub action: Option<Children>,
    /// Close callback
    pub on_close: Option<EventEmitter<()>>,
}
```

### Progress

```rust
pub struct Progress {
    // Internal fields
}

#[derive(Properties)]
pub struct ProgressProps {
    /// Progress percentage (0-100)
    pub percent: f32,
    /// Progress type: "line", "circle", "dashboard"
    pub progress_type: Option<String>,
    /// Progress size: "small", "default", "large"
    pub size: Option<String>,
    /// Progress status: "normal", "success", "exception"
    pub status: Option<String>,
    /// Whether to show progress text
    pub show_info: Option<bool>,
    /// Progress stroke width
    pub stroke_width: Option<f32>,
    /// Progress color
    pub color: Option<String>,
}
```

## Theming and Customization

OrbitKit components support comprehensive theming using the Orbit theming system. All components can be customized using either:

1. **Global Theme**: Configure theme variables in your application
2. **Component-specific Props**: Override specific properties for individual components
3. **CSS Overrides**: Use CSS to customize component appearance

### Theme Example

```rust
use orbit::prelude::*;
use orbitkit::theme::{Theme, ThemeProvider};

fn custom_theme() -> Theme {
    Theme::default()
        .with_color_primary("#6200ee")
        .with_color_secondary("#03dac6")
        .with_font_family("'Roboto', sans-serif")
        .with_border_radius("4px")
}

#[component]
pub fn App() -> impl View {
    html! {
        <ThemeProvider theme={custom_theme()}>
            <div class="app">
                <h1>Themed Application</h1>
                <Button>Themed Button</Button>
            </div>
        </ThemeProvider>
    }
}
```

## Accessibility Support

All OrbitKit components follow accessibility best practices:

- WCAG 2.1 AA compliant
- ARIA attributes for proper screen reader support
- Keyboard navigation
- Focus management
- Color contrast compliance
- Support for reduced motion preferences
- RTL language support

## Related Documentation

- [Component Model](../core-concepts/component-model.md)
- [Advanced Component Patterns](../core-concepts/advanced-component-patterns.md)
- [Style Scoping](../core-concepts/style-scoping.md)
- [Accessibility Implementation](../guides/accessibility.md)
