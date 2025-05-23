# Multi-File Components Guide

## Introduction

As Orbit applications grow in complexity, organizing components across multiple files becomes essential for maintainability and reusability. This guide covers patterns and best practices for creating multi-file components in the Orbit Framework.

## Table of Contents

- [Overview](#overview)
- [File Structure Patterns](#file-structure-patterns)
- [Component Composition](#component-composition)
- [State Management](#state-management)
- [Shared Types and Interfaces](#shared-types-and-interfaces)
- [Testing Multi-File Components](#testing-multi-file-components)
- [Best Practices](#best-practices)

## Overview

Multi-file components in Orbit allow you to:

- Split large components into manageable, focused files
- Share functionality between related components
- Create hierarchical component structures
- Maintain separation of concerns
- Improve performance through code splitting

## File Structure Patterns

There are several approaches to organizing multi-file components, depending on your component's complexity and your team's preferences.

### Pattern 1: Feature-Based Structure

Organize files by feature, with each feature having its own directory:

```
src/
  features/
    user-profile/
      UserProfile.orbit              # Main component
      UserAvatar.orbit               # Child component
      UserDetails.orbit              # Child component
      UserProfileHeader.orbit        # Child component
      UserProfileActions.orbit       # Child component
      types.rs                       # Shared types
      state.rs                       # State management
      utils.rs                       # Shared utilities
      index.ts                       # Re-exports
```

### Pattern 2: Component Type Structure

Organize by component type, particularly useful for design systems:

```
src/
  components/
    button/
      Button.orbit                   # Base button
      PrimaryButton.orbit            # Variant
      SecondaryButton.orbit          # Variant
      IconButton.orbit               # Variant
      types.rs                       # Shared types
      index.ts                       # Re-exports
```

### Pattern 3: Composition Pattern

Organize components that compose together to form complex UI elements:

```
src/
  components/
    data-table/
      DataTable.orbit                # Main component
      TableHeader.orbit              # Header component
      TableBody.orbit                # Body component
      TableRow.orbit                 # Row component
      TableCell.orbit                # Cell component
      TableFooter.orbit              # Footer component
      table-context.rs               # Shared context
      types.rs                       # Shared types
      index.ts                       # Re-exports
```

## Component Composition

### Importing Components

Import child components into parent components:

```rust
// In UserProfile.orbit
<script>
use orbit::prelude::*;
use crate::features::user_profile::UserAvatar;
use crate::features::user_profile::UserDetails;
use crate::features::user_profile::UserProfileHeader;
use crate::features::user_profile::UserProfileActions;
use crate::features::user_profile::types::UserData;

#[derive(Props)]
pub struct UserProfileProps {
    pub user: UserData,
}

pub struct UserProfile {
    props: UserProfileProps,
}

impl Component for UserProfile {
    type Props = UserProfileProps;
    
    fn new(props: Self::Props) -> Self {
        Self { props }
    }
}
</script>

<template>
  <div class="user-profile">
    <UserProfileHeader user={props.user} />
    <div class="profile-content">
      <UserAvatar src={props.user.avatar_url} size="large" />
      <UserDetails user={props.user} />
    </div>
    <UserProfileActions user={props.user} />
  </div>
</template>
```

### Re-exporting Components

Create an `index.ts` file to re-export components for cleaner imports:

```rust
// In features/user-profile/index.ts
pub use crate::features::user_profile::UserProfile;
pub use crate::features::user_profile::UserAvatar;
pub use crate::features::user_profile::UserDetails;
pub use crate::features::user_profile::UserProfileHeader;
pub use crate::features::user_profile::UserProfileActions;
pub use crate::features::user_profile::types::UserData;
```

This allows importing from a single path:

```rust
use crate::features::user_profile::{UserProfile, UserData};
```

## State Management

### Shared State Between Components

For components that need to share state, use a dedicated state module:

```rust
// In features/user-profile/state.rs
use orbit::state::*;
use crate::features::user_profile::types::UserData;

pub struct UserProfileState {
    pub user: Option<UserData>,
    pub is_editing: bool,
    pub validation_errors: Vec<String>,
}

impl Default for UserProfileState {
    fn default() -> Self {
        Self {
            user: None,
            is_editing: false,
            validation_errors: vec![],
        }
    }
}

// Action types
pub enum UserProfileAction {
    SetUser(UserData),
    StartEditing,
    FinishEditing,
    AddValidationError(String),
    ClearValidationErrors,
}

// Store creation
pub fn create_user_profile_store() -> Store<UserProfileState, UserProfileAction> {
    Store::new(
        UserProfileState::default(),
        |state: &mut UserProfileState, action: &UserProfileAction| {
            match action {
                UserProfileAction::SetUser(user) => {
                    state.user = Some(user.clone());
                },
                UserProfileAction::StartEditing => {
                    state.is_editing = true;
                },
                UserProfileAction::FinishEditing => {
                    state.is_editing = false;
                },
                UserProfileAction::AddValidationError(error) => {
                    state.validation_errors.push(error.clone());
                },
                UserProfileAction::ClearValidationErrors => {
                    state.validation_errors.clear();
                },
            }
        }
    )
}
```

Then use this state in your components:

```rust
// In UserProfile.orbit
<script>
use orbit::prelude::*;
use crate::features::user_profile::state::{create_user_profile_store, UserProfileAction};

pub struct UserProfile {
    store: Store<UserProfileState, UserProfileAction>,
}

impl Component for UserProfile {
    // ...
    
    fn new(props: Self::Props) -> Self {
        let store = create_user_profile_store();
        store.dispatch(UserProfileAction::SetUser(props.user.clone()));
        
        Self { store }
    }
}
</script>
```

### Using Context for State Sharing

For deeper component hierarchies, use Context to avoid prop drilling:

```rust
// In features/user-profile/context.rs
use orbit::context::*;
use crate::features::user_profile::types::UserData;
use crate::features::user_profile::state::{UserProfileState, UserProfileAction, Store};

pub struct UserProfileContext {
    pub store: Store<UserProfileState, UserProfileAction>,
}

impl Context for UserProfileContext {}

// Create a provider component to wrap child components
pub struct UserProfileProvider {
    props: UserProfileProviderProps,
    context: UserProfileContext,
}

#[derive(Props)]
pub struct UserProfileProviderProps {
    pub user: Option<UserData>,
    #[prop(default)]
    pub children: Option<Children>,
}

impl Component for UserProfileProvider {
    type Props = UserProfileProviderProps;
    
    fn new(props: Self::Props) -> Self {
        let store = create_user_profile_store();
        if let Some(user) = &props.user {
            store.dispatch(UserProfileAction::SetUser(user.clone()));
        }
        
        Self {
            props,
            context: UserProfileContext { store },
        }
    }
}

// Usage in template
<template>
  <context-provider context={context}>
    <slot></slot>
  </context-provider>
</template>
```

Then consume the context in child components:

```rust
// In UserProfileActions.orbit
<script>
use orbit::prelude::*;
use orbit::context::use_context;
use crate::features::user_profile::context::UserProfileContext;

pub struct UserProfileActions {}

impl Component for UserProfileActions {
    // ...
    
    fn mounted(&mut self) {
        // Access the context
        if let Some(context) = use_context::<UserProfileContext>() {
            let state = context.store.get_state();
            // Use the state...
        }
    }
    
    fn handle_save(&mut self) {
        if let Some(context) = use_context::<UserProfileContext>() {
            context.store.dispatch(UserProfileAction::FinishEditing);
            // Handle save...
        }
    }
}
</script>
```

## Shared Types and Interfaces

Create a dedicated types file for sharing data structures between components:

```rust
// In features/user-profile/types.rs
use serde::{Serialize, Deserialize};

#[derive(Clone, Serialize, Deserialize)]
pub struct UserData {
    pub id: String,
    pub name: String,
    pub email: String,
    pub avatar_url: String,
    pub role: UserRole,
    pub preferences: UserPreferences,
}

#[derive(Clone, Serialize, Deserialize)]
pub enum UserRole {
    Admin,
    Editor,
    Viewer,
}

#[derive(Clone, Serialize, Deserialize)]
pub struct UserPreferences {
    pub theme: String,
    pub notifications_enabled: bool,
    pub display_name_format: DisplayNameFormat,
}

#[derive(Clone, Serialize, Deserialize)]
pub enum DisplayNameFormat {
    FirstLast,
    LastFirst,
    FirstOnly,
}

// Shared interfaces/traits
pub trait UserFormatter {
    fn format_display_name(&self, user: &UserData) -> String;
}

// Implementation example
pub struct DefaultUserFormatter;

impl UserFormatter for DefaultUserFormatter {
    fn format_display_name(&self, user: &UserData) -> String {
        match user.preferences.display_name_format {
            DisplayNameFormat::FirstLast => format!("{} {}", user.name.split_whitespace().next().unwrap_or(""), 
                                               user.name.split_whitespace().last().unwrap_or("")),
            DisplayNameFormat::LastFirst => format!("{}, {}", user.name.split_whitespace().last().unwrap_or(""),
                                               user.name.split_whitespace().next().unwrap_or("")),
            DisplayNameFormat::FirstOnly => user.name.split_whitespace().next().unwrap_or("").to_string(),
        }
    }
}
```

## Testing Multi-File Components

Testing multi-file components often requires a combination of unit tests for individual components and integration tests for the composed solution:

### Unit Testing Individual Components

```rust
// In UserAvatar.test.rs
use orbit::test::*;
use crate::features::user_profile::UserAvatar;

#[test]
fn test_user_avatar_renders_with_image() {
    // Arrange
    let props = UserAvatarProps {
        src: "https://example.com/avatar.jpg".to_string(),
        size: "medium".to_string(),
    };
    
    // Act
    let component = render_component::<UserAvatar>(props);
    
    // Assert
    assert!(component.query_selector("img").is_some());
    assert_eq!(component.query_selector("img")?.get_attribute("src"), Some("https://example.com/avatar.jpg"));
    assert!(component.query_selector(".avatar-medium").is_some());
}
```

### Integration Testing Component Composition

```rust
// In UserProfile.test.rs
use orbit::test::*;
use crate::features::user_profile::{UserProfile, UserData};

#[test]
fn test_user_profile_renders_all_subcomponents() {
    // Arrange
    let user = UserData {
        id: "123".to_string(),
        name: "John Doe".to_string(),
        email: "john@example.com".to_string(),
        avatar_url: "https://example.com/avatar.jpg".to_string(),
        role: UserRole::Editor,
        preferences: UserPreferences {
            theme: "light".to_string(),
            notifications_enabled: true,
            display_name_format: DisplayNameFormat::FirstLast,
        },
    };
    
    let props = UserProfileProps { user };
    
    // Act
    let component = render_component::<UserProfile>(props);
    
    // Assert
    assert!(component.query_selector(".user-profile-header").is_some());
    assert!(component.query_selector(".user-avatar").is_some());
    assert!(component.query_selector(".user-details").is_some());
    assert!(component.query_selector(".user-profile-actions").is_some());
}

#[test]
fn test_edit_mode_changes_ui() {
    // Arrange
    let user = /* create test user */;
    let props = UserProfileProps { user };
    
    // Act
    let component = render_component::<UserProfile>(props);
    
    // Initial state
    assert!(component.query_selector(".edit-button").is_some());
    assert!(component.query_selector(".edit-form").is_none());
    
    // Trigger edit mode
    component.query_selector(".edit-button")?.click();
    
    // Assert edit mode
    assert!(component.query_selector(".edit-button").is_none());
    assert!(component.query_selector(".save-button").is_some());
    assert!(component.query_selector(".cancel-button").is_some());
    assert!(component.query_selector(".edit-form").is_some());
}
```

## Best Practices

### 1. Single Responsibility Principle

Each component file should have a single responsibility. If a component file grows too large or handles multiple concerns, it's a sign to split it into multiple files.

### 2. Consistent Naming Conventions

- Use PascalCase for component names (e.g., `UserProfile`)
- Use camelCase for methods and properties
- Use snake_case for Rust module names (e.g., `user_profile`)
- Keep filenames consistent with the component names they export

### 3. Proper Exports

- Export components from an `index.ts` file for cleaner imports
- Only export what's necessary to use the component publicly

### 4. Consistent Directory Structure

Choose a directory structure pattern and be consistent across your application:

```
ComponentName/
  ComponentName.orbit       # Main component
  ComponentName.test.ts     # Tests
  ComponentName.style.ts    # Extracted styles (optional)
  index.ts                  # Exports
  utils.ts                  # Component-specific utilities
  types.ts                  # Type definitions
  state.ts                  # State management
  context.ts                # Context definitions
  SubComponent1.orbit       # Child components
  SubComponent2.orbit
```

### 5. Careful State Management

- Decide whether state should be component-local or shared
- Use Context for deeply nested components to avoid prop drilling
- Consider using a state management library for complex state

### 6. Performance Considerations

- Don't overuse contexts, as they can cause unnecessary re-renders
- Consider code splitting for large component trees
- Implement `should_update` to prevent unnecessary rerenders

### 7. Documentation

Document your component structure and usage:

```rust
/// UserProfile component displays a user's profile information
/// 
/// # Example
/// ```
/// use crate::features::user_profile::{UserProfile, UserData};
/// 
/// let user = UserData { /* ... */ };
/// 
/// <UserProfile user={user} />
/// ```
pub struct UserProfile {
    // ...
}
```

## Conclusion

Multi-file components are essential for building maintainable and scalable Orbit applications. By following consistent patterns and best practices, you can create component libraries that are both powerful and easy to use.

For more information on specific aspects of component development, refer to:

- [Component Model](/docs/core-concepts/component-model.md)
- [State Management](/docs/core-concepts/state-management.md)
- [Advanced Component Patterns](/docs/core-concepts/advanced-component-patterns.md)
- [Testing Strategies](/docs/guides/testing-strategies.md)
