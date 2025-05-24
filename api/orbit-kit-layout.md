# orbit::kit::layout API Reference

The `orbit::kit::layout` module provides flexible layout components and utilities for creating responsive, well-structured user interfaces using Flexbox-based layout patterns.

## Overview

The layout system in Orbit provides:
- Flexible container components for organizing content
- CSS Flexbox-inspired layout properties
- Responsive layout patterns
- Grid-based layout utilities
- Spacing and alignment helpers

## Core Components

### Layout

The main layout container component that uses Flexbox for arranging child elements.

```rust
pub struct Layout {
    pub direction: Direction,
    pub align: Alignment,
    pub justify: Justification,
    pub gap: String,
    pub padding: String,
    pub children: Option<String>,
}

pub struct LayoutProps {
    pub direction: Option<Direction>,
    pub align: Option<Alignment>,
    pub justify: Option<Justification>,
    pub gap: Option<String>,
    pub padding: Option<String>,
    pub children: Option<String>,
}
```

**Example:**
```rust
use orbit::prelude::*;
use orbit::kit::layout::*;

#[component]
pub fn HeaderLayout() -> impl View {
    html! {
        <Layout 
            direction={Direction::Row}
            justify={Justification::SpaceBetween}
            align={Alignment::Center}
            padding={"16px"}
        >
            <Logo />
            <Navigation />
            <UserMenu />
        </Layout>
    }
}
```

## Layout Properties

### Direction

Controls the main axis direction of the layout.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Direction {
    #[default]
    Row,    // Horizontal layout (default)
    Column, // Vertical layout
}
```

**Usage:**
```rust
html! {
    <Layout direction={Direction::Column}>
        <Header />
        <Main />
        <Footer />
    </Layout>
}
```

### Alignment

Controls how items are aligned on the cross axis.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Alignment {
    #[default]
    Start,   // Align to start of cross axis
    Center,  // Center on cross axis
    End,     // Align to end of cross axis
    Stretch, // Stretch to fill cross axis
}
```

**Usage:**
```rust
html! {
    <Layout 
        direction={Direction::Row}
        align={Alignment::Center}
    >
        <div>Item 1</div>
        <div>Item 2</div>
    </Layout>
}
```

### Justification

Controls how items are distributed along the main axis.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Justification {
    #[default]
    Start,        // Pack items at start
    Center,       // Pack items at center
    End,          // Pack items at end
    SpaceBetween, // Space between items
    SpaceAround,  // Space around items
    SpaceEvenly,  // Even space between items
}
```

**Usage:**
```rust
html! {
    <Layout 
        direction={Direction::Row}
        justify={Justification::SpaceBetween}
    >
        <Button>Left</Button>
        <Button>Right</Button>
    </Layout>
}
```

## Layout Patterns

### Common Layout Patterns

#### Header-Main-Footer Layout
```rust
use orbit::prelude::*;
use orbit::kit::layout::*;

#[component]
pub fn AppLayout() -> impl View {
    html! {
        <Layout 
            direction={Direction::Column}
            align={Alignment::Stretch}
            class="app-layout"
            style="height: 100vh;"
        >
            <Header class="header" />
            <Layout 
                direction={Direction::Row}
                align={Alignment::Stretch}
                class="main-content"
                style="flex: 1;"
            >
                <Sidebar class="sidebar" />
                <Main class="content" />
            </Layout>
            <Footer class="footer" />
        </Layout>
    }
}
```

#### Centered Content
```rust
#[component]
pub fn CenteredModal() -> impl View {
    html! {
        <Layout 
            direction={Direction::Column}
            justify={Justification::Center}
            align={Alignment::Center}
            class="modal-backdrop"
            style="position: fixed; top: 0; left: 0; width: 100%; height: 100%;"
        >
            <div class="modal-content">
                <h2>Modal Title</h2>
                <p>Modal content goes here.</p>
                <Layout 
                    direction={Direction::Row}
                    justify={Justification::End}
                    gap={"8px"}
                >
                    <Button variant="secondary">Cancel</Button>
                    <Button variant="primary">Confirm</Button>
                </Layout>
            </div>
        </Layout>
    }
}
```

#### Card Grid Layout
```rust
#[component]
pub fn CardGrid() -> impl View {
    html! {
        <Layout 
            direction={Direction::Row}
            justify={Justification::Start}
            align={Alignment::Start}
            gap={"16px"}
            padding={"16px"}
            style="flex-wrap: wrap;"
        >
            {cards.iter().map(|card| html! {
                <Card 
                    style="flex: 0 0 calc(33.333% - 16px);"
                    title={card.title.clone()}
                    content={card.content.clone()}
                />
            }).collect::<Vec<_>>()}
        </Layout>
    }
}
```

## Advanced Layout Components

### Grid

A CSS Grid-based layout component for complex layouts.

```rust
pub struct Grid {
    pub columns: String,
    pub rows: String,
    pub gap: String,
    pub areas: Option<String>,
}

pub struct GridProps {
    pub columns: Option<String>,
    pub rows: Option<String>,
    pub gap: Option<String>,
    pub areas: Option<String>,
}
```

**Example:**
```rust
#[component]
pub fn DashboardGrid() -> impl View {
    html! {
        <Grid 
            columns={"1fr 2fr 1fr"}
            rows={"auto 1fr auto"}
            gap={"16px"}
            areas={r#"
                "header header header"
                "sidebar main aside"
                "footer footer footer"
            "#}
        >
            <Header style="grid-area: header;" />
            <Sidebar style="grid-area: sidebar;" />
            <Main style="grid-area: main;" />
            <Aside style="grid-area: aside;" />
            <Footer style="grid-area: footer;" />
        </Grid>
    }
}
```

### Stack

A layout component for stacking elements with consistent spacing.

```rust
pub struct Stack {
    pub direction: Direction,
    pub spacing: String,
    pub align: Alignment,
    pub divider: Option<Element>,
}

pub struct StackProps {
    pub direction: Option<Direction>,
    pub spacing: Option<String>,
    pub align: Option<Alignment>,
    pub divider: Option<Element>,
}
```

**Example:**
```rust
#[component]
pub fn NavigationStack() -> impl View {
    html! {
        <Stack 
            direction={Direction::Column}
            spacing={"8px"}
            divider={html! { <Divider /> }}
        >
            <NavItem href="/dashboard">Dashboard</NavItem>
            <NavItem href="/projects">Projects</NavItem>
            <NavItem href="/settings">Settings</NavItem>
        </Stack>
    }
}
```

### Wrap

A layout component that wraps items to new lines when needed.

```rust
pub struct Wrap {
    pub direction: Direction,
    pub spacing: String,
    pub align: Alignment,
    pub justify: Justification,
}

pub struct WrapProps {
    pub direction: Option<Direction>,
    pub spacing: Option<String>,
    pub align: Option<Alignment>,
    pub justify: Option<Justification>,
}
```

**Example:**
```rust
#[component]
pub fn TagList() -> impl View {
    html! {
        <Wrap 
            spacing={"8px"}
            align={Alignment::Center}
        >
            {tags.iter().map(|tag| html! {
                <Tag label={tag.clone()} />
            }).collect::<Vec<_>>()}
        </Wrap>
    }
}
```

## Responsive Layout

### Breakpoints

Define responsive breakpoints for different screen sizes.

```rust
pub struct Breakpoints {
    pub xs: String,   // Extra small devices
    pub sm: String,   // Small devices
    pub md: String,   // Medium devices
    pub lg: String,   // Large devices
    pub xl: String,   // Extra large devices
}

impl Default for Breakpoints {
    fn default() -> Self {
        Self {
            xs: "0px".to_string(),
            sm: "576px".to_string(),
            md: "768px".to_string(),
            lg: "992px".to_string(),
            xl: "1200px".to_string(),
        }
    }
}
```

### Responsive Props

Create responsive layouts that adapt to different screen sizes.

```rust
#[derive(Clone)]
pub struct ResponsiveDirection {
    pub xs: Option<Direction>,
    pub sm: Option<Direction>,
    pub md: Option<Direction>,
    pub lg: Option<Direction>,
    pub xl: Option<Direction>,
}

#[component]
pub fn ResponsiveLayout() -> impl View {
    html! {
        <Layout 
            direction={ResponsiveDirection {
                xs: Some(Direction::Column),
                md: Some(Direction::Row),
                ..Default::default()
            }}
            gap={"16px"}
        >
            <Sidebar />
            <Main />
        </Layout>
    }
}
```

## Layout Utilities

### Spacing System

Consistent spacing values for gaps, padding, and margins.

```rust
pub struct Spacing;

impl Spacing {
    pub const XS: &'static str = "4px";
    pub const SM: &'static str = "8px";
    pub const MD: &'static str = "16px";
    pub const LG: &'static str = "24px";
    pub const XL: &'static str = "32px";
    pub const XXL: &'static str = "48px";
}

// Usage
html! {
    <Layout gap={Spacing::MD} padding={Spacing::LG}>
        // Content
    </Layout>
}
```

### Alignment Helpers

Utility functions for common alignment patterns.

```rust
pub fn center_content() -> (Alignment, Justification) {
    (Alignment::Center, Justification::Center)
}

pub fn spread_horizontal() -> (Alignment, Justification) {
    (Alignment::Center, Justification::SpaceBetween)
}

pub fn stack_vertical() -> (Direction, Alignment) {
    (Direction::Column, Alignment::Stretch)
}

// Usage
#[component]
pub fn QuickLayout() -> impl View {
    let (align, justify) = center_content();
    
    html! {
        <Layout 
            align={align}
            justify={justify}
        >
            <div>Centered content</div>
        </Layout>
    }
}
```

## Integration with Components

### Layout-Aware Components

Components can be designed to work well within layout containers.

```rust
#[derive(Props)]
pub struct FlexItemProps {
    pub flex: Option<String>,
    pub order: Option<i32>,
    pub align_self: Option<Alignment>,
    pub children: Children,
}

#[component]
pub fn FlexItem(props: FlexItemProps) -> impl View {
    let style = format!(
        "flex: {}; order: {}; align-self: {};",
        props.flex.unwrap_or("1".to_string()),
        props.order.unwrap_or(0),
        match props.align_self {
            Some(Alignment::Start) => "flex-start",
            Some(Alignment::Center) => "center",
            Some(Alignment::End) => "flex-end",
            Some(Alignment::Stretch) => "stretch",
            None => "auto",
        }
    );
    
    html! {
        <div style={style}>
            {props.children}
        </div>
    }
}

// Usage
#[component]
pub fn FlexExample() -> impl View {
    html! {
        <Layout direction={Direction::Row}>
            <FlexItem flex={"1"}>
                <p>Takes available space</p>
            </FlexItem>
            <FlexItem flex={"0 0 200px"}>
                <p>Fixed width</p>
            </FlexItem>
        </Layout>
    }
}
```

## Performance Considerations

### Layout Optimization

- Use `Layout` components sparingly for complex nested layouts
- Prefer CSS Grid for complex grid layouts over nested Flexbox
- Consider virtual scrolling for large lists within layouts

### Best Practices

1. **Semantic Structure**: Use layout components to enhance semantic structure
2. **Consistent Spacing**: Use the spacing system for consistent visual rhythm
3. **Responsive Design**: Design layouts mobile-first with responsive breakpoints
4. **Accessibility**: Ensure layout changes don't break screen reader navigation

```rust
// Good: Semantic and accessible
#[component]
pub fn ArticleLayout() -> impl View {
    html! {
        <article>
            <Layout direction={Direction::Column} gap={Spacing::LG}>
                <header>
                    <h1>Article Title</h1>
                    <Layout direction={Direction::Row} justify={Justification::SpaceBetween}>
                        <span>By Author</span>
                        <time datetime="2025-01-01">January 1, 2025</time>
                    </Layout>
                </header>
                <main>
                    <p>Article content...</p>
                </main>
                <footer>
                    <Layout direction={Direction::Row} gap={Spacing::SM}>
                        <Button>Share</Button>
                        <Button>Like</Button>
                    </Layout>
                </footer>
            </Layout>
        </article>
    }
}
```

## Testing Layout Components

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use orbit::testing::*;
    
    #[test]
    fn test_layout_direction() {
        let mut renderer = TestRenderer::new();
        
        #[component]
        fn TestLayout() -> impl View {
            html! {
                <Layout direction={Direction::Column} data-testid="layout">
                    <div>Item 1</div>
                    <div>Item 2</div>
                </Layout>
            }
        }
        
        let component = renderer.render(html! { <TestLayout /> });
        let layout = component.find_by_test_id("layout");
        
        assert!(layout.has_style("flex-direction", "column"));
    }
    
    #[test]
    fn test_responsive_layout() {
        let mut renderer = TestRenderer::new();
        renderer.set_viewport(600, 800); // Mobile viewport
        
        // Test mobile layout
        // ...
        
        renderer.set_viewport(1200, 800); // Desktop viewport
        
        // Test desktop layout
        // ...
    }
}
```

## Migration from CSS

If migrating from CSS-based layouts:

```css
/* Old CSS */
.container {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: 16px;
    padding: 24px;
}
```

```rust
// New Orbit Layout
html! {
    <Layout 
        direction={Direction::Row}
        justify={Justification::SpaceBetween}
        align={Alignment::Center}
        gap={"16px"}
        padding={"24px"}
    >
        // Content
    </Layout>
}
```

## See Also

- [Component Reference](./component-reference.md) - Built-in UI components
- [Style Scoping](../core-concepts/style-scoping.md) - Styling layout components
- [Responsive Design](../guides/responsive-design.md) - Creating responsive layouts
- [Accessibility](../guides/accessibility.md) - Accessible layout patterns
