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

<script>
use orbitrs::prelude::*;

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
use orbitrs::prelude::*;
use orbitrs::accessibility::focus;

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
use orbitrs::accessibility::checker;

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
use orbitrs::accessibility::announcer;

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

<script>
// Implementation with keyboard navigation handling
</script>
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
use orbitrs::testing::*;
use orbitrs::accessibility::testing::*;

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

## 🔗 Resources

- [WCAG 2.1 Guidelines](https://www.w3.org/TR/WCAG21/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Orbit Accessibility API Documentation](../api/accessibility.md)

## 🔄 Related Topics

- [Event Handling](../core-concepts/event-handling.md) - Learn about event handling in Orbit
- [Component Model](../core-concepts/component-model.md) - Understand how components work
- [Performance Optimization](./performance-optimization.md) - Optimize your accessible applications
