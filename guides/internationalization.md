# Internationalization (i18n) in Orbit

> **Note**: The internationalization system is planned for future development (targeted for Q4 2026). This guide outlines the planned approach and will be updated once the feature is implemented.

## Overview

Internationalization (i18n) is the process of designing and preparing your application to support multiple languages and regions. This guide explains the planned approach for implementing internationalization in the Orbit UI Framework.

## Planned Features

The Orbit internationalization system will include:

1. **Message Formatting**: A flexible system for formatting strings with variables, pluralization, and other locale-specific rules
2. **Resource Management**: Tools for organizing and loading translation resources
3. **Component Integration**: First-class support for internationalized components 
4. **Runtime Locale Switching**: Dynamic language switching without page reloads
5. **Bidirectional Text Support**: Support for languages that read right-to-left (RTL)
6. **Date, Number, and Currency Formatting**: Locale-aware formatting utilities

## Proposed Implementation

### Message Format

Orbit will use a standard message format similar to ICU MessageFormat, which supports:

- Variable substitution
- Pluralization
- Gender-specific messages
- Selecting messages based on conditions

Example syntax:

```
// Simple message with a variable
"welcome": "Hello, {username}!"

// Message with pluralization
"items": "{count, plural, =0{No items} one{One item} other{{count} items}}"

// Gender-based message
"invitation": "{gender, select, male{He invited you} female{She invited you} other{They invited you}}"
```

### Component Integration

Components will have built-in support for internationalization:

```rust
// Example component with i18n support (planned syntax)
use orbit::i18n::t;

#[component]
fn Greeting(username: String) {
    let greeting = t!("welcome", { "username": username });
    
    <div>
        {greeting}
    </div>
}
```

### Resource Files

Translation resources will be organized in one of several supported formats:

#### JSON Format (Recommended)

```json
{
  "en": {
    "welcome": "Hello, {username}!",
    "items": "{count, plural, =0{No items} one{One item} other{{count} items}}",
    "invitation": "{gender, select, male{He invited you} female{She invited you} other{They invited you}}"
  },
  "fr": {
    "welcome": "Bonjour, {username} !",
    "items": "{count, plural, =0{Aucun élément} one{Un élément} other{{count} éléments}}",
    "invitation": "{gender, select, male{Il vous a invité} female{Elle vous a invité} other{Ils vous ont invité}}"
  }
}
```

#### YAML Format

```yaml
en:
  welcome: "Hello, {username}!"
  items: "{count, plural, =0{No items} one{One item} other{{count} items}}"
  
fr:
  welcome: "Bonjour, {username} !"
  items: "{count, plural, =0{Aucun élément} one{Un élément} other{{count} éléments}}"
```

### Configuration

Project configuration will include internationalization settings:

```json
// orbit.config.json
{
  "i18n": {
    "defaultLocale": "en",
    "supportedLocales": ["en", "fr", "de", "ja"],
    "fallbackLocale": "en",
    "resourcesPath": "./locales",
    "resourceFormat": "json",
    "namespaces": ["common", "errors", "components"]
  }
}
```

## Planned CLI Support

The Orbiton CLI will include tools for managing translations:

```bash
# Extract messages from source code
orbiton i18n extract

# Check for missing translations
orbiton i18n check

# Import translations from external sources
orbiton i18n import --source crowdin --api-key XYZ

# Export translations to external platforms
orbiton i18n export --target crowdin --api-key XYZ
```

## Runtime API (Planned)

The planned runtime API will include:

```rust
// Set the current locale
orbit::i18n::set_locale("fr");

// Get the current locale
let current_locale = orbit::i18n::get_locale();

// Translate a message
let message = orbit::i18n::t("welcome", { "username": "John" });

// Format a date according to the current locale
let formatted_date = orbit::i18n::format_date(date, DateFormatOptions::Medium);

// Format a number according to the current locale
let formatted_number = orbit::i18n::format_number(1000.5, NumberFormatOptions::Currency("USD"));
```

## RTL Support (Planned)

For right-to-left languages, Orbit will provide:

- Automatic layout mirroring for RTL languages
- CSS utilities for RTL-aware styling
- Direction-aware component APIs

Example usage:

```rust
// Example component with RTL support (planned syntax)
#[component]
fn Button(label: String, rtl: Option<bool>) {
    // If rtl is not specified, it will be determined from the current locale
    let is_rtl = rtl.unwrap_or_else(|| orbit::i18n::is_rtl());
    
    <button class={if is_rtl { "button rtl" } else { "button" }}>
        {label}
    </button>
}
```

## Best Practices (Planned)

The following best practices will be recommended once the i18n system is implemented:

1. **Use message keys instead of hard-coded strings**
   ```rust
   // Good
   let message = t!("welcome.message");
   
   // Avoid
   let message = "Welcome to the application";
   ```

2. **Parameterize messages instead of concatenating strings**
   ```rust
   // Good
   let message = t!("items.count", { "count": count });
   
   // Avoid
   let message = "You have " + count.to_string() + " items";
   ```

3. **Organize translations by feature or component**
   ```
   locales/
     en/
       common.json
       user-profile.json
       shopping-cart.json
     fr/
       common.json
       user-profile.json
       shopping-cart.json
   ```

4. **Test with multiple locales**
   - Include testing with non-Latin scripts
   - Test with RTL languages
   - Verify pluralization rules work correctly

## Third-Party Integration (Planned)

Orbit will provide integration with popular translation management systems:

- Crowdin
- Lokalise
- Phrase
- POEditor
- Transifex

## Resources

While waiting for the Orbit i18n system to be implemented, you may want to explore these existing internationalization libraries for Rust:

- [fluent-rs](https://github.com/projectfluent/fluent-rs) - Mozilla's Fluent localization system
- [rust-i18n](https://github.com/longbridgeapp/rust-i18n) - Rust internationalization library
- [gettext-rs](https://github.com/Koka/gettext-rs) - GNU gettext binding for Rust

## Future Roadmap

The internationalization system is planned for implementation in Q4 2026. Following its release, these enhancements are being considered:

1. **Machine Translation Integration**: API integrations with services like Google Translate for automatic draft translations
2. **Translation Memory**: System to reuse previously translated content
3. **Content Negotiation**: Automatic locale detection based on HTTP headers
4. **Advanced Pluralization**: Support for languages with complex plural forms
5. **Locale-Specific Components**: Component variants optimized for specific locales

## Contributing

If you have experience with internationalization systems and would like to contribute to the design and implementation of the Orbit i18n system, please join our discussions in the [Orbit contributing guide](../CONTRIBUTING.md).
