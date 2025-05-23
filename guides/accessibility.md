# ♿ Accessibility Guide

## 📝 Overview

Building accessible applications is not just a best practice—it's essential for creating inclusive software that can be used by everyone, regardless of their abilities or disabilities. This guide covers how to implement accessibility features in your Orbit applications, ensuring they are usable by people with various disabilities including visual, auditory, motor, and cognitive impairments.

## 🔑 Key Principles

### WCAG Conformance

The Orbit Framework is designed to help you build applications that conform to the Web Content Accessibility Guidelines (WCAG) 2.1 AA level, which covers these four principles:

1. **Perceivable** - Information must be presentable to users in ways they can perceive
2. **Operable** - Interface components must be operable by all users
3. **Understandable** - Information and operation must be understandable
4. **Robust** - Content must be robust enough to work with various assistive technologies

## 🧩 Accessibility Features in Orbit

### Component-Level Accessibility

Orbit's built-in components are designed with accessibility in mind:

```orbit
<!-- Accessible button with proper semantics and keyboard support -->
<Button 
  aria-label="Submit form"
  :disabled="!isValid"
>
  Submit
</Button>

<!-- Form inputs with proper labels and validation -->
<FormField 
  label="Email Address" 
  required 
  error-message="Please enter a valid email"
>
  <Input 
    type="email" 
    name="email"
    :aria-invalid="!isEmailValid"
    :aria-describedby="emailErrorId"
  />
  <ErrorMessage id="emailErrorId" v-if="!isEmailValid">
    Please enter a valid email address
  </ErrorMessage>
</FormField>
```

### Semantic Structure

Orbit encourages proper semantic structure through its component model:

```orbit
<template>
  <main>
    <header>
      <h1>{{ pageTitle }}</h1>
      <Nav />
    </header>
    
    <article>
      <section aria-labelledby="section-1-heading">
        <h2 id="section-1-heading">Section Title</h2>
        <p>Section content...</p>
      </section>
    </article>
    
    <aside aria-label="Related information">
      <!-- Sidebar content -->
    </aside>
    
    <footer>
      <!-- Footer content -->
    </footer>
  </main>
</template>
```

### Keyboard Navigation

Ensure all interactive elements can be used with a keyboard alone:

```orbit
<template>
  <div 
    tabindex="0"
    role="button"
    @click="toggle"
    @keydown="handleKeyDown"
    :aria-expanded="expanded"
  >
    Toggle Panel
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct TogglePanel {
    expanded: State<bool>,
}

impl TogglePanel {
    pub fn new() -> Self {
        Self {
            expanded: State::new(false),
        }
    }
    
    pub fn toggle(&self, _: &ClickEvent) {
        self.expanded.set(!*self.expanded.get());
    }
    
    pub fn handle_key_down(&self, event: &KeydownEvent) {
        // Handle Enter or Space key to toggle
        if event.key() == "Enter" || event.key() == " " {
            event.prevent_default();
            self.toggle(&ClickEvent::default());
        }
    }
}
</script>
```

### Focus Management

Implement proper focus management for interactive components and dynamic content:

```rust
use orbit::prelude::*;
use orbit::accessibility::focus;

#[component]
pub struct Dialog {
    is_open: State<bool>,
    #[ref]
    dialog_ref: NodeRef,
}

impl Dialog {
    pub fn new() -> Self {
        Self {
            is_open: State::new(false),
            dialog_ref: NodeRef::new(),
        }
    }
    
    pub fn open(&self) {
        self.is_open.set(true);
        
        // Focus the dialog when it opens
        // This is typically used in the updated lifecycle hook
        if let Some(element) = self.dialog_ref.get() {
            focus::trap_focus(&element);
        }
    }
    
    pub fn close(&self) {
        self.is_open.set(false);
        focus::release_focus();
    }
}
```

## 🧰 Built-in Accessibility Tools

### Accessibility Checker

Orbit provides built-in tools to check for common accessibility issues:

```rust
// In your development environment
use orbit::accessibility::checker;

// Run a check on your component
fn check_component_accessibility() {
    let issues = checker::check_component::<MyComponent>();
    
    for issue in issues {
        println!("Accessibility issue: {}", issue.description);
        println!("Severity: {}", issue.severity);
        println!("How to fix: {}", issue.recommendation);
        println!("WCAG criterion: {}", issue.wcag_criterion);
        println!();
    }
}
```

### Live Announcements

Use the announcement API to notify screen readers of important updates:

```rust
use orbit::accessibility::announcer;

fn update_search_results(query: &str, result_count: usize) {
    // Update the UI with new results
    
    // Announce the update to screen readers
    announcer::announce(
        format!("Found {} results for search: {}", result_count, query),
        announcer::Priority::Polite
    );
}
```

## 📝 Practical Implementation Guide

### 1. Semantic HTML

Use the appropriate HTML elements for their intended purpose:

```orbit
<!-- Good: Using semantic elements -->
<article>
  <h2>Article Title</h2>
  <p>Article content...</p>
</article>

<!-- Avoid: Using divs for everything -->
<div class="article">
  <div class="title">Article Title</div>
  <div class="content">Article content...</div>
</div>
```

### 2. ARIA Attributes

Use ARIA attributes when needed to enhance accessibility:

```orbit
<template>
  <div role="tablist">
    <button 
      role="tab" 
      :aria-selected="activeTab === 'tab1'"
      aria-controls="panel1"
      id="tab1"
      @click="setActiveTab('tab1')"
    >
      Tab 1
    </button>
    <!-- More tabs... -->
    
    <div 
      role="tabpanel" 
      id="panel1" 
      :hidden="activeTab !== 'tab1'"
      aria-labelledby="tab1"
    >
      Tab 1 content
    </div>
    <!-- More panels... -->
  </div>
</template>
```

### 3. Keyboard Navigation

Ensure all interactive elements are keyboard accessible:

```orbit
<template>
  <div class="dropdown">
    <button 
      @click="toggle"
      @keydown="handleKeyDown"
      aria-haspopup="true"
      :aria-expanded="isOpen"
    >
      Menu
    </button>
    
    <ul v-if="isOpen" role="menu">
      <li role="menuitem" tabindex="-1" @click="selectItem('item1')">Item 1</li>
      <li role="menuitem" tabindex="-1" @click="selectItem('item2')">Item 2</li>
    </ul>
  </div>
</template>

<code lang="rust">
// Implementation with keyboard navigation handling
</code>
```

### 4. Color and Contrast

Ensure sufficient color contrast and don't rely on color alone to convey information:

```orbit
<template>
  <div>
    <!-- Good: Uses both color and an icon -->
    <span class="status-error">
      <Icon name="error" />
      Error: Form submission failed
    </span>
    
    <!-- Avoid: Relies only on color -->
    <span class="status-error">
      Form submission failed
    </span>
  </div>
</template>

<style>
/* Ensure text contrast ratio is at least 4.5:1 for normal text */
.status-error {
  /* WCAG AA compliant color with sufficient contrast */
  color: #d32f2f;
  font-weight: bold;
}
</style>
```

### 5. Form Accessibility

Make forms accessible with proper labels, error messages, and field relationships:

```orbit
<template>
  <form @submit.prevent="submitForm">
    <fieldset>
      <legend>Personal Information</legend>
      
      <div class="form-group">
        <label for="name">Full Name</label>
        <Input id="name" v-model="form.name" required />
      </div>
      
      <div class="form-group">
        <label for="email">Email Address</label>
        <Input 
          id="email" 
          type="email" 
          v-model="form.email" 
          required
          :aria-invalid="errors.email ? true : false"
          :aria-describedby="errors.email ? 'email-error' : undefined"
        />
        <div id="email-error" class="error" v-if="errors.email">
          {{ errors.email }}
        </div>
      </div>
      
      <button type="submit">Submit</button>
    </fieldset>
  </form>
</template>
```

## 🧪 Testing Accessibility

### Automated Testing

Orbit provides integration with accessibility testing tools:

```rust
// In your test suite
use orbit::testing::*;
use orbit::accessibility::testing::*;

#[test]
fn test_component_accessibility() {
    // Render the component for testing
    let component = render_for_testing::<MyComponent>();
    
    // Run accessibility tests
    let results = run_accessibility_checks(&component);
    
    // Assert that there are no violations
    assert!(results.violations.is_empty(), 
        "Accessibility violations found: {:?}", results.violations);
}
```

### Manual Testing

Always complement automated testing with manual checks:

1. **Keyboard Navigation**: Navigate through your application using only the keyboard
2. **Screen Reader Testing**: Use VoiceOver (macOS), NVDA or JAWS (Windows), or Orca (Linux)
3. **High Contrast Mode**: Test in high contrast mode and with color adjustments
4. **Zoom Testing**: Test at different zoom levels (up to 200%)
5. **Device Testing**: Test on various devices and browsers

## 📋 Accessibility Checklist

Use this checklist to ensure your application meets basic accessibility requirements:

- [ ] All interactive elements are keyboard accessible
- [ ] Focus states are clearly visible
- [ ] Proper heading structure (h1-h6) is maintained
- [ ] Images have alternative text
- [ ] Forms have proper labels and error messages
- [ ] Color contrast meets WCAG AA standards
- [ ] Dynamic content changes are announced to screen readers
- [ ] Page has a logical tab order
- [ ] No content flashes more than three times per second
- [ ] Application works with screen readers
- [ ] All functionality works at 200% zoom

## 🔠 Advanced Accessibility Patterns

These advanced patterns build on the foundation described above to create more complex accessible interfaces. For detailed implementation examples, see the [Advanced Component Patterns](../core-concepts/advanced-component-patterns.md#-accessibility-first-component-design) documentation.

### Composite Role Patterns

When building complex components from simpler ones, it's essential to manage ARIA relationships properly. This pattern helps maintain proper semantic relationships between parent and child components in complex UI elements like menus, tabs, and dialogs.

```rust
use orbit::prelude::*;
use orbit::a11y::*;

#[component]
pub struct AccessibleCombobox {
    id: Prop<String>,
    expanded: State<bool>,
    selected_option: State<Option<String>>,
    options: Prop<Vec<String>>,
    label: Prop<String>,
}

impl AccessibleCombobox {
    pub fn new() -> Self {
        Self {
            id: Prop::with_default(generate_unique_id()),
            expanded: State::new(false),
            selected_option: State::new(None),
            options: Prop::new(),
            label: Prop::with_default("Select an option".to_string()),
        }
    }
    
    pub fn input_id(&self) -> String {
        format!("{}-input", self.id.get())
    }
    
    pub fn listbox_id(&self) -> String {
        format!("{}-listbox", self.id.get())
    }
    
    pub fn toggle(&self, _: &ClickEvent) {
        self.expanded.update(|expanded| !expanded);
    }
    
    pub fn select_option(&self, option: String) {
        self.selected_option.set(Some(option));
        self.expanded.set(false);
    }
}

impl Component for AccessibleCombobox {
    fn render(&self) -> Node {
        html! {
            <div class="combobox-container">
                <label id={format!("{}-label", self.id.get())} for={self.input_id()}>
                    {self.label.get().clone()}
                </label>
                
                <input 
                    type="text"
                    id={self.input_id()}
                    role="combobox"
                    aria-autocomplete="list"
                    aria-expanded={self.expanded.get().to_string()}
                    aria-controls={self.listbox_id()}
                    aria-labelledby={format!("{}-label", self.id.get())}
                    :value={self.selected_option.get().clone().unwrap_or_default()}
                    @click="toggle"
                />
                
                {
                    if *self.expanded.get() {
                        html! {
                            <ul 
                                id={self.listbox_id()}
                                role="listbox"
                                aria-labelledby={format!("{}-label", self.id.get())}
                                class="options-list"
                            >
                                {
                                    self.options.get().iter().enumerate().map(|(i, option)| {
                                        let option_clone = option.clone();
                                        let is_selected = self.selected_option.get().as_ref() == Some(option);
                                        
                                        html! {
                                            <li 
                                                role="option"
                                                id={format!("{}-option-{}", self.id.get(), i)}
                                                class={format!("option {}", if is_selected { "selected" } else { "" })}
                                                aria-selected={is_selected.to_string()}
                                                tabindex="0"
                                                @click={move |_| self.select_option(option_clone.clone())}
                                                @keydown={move |e: &KeydownEvent| {
                                                    if e.key() == "Enter" || e.key() == " " {
                                                        e.prevent_default();
                                                        self.select_option(option_clone.clone());
                                                    }
                                                }}
                                            >
                                                {option.clone()}
                                            </li>
                                        }
                                    }).collect::<Vec<_>>()
                                }
                            </ul>
                        }
                    } else {
                        html! { <></> }
                    }
                }
            </div>
        }
    }
}
```

### Focus Management Patterns

Implementing proper focus management is crucial for modal dialogs, expanding panels, and other dynamic UI elements. This pattern ensures keyboard users can navigate your application effectively and aren't trapped in certain UI components.

```rust
use orbit::prelude::*;
use orbit::a11y::focus;

// A reusable focus trap hook for modals and other UI that needs focus containment
pub fn use_focus_trap(root_ref: NodeRef) -> FocusTrapHook {
    let active = State::new(false);
    let previous_focus = State::new(None::<NodeRef>);
    
    FocusTrapHook {
        active,
        activate: Box::new(move || {
            if !*active.get() {
                // Store current focus before trapping
                previous_focus.set(Some(focus::get_active_element()));
                
                // Activate trap
                if let Some(element) = root_ref.get() {
                    focus::trap_focus(&element);
                    
                    // Focus first focusable element
                    focus::focus_first_element(&element);
                    
                    active.set(true);
                }
            }
        }),
        deactivate: Box::new(move || {
            if *active.get() {
                // Release trap
                focus::release_focus();
                
                // Restore previous focus
                if let Some(prev) = &*previous_focus.get() {
                    focus::focus_element(prev);
                }
                
                active.set(false);
            }
        }),
    }
}
```

### Live Region Updates Pattern

Notify screen reader users of dynamic content changes without shifting focus:

```rust
use orbit::prelude::*;
use orbit::a11y::*;

#[component]
pub struct LiveRegionDemo {
    search_query: State<String>,
    results: State<Vec<SearchResult>>,
    is_loading: State<bool>,
    result_count: Computed<usize>,
}

impl LiveRegionDemo {
    pub fn new() -> Self {
        Self {
            search_query: State::new(String::new()),
            results: State::new(Vec::new()),
            is_loading: State::new(false),
            result_count: Computed::new(|self_| {
                self_.results.get().len()
            }),
        }
    }
    
    pub fn update_query(&self, event: &InputEvent) {
        self.search_query.set(event.target_value());
    }
    
    pub async fn perform_search(&self) {
        let query = self.search_query.get().clone();
        if query.is_empty() {
            return;
        }
        
        self.is_loading.set(true);
        
        // Simulate API call
        delay(500).await;
        
        // Update results
        let new_results = fetch_search_results(&query).await;
        self.results.set(new_results);
        self.is_loading.set(false);
    }
    
    pub fn announcement_text(&self) -> String {
        if *self.is_loading.get() {
            "Searching, please wait...".to_string()
        } else {
            format!("Found {} results for '{}'", 
                    self.result_count.get(), 
                    self.search_query.get())
        }
    }
}

impl Component for LiveRegionDemo {
    fn render(&self) -> Node {
        html! {
            <div class="search-container">
                <h2>Search Example with Live Region</h2>
                
                <form @submit.prevent="perform_search">
                    <label for="search-input">Search:</label>
                    <input 
                        id="search-input"
                        type="search"
                        :value={*self.search_query.get()}
                        @input="update_query"
                        aria-controls="search-results"
                    />
                    <button type="submit">Search</button>
                </form>
                
                <!-- Live region for announcing results -->
                <div 
                    class="sr-only" 
                    aria-live="polite"
                    aria-atomic="true"
                >
                    {self.announcement_text()}
                </div>
                
                <div id="search-results" class="search-results" aria-label="Search Results">
                    {
                        if *self.is_loading.get() {
                            html! { <LoadingSpinner aria-label="Loading results" /> }
                        } else if self.results.get().is_empty() && !self.search_query.get().is_empty() {
                            html! { <p>No results found.</p> }
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
```

## 🔄 Progressive Enhancement for Accessibility

Progressive enhancement ensures your application remains functional even when certain features are unavailable or disabled. This is especially important for accessibility, as users may have JavaScript disabled, be using older assistive technology, or have limited browser capabilities.

### Basic HTML First

Start with semantic HTML that works without JavaScript or CSS:

```orbit
<template>
  <form method="post" action="/api/submit">
    <fieldset>
      <legend>Contact Information</legend>
      
      <label for="name">Name:</label>
      <input id="name" name="name" type="text" required />
      
      <label for="email">Email:</label>
      <input id="email" name="email" type="email" required />
      
      <button type="submit">Submit</button>
    </fieldset>
  </form>
</template>
```

### Enhance with JavaScript

Then add JavaScript enhancements that improve the experience but aren't required:

```orbit
<template>
  <form 
    method="post" 
    action="/api/submit"
    @submit.prevent="enhancedSubmit"
  >
    <fieldset>
      <legend>Contact Information</legend>
      
      <label for="name">Name:</label>
      <input 
        id="name" 
        name="name" 
        type="text" 
        required 
        :value={form.name}
        @input="updateName"
      />
      
      <label for="email">Email:</label>
      <input 
        id="email" 
        name="email" 
        type="email" 
        required 
        :value={form.email}
        @input="updateEmail"
      />
      
      <button 
        type="submit"
        :disabled="isSubmitting"
      >
        {isSubmitting ? "Submitting..." : "Submit"}
      </button>
    </fieldset>
  </form>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct EnhancedForm {
    form: State<FormData>,
    is_submitting: State<bool>,
}

impl EnhancedForm {
    pub fn new() -> Self {
        Self {
            form: State::new(FormData {
                name: String::new(),
                email: String::new(),
            }),
            is_submitting: State::new(false),
        }
    }
    
    pub fn update_name(&self, event: &InputEvent) {
        self.form.update(|data| {
            data.name = event.target_value();
        });
    }
    
    pub fn update_email(&self, event: &InputEvent) {
        self.form.update(|data| {
            data.email = event.target_value();
        });
    }
    
    pub async fn enhanced_submit(&self, _: &SubmitEvent) {
        self.is_submitting.set(true);
        
        // If fetch fails, the form will still submit normally via action attribute
        if let Ok(response) = fetch_post("/api/submit", &*self.form.get()).await {
            // Handle response...
        }
        
        self.is_submitting.set(false);
    }
}
</code>
```

## 📱 Responsive Accessibility

Accessibility needs can vary depending on device and screen size. Consider these techniques for responsive accessibility:

### Touch Target Sizing

Make sure interactive elements are large enough for touch interaction:

```orbit
<style>
/* Small screens - increase touch target size */
@media (max-width: 768px) {
  button, .interactive {
    /* Minimum touch target size of 44x44px per WCAG */
    min-height: 44px;
    min-width: 44px;
    /* Add padding for text elements */
    padding: 8px 12px;
  }
  
  /* Increase spacing between interactive elements */
  li, button + button, .form-group {
    margin-bottom: 12px;
  }
}
</style>
```

### Responsive Navigation Patterns

Implement accessible responsive navigation:

```orbit
<template>
  <nav aria-label="Main Navigation">
    <!-- Desktop navigation -->
    <ul class="desktop-nav">
      <li><a href="/">Home</a></li>
      <li><a href="/products">Products</a></li>
      <li><a href="/about">About</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
    
    <!-- Mobile navigation (hamburger menu) -->
    <div class="mobile-nav">
      <button 
        aria-expanded={mobileMenuOpen}
        aria-controls="mobile-menu"
        @click="toggleMobileMenu"
        class="menu-toggle"
        aria-label="Toggle navigation menu"
      >
        <span class="hamburger-icon">≡</span>
      </button>
      
      <div 
        id="mobile-menu" 
        class="mobile-menu" 
        :hidden={!mobileMenuOpen}
      >
        <ul>
          <li><a href="/">Home</a></li>
          <li><a href="/products">Products</a></li>
          <li><a href="/about">About</a></li>
          <li><a href="/contact">Contact</a></li>
        </ul>
      </div>
    </div>
  </nav>
</template>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct ResponsiveNav {
    mobile_menu_open: State<bool>,
}

impl ResponsiveNav {
    pub fn new() -> Self {
        Self {
            mobile_menu_open: State::new(false),
        }
    }
    
    pub fn toggle_mobile_menu(&self, _: &ClickEvent) {
        self.mobile_menu_open.update(|open| !open);
    }
    
    // Ensure ESC key closes the menu
    #[lifecycle(mounted)]
    pub fn setup_key_handler(&self) {
        let mobile_menu_open = self.mobile_menu_open.clone();
        
        document_event_listener("keydown", move |event: KeyboardEvent| {
            if event.key() == "Escape" && *mobile_menu_open.get() {
                mobile_menu_open.set(false);
            }
        });
    }
}
</code>
```

## 🔄 Related Topics

- [Advanced Component Patterns](../core-concepts/advanced-component-patterns.md#-accessibility-first-component-design) - Accessibility-first design patterns
- [Event Handling](../core-concepts/event-handling.md) - Learn about event handling in Orbit
- [Component Model](../core-concepts/component-model.md) - Understand how components work
- [Performance Optimization](./performance-optimization.md) - Optimize your accessible applications
