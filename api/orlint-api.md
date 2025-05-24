# Orlint API Reference

This document provides a comprehensive reference for using Orlint (Orbit Analyzer) as a library in your Rust projects, enabling custom static analysis tools and IDE integrations.

## Overview

Orlint provides a programmatic API for:
- Parsing and analyzing `.orbit` files
- Running built-in and custom lint rules
- Integrating static analysis into build pipelines
- Building custom developer tools and IDE extensions
- Creating automated code quality checks

## Installation

Add to your `Cargo.toml`:

```toml
[dependencies]
orlint = "0.1.0"

# Optional: for async operations
tokio = { version = "1.0", features = ["full"] }

# Optional: for serde support
serde = { version = "1.0", features = ["derive"] }
```

## Core API

### Analyzer

#### `Analyzer`

The main entry point for static analysis operations.

```rust
use orlint::{Analyzer, Config, Rule, AnalysisResult};
use std::path::Path;

pub struct Analyzer {
    config: Config,
    rules: Vec<Box<dyn Rule>>,
}

impl Analyzer {
    /// Create a new analyzer with default configuration
    pub fn new() -> Self {
        Self {
            config: Config::default(),
            rules: Vec::new(),
        }
    }
    
    /// Create analyzer with custom configuration
    pub fn with_config(config: Config) -> Self {
        Self {
            config,
            rules: Vec::new(),
        }
    }
    
    /// Add a built-in rule
    pub fn add_rule<R: Rule + 'static>(&mut self, rule: R) {
        self.rules.push(Box::new(rule));
    }
    
    /// Add multiple rules at once
    pub fn add_rules(&mut self, rules: Vec<Box<dyn Rule>>) {
        self.rules.extend(rules);
    }
    
    /// Load rules from configuration
    pub fn load_rules_from_config(&mut self) -> Result<(), Error> {
        // Loads enabled rules based on configuration
        // Applies rule-specific configuration
    }
    
    /// Analyze a single file
    pub fn analyze_file<P: AsRef<Path>>(&self, path: P) -> Result<FileAnalysis, Error> {
        // Parses .orbit file
        // Runs all enabled rules
        // Returns analysis results
    }
    
    /// Analyze multiple files
    pub fn analyze_files<P: AsRef<Path>>(&self, paths: &[P]) -> Result<Vec<FileAnalysis>, Error> {
        // Analyzes each file independently
        // Aggregates results
    }
    
    /// Analyze entire directory
    pub fn analyze_directory<P: AsRef<Path>>(&self, path: P) -> Result<DirectoryAnalysis, Error> {
        // Discovers .orbit files recursively
        // Analyzes all found files
        // Provides directory-level metrics
    }
    
    /// Analyze with custom filter
    pub fn analyze_with_filter<P, F>(&self, path: P, filter: F) -> Result<DirectoryAnalysis, Error>
    where
        P: AsRef<Path>,
        F: Fn(&Path) -> bool,
    {
        // Analyzes files matching the filter function
    }
}
```

### Configuration

#### `Config`

```rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Config {
    pub rules: RuleConfig,
    pub output: OutputConfig,
    pub performance: PerformanceConfig,
    pub ignore: IgnoreConfig,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct RuleConfig {
    pub enabled: Vec<String>,
    pub disabled: Vec<String>,
    pub rule_settings: HashMap<String, serde_json::Value>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct OutputConfig {
    pub format: OutputFormat,
    pub show_source: bool,
    pub show_line_numbers: bool,
    pub max_issues_per_file: Option<usize>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum OutputFormat {
    Text,
    Json,
    Html,
    Sarif,
    CheckStyle,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PerformanceConfig {
    pub parallel_analysis: bool,
    pub max_threads: Option<usize>,
    pub cache_enabled: bool,
    pub cache_dir: Option<String>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct IgnoreConfig {
    pub patterns: Vec<String>,
    pub directories: Vec<String>,
    pub files: Vec<String>,
}

impl Config {
    /// Load configuration from file
    pub fn load<P: AsRef<Path>>(path: P) -> Result<Self, Error> {
        // Loads from .orlint.toml
    }
    
    /// Create with default settings
    pub fn default() -> Self {
        // Returns sensible defaults
    }
    
    /// Merge with another configuration
    pub fn merge(&mut self, other: Config) {
        // Merges configurations with precedence
    }
    
    /// Validate configuration
    pub fn validate(&self) -> Result<(), Vec<String>> {
        // Checks for invalid settings
        // Returns validation errors
    }
}
```

### Rules

#### `Rule` Trait

```rust
use orlint::ast::{OrbitFile, Node};
use orlint::visitor::Visitor;

pub trait Rule: Send + Sync {
    /// Unique identifier for the rule
    fn name(&self) -> &'static str;
    
    /// Human-readable description
    fn description(&self) -> &'static str;
    
    /// Rule category (e.g., "performance", "accessibility", "best-practices")
    fn category(&self) -> RuleCategory;
    
    /// Default severity level
    fn default_severity(&self) -> Severity;
    
    /// Whether the rule can automatically fix issues
    fn supports_auto_fix(&self) -> bool {
        false
    }
    
    /// Run the rule on a parsed file
    fn check(&self, file: &OrbitFile, context: &mut RuleContext) -> Vec<Issue>;
    
    /// Apply automatic fix if supported
    fn fix(&self, file: &mut OrbitFile, issue: &Issue) -> Result<Fix, Error> {
        Err(Error::AutoFixNotSupported(self.name().to_string()))
    }
    
    /// Configure the rule with settings
    fn configure(&mut self, settings: &serde_json::Value) -> Result<(), Error> {
        Ok(()) // Default: no configuration
    }
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum RuleCategory {
    Correctness,
    Performance,
    Accessibility,
    BestPractices,
    Maintainability,
    Security,
    Compatibility,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord)]
pub enum Severity {
    Error,
    Warning,
    Info,
    Hint,
}
```

#### Built-in Rules

```rust
use orlint::rules::*;

// Accessibility rules
pub struct AccessibilityRule;
pub struct AltTextRule;
pub struct KeyboardNavigationRule;
pub struct ColorContrastRule;

// Performance rules
pub struct LargeComponentRule;
pub struct InlineStylesRule;
pub struct UnusedImportsRule;
pub struct ExpensiveOperationsRule;

// Best practices rules
pub struct NamingConventionRule;
pub struct ComponentStructureRule;
pub struct PropsValidationRule;
pub struct StateManagementRule;

// Security rules
pub struct XssPreventionRule;
pub struct DataValidationRule;
pub struct SecurityHeadersRule;

impl AccessibilityRule {
    pub fn new() -> Self {
        Self
    }
    
    pub fn with_config(config: AccessibilityConfig) -> Self {
        // Configure rule with specific settings
    }
}

// Example rule implementation
impl Rule for AccessibilityRule {
    fn name(&self) -> &'static str {
        "accessibility"
    }
    
    fn description(&self) -> &'static str {
        "Ensures components follow accessibility best practices"
    }
    
    fn category(&self) -> RuleCategory {
        RuleCategory::Accessibility
    }
    
    fn default_severity(&self) -> Severity {
        Severity::Warning
    }
    
    fn check(&self, file: &OrbitFile, context: &mut RuleContext) -> Vec<Issue> {
        let mut issues = Vec::new();
        
        // Check for missing alt text on images
        for element in file.template.elements() {
            if element.tag == "img" && !element.has_attribute("alt") {
                issues.push(Issue {
                    rule: self.name(),
                    severity: Severity::Warning,
                    message: "Image elements should have alt text".to_string(),
                    location: element.location.clone(),
                    suggestion: Some("Add an alt attribute".to_string()),
                    fix: None,
                });
            }
        }
        
        issues
    }
}
```

### Analysis Results

#### `AnalysisResult`

```rust
#[derive(Debug, Clone)]
pub struct FileAnalysis {
    pub path: PathBuf,
    pub issues: Vec<Issue>,
    pub metrics: FileMetrics,
    pub parse_time: Duration,
    pub analysis_time: Duration,
}

#[derive(Debug, Clone)]
pub struct DirectoryAnalysis {
    pub path: PathBuf,
    pub files: Vec<FileAnalysis>,
    pub summary: AnalysisSummary,
    pub total_time: Duration,
}

#[derive(Debug, Clone)]
pub struct Issue {
    pub rule: String,
    pub severity: Severity,
    pub message: String,
    pub location: Location,
    pub suggestion: Option<String>,
    pub fix: Option<Fix>,
}

#[derive(Debug, Clone)]
pub struct Location {
    pub file: PathBuf,
    pub line: usize,
    pub column: usize,
    pub length: Option<usize>,
}

#[derive(Debug, Clone)]
pub struct Fix {
    pub description: String,
    pub edits: Vec<TextEdit>,
}

#[derive(Debug, Clone)]
pub struct TextEdit {
    pub range: Range,
    pub new_text: String,
}

#[derive(Debug, Clone)]
pub struct Range {
    pub start: Position,
    pub end: Position,
}

#[derive(Debug, Clone)]
pub struct Position {
    pub line: usize,
    pub character: usize,
}

#[derive(Debug, Clone)]
pub struct FileMetrics {
    pub lines_of_code: usize,
    pub component_count: usize,
    pub template_lines: usize,
    pub script_lines: usize,
    pub style_lines: usize,
    pub complexity_score: f64,
}

#[derive(Debug, Clone)]
pub struct AnalysisSummary {
    pub total_files: usize,
    pub total_issues: usize,
    pub issues_by_severity: HashMap<Severity, usize>,
    pub issues_by_category: HashMap<RuleCategory, usize>,
    pub top_issues: Vec<(String, usize)>, // rule name, count
}
```

### Custom Rule Development

#### `RuleBuilder`

```rust
pub struct RuleBuilder {
    name: String,
    description: String,
    category: RuleCategory,
    severity: Severity,
    visitor: Option<Box<dyn Visitor>>,
}

impl RuleBuilder {
    pub fn new(name: &str) -> Self {
        Self {
            name: name.to_string(),
            description: String::new(),
            category: RuleCategory::BestPractices,
            severity: Severity::Warning,
            visitor: None,
        }
    }
    
    pub fn description(mut self, desc: &str) -> Self {
        self.description = desc.to_string();
        self
    }
    
    pub fn category(mut self, category: RuleCategory) -> Self {
        self.category = category;
        self
    }
    
    pub fn severity(mut self, severity: Severity) -> Self {
        self.severity = severity;
        self
    }
    
    pub fn visitor<V: Visitor + 'static>(mut self, visitor: V) -> Self {
        self.visitor = Some(Box::new(visitor));
        self
    }
    
    pub fn build(self) -> Box<dyn Rule> {
        Box::new(CustomRule {
            name: self.name,
            description: self.description,
            category: self.category,
            severity: self.severity,
            visitor: self.visitor,
        })
    }
}

// Example custom rule
struct CustomRule {
    name: String,
    description: String,
    category: RuleCategory,
    severity: Severity,
    visitor: Option<Box<dyn Visitor>>,
}

impl Rule for CustomRule {
    fn name(&self) -> &'static str {
        Box::leak(self.name.clone().into_boxed_str())
    }
    
    fn description(&self) -> &'static str {
        Box::leak(self.description.clone().into_boxed_str())
    }
    
    fn category(&self) -> RuleCategory {
        self.category
    }
    
    fn default_severity(&self) -> Severity {
        self.severity
    }
    
    fn check(&self, file: &OrbitFile, context: &mut RuleContext) -> Vec<Issue> {
        if let Some(visitor) = &self.visitor {
            visitor.visit(file, context)
        } else {
            Vec::new()
        }
    }
}
```

### AST and Parsing

#### `OrbitFile`

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct OrbitFile {
    pub path: PathBuf,
    pub template: Template,
    pub style: Style,
    pub script: Script,
    pub metadata: FileMetadata,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Template {
    pub root: Element,
    pub source_map: SourceMap,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Element {
    pub tag: String,
    pub attributes: Vec<Attribute>,
    pub children: Vec<Node>,
    pub location: Location,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum Node {
    Element(Element),
    Text(TextNode),
    Expression(Expression),
    Comment(CommentNode),
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Attribute {
    pub name: String,
    pub value: Option<AttributeValue>,
    pub location: Location,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum AttributeValue {
    Static(String),
    Dynamic(Expression),
    Event(EventHandler),
}

impl OrbitFile {
    /// Parse from source code
    pub fn parse(source: &str) -> Result<Self, ParseError> {
        // Parses .orbit file into AST
    }
    
    /// Parse from file
    pub fn parse_file<P: AsRef<Path>>(path: P) -> Result<Self, ParseError> {
        // Reads file and parses content
    }
    
    /// Get all elements recursively
    pub fn all_elements(&self) -> Vec<&Element> {
        // Traverses AST and returns all elements
    }
    
    /// Find elements by tag name
    pub fn find_elements(&self, tag: &str) -> Vec<&Element> {
        // Finds elements matching the tag name
    }
    
    /// Get source location for position
    pub fn location_at(&self, line: usize, column: usize) -> Option<&Location> {
        // Maps position to AST node location
    }
}
```

### Visitor Pattern

#### `Visitor`

```rust
pub trait Visitor {
    fn visit(&self, file: &OrbitFile, context: &mut RuleContext) -> Vec<Issue>;
    
    fn visit_element(&self, element: &Element, context: &mut RuleContext) -> Vec<Issue> {
        Vec::new()
    }
    
    fn visit_attribute(&self, attribute: &Attribute, context: &mut RuleContext) -> Vec<Issue> {
        Vec::new()
    }
    
    fn visit_expression(&self, expression: &Expression, context: &mut RuleContext) -> Vec<Issue> {
        Vec::new()
    }
    
    fn visit_style(&self, style: &Style, context: &mut RuleContext) -> Vec<Issue> {
        Vec::new()
    }
    
    fn visit_script(&self, script: &Script, context: &mut RuleContext) -> Vec<Issue> {
        Vec::new()
    }
}

// Example visitor implementation
pub struct ComponentNamingVisitor {
    pattern: regex::Regex,
}

impl ComponentNamingVisitor {
    pub fn new(pattern: &str) -> Result<Self, regex::Error> {
        Ok(Self {
            pattern: regex::Regex::new(pattern)?,
        })
    }
}

impl Visitor for ComponentNamingVisitor {
    fn visit(&self, file: &OrbitFile, context: &mut RuleContext) -> Vec<Issue> {
        let mut issues = Vec::new();
        
        // Check if component name matches pattern
        if let Some(component_name) = extract_component_name(&file.script) {
            if !self.pattern.is_match(&component_name) {
                issues.push(Issue {
                    rule: "component-naming",
                    severity: Severity::Warning,
                    message: format!("Component name '{}' doesn't match naming convention", component_name),
                    location: file.script.component_location.clone(),
                    suggestion: Some("Use PascalCase for component names".to_string()),
                    fix: None,
                });
            }
        }
        
        issues
    }
}
```

## Usage Examples

### Basic Analysis

```rust
use orlint::{Analyzer, Config};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create analyzer with default configuration
    let mut analyzer = Analyzer::new();
    
    // Load rules from configuration
    analyzer.load_rules_from_config()?;
    
    // Analyze a single file
    let result = analyzer.analyze_file("src/components/Button.orbit")?;
    
    println!("Found {} issues in {}", result.issues.len(), result.path.display());
    for issue in result.issues {
        println!("  {}: {} at line {}", issue.severity, issue.message, issue.location.line);
    }
    
    Ok(())
}
```

### Custom Configuration

```rust
use orlint::{Analyzer, Config, RuleConfig, OutputConfig, OutputFormat};
use std::collections::HashMap;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create custom configuration
    let config = Config {
        rules: RuleConfig {
            enabled: vec![
                "accessibility".to_string(),
                "performance".to_string(),
                "best-practices".to_string(),
            ],
            disabled: vec!["experimental".to_string()],
            rule_settings: {
                let mut settings = HashMap::new();
                settings.insert(
                    "accessibility".to_string(), 
                    serde_json::json!({
                        "require_alt_text": true,
                        "check_color_contrast": true,
                        "minimum_contrast_ratio": 4.5
                    })
                );
                settings
            },
        },
        output: OutputConfig {
            format: OutputFormat::Json,
            show_source: true,
            show_line_numbers: true,
            max_issues_per_file: Some(50),
        },
        performance: Default::default(),
        ignore: Default::default(),
    };
    
    let analyzer = Analyzer::with_config(config);
    let result = analyzer.analyze_directory("src/")?;
    
    println!("Analyzed {} files", result.files.len());
    println!("Total issues: {}", result.summary.total_issues);
    
    Ok(())
}
```

### Custom Rule Development

```rust
use orlint::{Rule, RuleCategory, Severity, Issue, RuleContext};
use orlint::ast::{OrbitFile, Node};

struct NoInlineStylesRule;

impl Rule for NoInlineStylesRule {
    fn name(&self) -> &'static str {
        "no-inline-styles"
    }
    
    fn description(&self) -> &'static str {
        "Prevents use of inline styles in templates"
    }
    
    fn category(&self) -> RuleCategory {
        RuleCategory::BestPractices
    }
    
    fn default_severity(&self) -> Severity {
        Severity::Warning
    }
    
    fn check(&self, file: &OrbitFile, _context: &mut RuleContext) -> Vec<Issue> {
        let mut issues = Vec::new();
        
        for element in file.all_elements() {
            for attr in &element.attributes {
                if attr.name == "style" {
                    issues.push(Issue {
                        rule: self.name().to_string(),
                        severity: self.default_severity(),
                        message: "Inline styles should be avoided. Use CSS classes instead.".to_string(),
                        location: attr.location.clone(),
                        suggestion: Some("Move styles to the <style> section".to_string()),
                        fix: None,
                    });
                }
            }
        }
        
        issues
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut analyzer = Analyzer::new();
    analyzer.add_rule(NoInlineStylesRule);
    
    let result = analyzer.analyze_file("src/components/MyComponent.orbit")?;
    
    for issue in result.issues {
        println!("{}: {}", issue.severity, issue.message);
    }
    
    Ok(())
}
```

### IDE Integration

```rust
use orlint::{Analyzer, Config, Issue, Location};
use std::path::PathBuf;

pub struct OrbitLanguageServer {
    analyzer: Analyzer,
    open_files: HashMap<PathBuf, String>,
}

impl OrbitLanguageServer {
    pub fn new() -> Self {
        let config = Config::load(".orlint.toml").unwrap_or_default();
        let mut analyzer = Analyzer::with_config(config);
        analyzer.load_rules_from_config().unwrap();
        
        Self {
            analyzer,
            open_files: HashMap::new(),
        }
    }
    
    pub fn did_open(&mut self, path: PathBuf, content: String) {
        self.open_files.insert(path, content);
    }
    
    pub fn did_change(&mut self, path: &PathBuf, content: String) {
        self.open_files.insert(path.clone(), content);
    }
    
    pub fn get_diagnostics(&self, path: &PathBuf) -> Vec<Issue> {
        if let Some(_content) = self.open_files.get(path) {
            self.analyzer.analyze_file(path).map(|result| result.issues).unwrap_or_default()
        } else {
            Vec::new()
        }
    }
    
    pub fn get_quick_fixes(&self, path: &PathBuf, location: &Location) -> Vec<String> {
        let issues = self.get_diagnostics(path);
        issues.into_iter()
            .filter(|issue| issue.location.line == location.line)
            .filter_map(|issue| issue.suggestion)
            .collect()
    }
}
```

### Build System Integration

```rust
use orlint::{Analyzer, Config, Severity};
use std::process;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let analyzer = Analyzer::with_config(Config::load(".orlint.toml")?);
    let result = analyzer.analyze_directory("src/")?;
    
    // Count errors
    let error_count = result.files.iter()
        .flat_map(|file| &file.issues)
        .filter(|issue| issue.severity == Severity::Error)
        .count();
    
    if error_count > 0 {
        eprintln!("Analysis failed with {} errors", error_count);
        process::exit(1);
    }
    
    // Print summary
    println!("Analysis completed successfully");
    println!("Files analyzed: {}", result.summary.total_files);
    println!("Issues found: {}", result.summary.total_issues);
    
    for (severity, count) in &result.summary.issues_by_severity {
        println!("  {:?}: {}", severity, count);
    }
    
    Ok(())
}
```

## Error Handling

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum Error {
    #[error("Parse error: {0}")]
    Parse(#[from] ParseError),
    
    #[error("Configuration error: {0}")]
    Config(String),
    
    #[error("Rule error: {0}")]
    Rule(String),
    
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    
    #[error("Auto-fix not supported for rule: {0}")]
    AutoFixNotSupported(String),
    
    #[error("Invalid rule configuration: {0}")]
    InvalidRuleConfig(String),
}

#[derive(Error, Debug)]
pub enum ParseError {
    #[error("Syntax error at line {line}, column {column}: {message}")]
    Syntax { line: usize, column: usize, message: String },
    
    #[error("Unexpected end of file")]
    UnexpectedEof,
    
    #[error("Invalid template syntax: {0}")]
    InvalidTemplate(String),
    
    #[error("Invalid style syntax: {0}")]
    InvalidStyle(String),
    
    #[error("Invalid script syntax: {0}")]
    InvalidScript(String),
}
```

## Advanced Features

### Incremental Analysis

```rust
use orlint::incremental::{AnalysisCache, CacheEntry};

pub struct IncrementalAnalyzer {
    analyzer: Analyzer,
    cache: AnalysisCache,
}

impl IncrementalAnalyzer {
    pub fn new(config: Config) -> Self {
        Self {
            analyzer: Analyzer::with_config(config),
            cache: AnalysisCache::new(),
        }
    }
    
    pub fn analyze_incremental<P: AsRef<Path>>(&mut self, path: P) -> Result<FileAnalysis, Error> {
        let path = path.as_ref();
        
        // Check if file has changed since last analysis
        if let Some(cached) = self.cache.get(path) {
            if !cached.is_stale() {
                return Ok(cached.result.clone());
            }
        }
        
        // Run fresh analysis
        let result = self.analyzer.analyze_file(path)?;
        
        // Cache the result
        self.cache.insert(path.to_path_buf(), CacheEntry::new(result.clone()));
        
        Ok(result)
    }
}
```

### Rule Registry

```rust
use orlint::registry::{RuleRegistry, RuleInfo};

pub struct RuleRegistry {
    rules: HashMap<String, Box<dyn Rule>>,
    metadata: HashMap<String, RuleInfo>,
}

impl RuleRegistry {
    pub fn new() -> Self {
        let mut registry = Self {
            rules: HashMap::new(),
            metadata: HashMap::new(),
        };
        
        // Register built-in rules
        registry.register_builtin_rules();
        
        registry
    }
    
    pub fn register<R: Rule + 'static>(&mut self, rule: R) {
        let name = rule.name().to_string();
        let info = RuleInfo {
            name: name.clone(),
            description: rule.description().to_string(),
            category: rule.category(),
            default_severity: rule.default_severity(),
            supports_auto_fix: rule.supports_auto_fix(),
        };
        
        self.rules.insert(name.clone(), Box::new(rule));
        self.metadata.insert(name, info);
    }
    
    pub fn get_rule(&self, name: &str) -> Option<&dyn Rule> {
        self.rules.get(name).map(|r| r.as_ref())
    }
    
    pub fn list_rules(&self) -> Vec<&RuleInfo> {
        self.metadata.values().collect()
    }
    
    pub fn rules_by_category(&self, category: RuleCategory) -> Vec<&RuleInfo> {
        self.metadata.values()
            .filter(|info| info.category == category)
            .collect()
    }
}
```

## See Also

- [Orlint CLI Usage Guide](../../orlint/docs/cli-usage.md) - Command-line interface
- [Orlint Configuration Guide](../../orlint/docs/configuration.md) - Configuration options
- [Custom Rules Guide](../../orlint/docs/custom-lint-rules.md) - Creating custom rules
- [Orbiton API Reference](./orbiton-lib-api.md) - Orbiton library API
- [VS Code Integration](../../orlint/docs/vscode-integration.md) - Editor integration
