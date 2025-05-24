# Orbiton Library API Reference

This document provides a comprehensive reference for using Orbiton as a library in your Rust projects, enabling programmatic project management and build automation.

## Overview

The Orbiton library (`orbiton` crate) can be used directly in Rust applications to:
- Create and manage Orbit projects programmatically
- Integrate Orbit compilation into build systems
- Automate project scaffolding and component generation
- Build custom tooling around Orbit development workflows

## Installation

Add to your `Cargo.toml`:

```toml
[dependencies]
orbiton = "0.1.0"

# Optional: for async operations
tokio = { version = "1.0", features = ["full"] }
```

## Core API

### Project Management

#### `OrbitProject`

```rust
use orbiton::project::OrbitProject;
use std::path::Path;

pub struct OrbitProject {
    path: PathBuf,
    config: OrbitConfig,
}

impl OrbitProject {
    /// Create a new Orbit project at the specified path
    pub fn new<P: AsRef<Path>>(path: P, template: Option<&str>) -> Result<Self, Error> {
        // Creates project structure with:
        // - Cargo.toml
        // - orbiton.toml
        // - src/ directory
        // - Initial components based on template
    }
    
    /// Load an existing Orbit project
    pub fn load<P: AsRef<Path>>(path: P) -> Result<Self, Error> {
        // Loads project configuration
        // Validates project structure
    }
    
    /// Get project configuration
    pub fn config(&self) -> &OrbitConfig {
        &self.config
    }
    
    /// Build the project with specified options
    pub async fn build(&self, options: BuildOptions) -> Result<BuildResult, Error> {
        // Compiles .orbit files to Rust
        // Runs cargo build
        // Handles platform-specific bundling
    }
    
    /// Start development server
    pub async fn start_dev_server(&self, options: DevOptions) -> Result<DevServer, Error> {
        // Starts HTTP and WebSocket servers
        // Enables hot module replacement
        // Watches for file changes
    }
    
    /// Generate a new component
    pub fn generate_component(&self, name: &str, options: ComponentOptions) -> Result<(), Error> {
        // Creates .orbit file from template
        // Updates mod.rs if needed
        // Generates test files if requested
    }
    
    /// Analyze project with Orlint
    pub fn analyze(&self, options: AnalysisOptions) -> Result<AnalysisReport, Error> {
        // Runs static analysis on .orbit files
        // Returns structured results
    }
}
```

#### `OrbitConfig`

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct OrbitConfig {
    pub project: ProjectConfig,
    pub dev: DevConfig,
    pub build: BuildConfig,
    pub component: ComponentConfig,
    pub analyze: AnalyzeConfig,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ProjectConfig {
    pub name: String,
    pub version: String,
    pub description: Option<String>,
    pub template: String,
    pub renderer: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DevConfig {
    pub port: u16,
    pub host: String,
    pub open: bool,
    pub hot_reload: bool,
    pub proxy: Option<ProxyConfig>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct BuildConfig {
    pub target: Vec<String>,
    pub optimize: String,
    pub minify: bool,
    pub sourcemap: bool,
    pub output_dir: String,
}

impl OrbitConfig {
    /// Load configuration from orbiton.toml
    pub fn load<P: AsRef<Path>>(path: P) -> Result<Self, Error> {
        // Loads and validates TOML configuration
    }
    
    /// Save configuration to orbiton.toml
    pub fn save<P: AsRef<Path>>(&self, path: P) -> Result<(), Error> {
        // Serializes configuration to TOML
    }
    
    /// Merge with command-line options
    pub fn merge_with_cli(&mut self, cli_options: CliOptions) {
        // Overrides config with CLI arguments
    }
}
```

### Build System Integration

#### `BuildOptions`

```rust
#[derive(Debug, Clone)]
pub struct BuildOptions {
    pub target: BuildTarget,
    pub mode: BuildMode,
    pub optimize: OptimizationLevel,
    pub output_dir: Option<PathBuf>,
    pub minify: bool,
    pub sourcemap: bool,
    pub features: Vec<String>,
}

#[derive(Debug, Clone)]
pub enum BuildTarget {
    Web,
    Desktop,
    Mobile,
    Embedded,
}

#[derive(Debug, Clone)]
pub enum BuildMode {
    Debug,
    Release,
    Test,
}

#[derive(Debug, Clone)]
pub enum OptimizationLevel {
    None,
    Balanced,
    Size,
    Speed,
}
```

#### `BuildResult`

```rust
#[derive(Debug)]
pub struct BuildResult {
    pub success: bool,
    pub duration: Duration,
    pub artifacts: Vec<BuildArtifact>,
    pub warnings: Vec<String>,
    pub errors: Vec<String>,
}

#[derive(Debug)]
pub struct BuildArtifact {
    pub path: PathBuf,
    pub size: u64,
    pub artifact_type: ArtifactType,
}

#[derive(Debug)]
pub enum ArtifactType {
    Executable,
    WebBundle,
    Library,
    Asset,
}
```

### Development Server

#### `DevServer`

```rust
use tokio::net::TcpListener;

pub struct DevServer {
    http_listener: TcpListener,
    websocket_listener: TcpListener,
    project: OrbitProject,
    options: DevOptions,
}

impl DevServer {
    /// Start the development server
    pub async fn start(project: OrbitProject, options: DevOptions) -> Result<Self, Error> {
        // Binds to HTTP and WebSocket ports
        // Sets up file watching
        // Initializes hot reload system
    }
    
    /// Run the server until stopped
    pub async fn run(&mut self) -> Result<(), Error> {
        // Main server event loop
        // Handles HTTP requests
        // Manages WebSocket connections
        // Processes file change events
    }
    
    /// Stop the server gracefully
    pub async fn stop(&mut self) -> Result<(), Error> {
        // Closes connections
        // Stops file watchers
        // Cleans up resources
    }
    
    /// Broadcast update to connected clients
    pub async fn broadcast_update(&self, update: HmrUpdate) -> Result<(), Error> {
        // Sends WebSocket message to all clients
        // Triggers hot reload or full page refresh
    }
    
    /// Get server status
    pub fn status(&self) -> ServerStatus {
        // Returns current server state
    }
}

#[derive(Debug)]
pub struct DevOptions {
    pub port: u16,
    pub host: String,
    pub open: bool,
    pub watch_paths: Vec<PathBuf>,
    pub ignore_patterns: Vec<String>,
    pub proxy: Option<ProxyConfig>,
}

#[derive(Debug)]
pub enum HmrUpdate {
    ComponentUpdate { path: PathBuf, content: String },
    StyleUpdate { path: PathBuf, content: String },
    FullReload,
}

#[derive(Debug)]
pub enum ServerStatus {
    Starting,
    Running { http_port: u16, ws_port: u16 },
    Stopping,
    Stopped,
    Error(String),
}
```

### Component Generation

#### `ComponentGenerator`

```rust
pub struct ComponentGenerator {
    templates: HashMap<String, ComponentTemplate>,
    config: ComponentConfig,
}

impl ComponentGenerator {
    /// Create a new component generator
    pub fn new(config: ComponentConfig) -> Self {
        // Loads component templates
        // Sets up naming conventions
    }
    
    /// Generate a component with specified options
    pub fn generate(&self, name: &str, options: ComponentOptions) -> Result<GeneratedComponent, Error> {
        // Creates component from template
        // Applies naming conventions
        // Generates associated files
    }
    
    /// List available templates
    pub fn available_templates(&self) -> Vec<&str> {
        // Returns template names
    }
    
    /// Add custom template
    pub fn add_template(&mut self, name: String, template: ComponentTemplate) {
        // Registers new template
    }
}

#[derive(Debug, Clone)]
pub struct ComponentOptions {
    pub template: String,
    pub path: PathBuf,
    pub props: Vec<PropDefinition>,
    pub generate_test: bool,
    pub generate_story: bool,
    pub generate_types: bool,
}

#[derive(Debug)]
pub struct GeneratedComponent {
    pub files: Vec<GeneratedFile>,
    pub dependencies: Vec<String>,
}

#[derive(Debug)]
pub struct GeneratedFile {
    pub path: PathBuf,
    pub content: String,
    pub file_type: FileType,
}

#[derive(Debug)]
pub enum FileType {
    OrbitComponent,
    RustCode,
    TypeDefinition,
    Test,
    Story,
}
```

### Integration with Orlint

#### `AnalysisRunner`

```rust
use orlint::{Analyzer, Rule, AnalysisResult};

pub struct AnalysisRunner {
    analyzer: Analyzer,
    config: AnalysisConfig,
}

impl AnalysisRunner {
    /// Create a new analysis runner
    pub fn new(config: AnalysisConfig) -> Result<Self, Error> {
        // Initializes Orlint analyzer
        // Loads configuration and rules
    }
    
    /// Run analysis on project
    pub fn analyze_project(&self, project: &OrbitProject) -> Result<AnalysisReport, Error> {
        // Discovers .orbit files
        // Runs static analysis
        // Aggregates results
    }
    
    /// Run analysis on specific files
    pub fn analyze_files(&self, files: &[PathBuf]) -> Result<AnalysisReport, Error> {
        // Analyzes specified files only
    }
    
    /// Watch for changes and re-analyze
    pub async fn watch_and_analyze(&mut self, project: &OrbitProject) -> Result<(), Error> {
        // Sets up file watching
        // Re-runs analysis on changes
        // Emits analysis events
    }
}

#[derive(Debug)]
pub struct AnalysisReport {
    pub summary: AnalysisSummary,
    pub files: Vec<FileAnalysis>,
    pub metrics: AnalysisMetrics,
}

#[derive(Debug)]
pub struct AnalysisSummary {
    pub total_files: usize,
    pub total_issues: usize,
    pub error_count: usize,
    pub warning_count: usize,
    pub info_count: usize,
}
```

## Usage Examples

### Basic Project Creation

```rust
use orbiton::project::OrbitProject;
use std::path::Path;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create a new project
    let project = OrbitProject::new("my-app", Some("app"))?;
    
    // Generate a component
    let component_options = ComponentOptions {
        template: "basic".to_string(),
        path: "src/components".into(),
        props: vec![],
        generate_test: true,
        generate_story: false,
        generate_types: true,
    };
    
    project.generate_component("Button", component_options)?;
    
    // Build the project
    let build_options = BuildOptions {
        target: BuildTarget::Web,
        mode: BuildMode::Release,
        optimize: OptimizationLevel::Balanced,
        output_dir: Some("dist".into()),
        minify: true,
        sourcemap: false,
        features: vec![],
    };
    
    let result = project.build(build_options).await?;
    println!("Build completed in {:?}", result.duration);
    
    Ok(())
}
```

### Custom Build Tool Integration

```rust
use orbiton::project::OrbitProject;
use orbiton::builder::{BuildOptions, BuildMode, BuildTarget};

pub struct CustomBuildTool {
    projects: Vec<OrbitProject>,
}

impl CustomBuildTool {
    pub fn new() -> Self {
        Self { projects: Vec::new() }
    }
    
    pub fn add_project<P: AsRef<Path>>(&mut self, path: P) -> Result<(), Error> {
        let project = OrbitProject::load(path)?;
        self.projects.push(project);
        Ok(())
    }
    
    pub async fn build_all(&self) -> Result<Vec<BuildResult>, Error> {
        let mut results = Vec::new();
        
        for project in &self.projects {
            let options = BuildOptions {
                target: BuildTarget::Web,
                mode: BuildMode::Release,
                optimize: OptimizationLevel::Speed,
                output_dir: None,
                minify: true,
                sourcemap: false,
                features: vec!["production".to_string()],
            };
            
            let result = project.build(options).await?;
            results.push(result);
        }
        
        Ok(results)
    }
}
```

### Development Server with Custom Logic

```rust
use orbiton::dev::DevServer;
use orbiton::project::OrbitProject;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let project = OrbitProject::load(".")?;
    
    let dev_options = DevOptions {
        port: 3000,
        host: "localhost".to_string(),
        open: true,
        watch_paths: vec!["src".into(), "static".into()],
        ignore_patterns: vec!["*.tmp".to_string(), "node_modules".to_string()],
        proxy: None,
    };
    
    let mut server = DevServer::start(project, dev_options).await?;
    
    // Run server with custom handling
    tokio::select! {
        result = server.run() => {
            if let Err(e) = result {
                eprintln!("Server error: {}", e);
            }
        }
        _ = tokio::signal::ctrl_c() => {
            println!("Received Ctrl+C, shutting down...");
            server.stop().await?;
        }
    }
    
    Ok(())
}
```

### Analysis Integration

```rust
use orbiton::analysis::AnalysisRunner;
use orbiton::project::OrbitProject;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let project = OrbitProject::load(".")?;
    
    let analysis_config = AnalysisConfig::load(".orlint.toml")?;
    let mut runner = AnalysisRunner::new(analysis_config)?;
    
    // Run one-time analysis
    let report = runner.analyze_project(&project)?;
    
    println!("Analysis Results:");
    println!("  Files analyzed: {}", report.summary.total_files);
    println!("  Issues found: {}", report.summary.total_issues);
    println!("  Errors: {}", report.summary.error_count);
    println!("  Warnings: {}", report.summary.warning_count);
    
    // Watch for changes and re-analyze
    runner.watch_and_analyze(&project).await?;
    
    Ok(())
}
```

## Error Handling

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum Error {
    #[error("Project error: {0}")]
    Project(String),
    
    #[error("Build error: {0}")]
    Build(String),
    
    #[error("Server error: {0}")]
    Server(String),
    
    #[error("Configuration error: {0}")]
    Config(String),
    
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    
    #[error("TOML parsing error: {0}")]
    Toml(#[from] toml::de::Error),
    
    #[error("Analysis error: {0}")]
    Analysis(String),
}

pub type Result<T> = std::result::Result<T, Error>;
```

## Configuration

The library respects the same configuration files as the CLI:

- `orbiton.toml` - Main project configuration
- `.orlint.toml` - Analysis configuration
- `Cargo.toml` - Rust project configuration

### Environment Variables

- `ORBITON_CONFIG_PATH` - Custom configuration file path
- `ORBITON_LOG_LEVEL` - Logging verbosity
- `ORBITON_CACHE_DIR` - Custom cache directory

## Features

Enable optional features in `Cargo.toml`:

```toml
[dependencies]
orbiton = { version = "0.1.0", features = ["dev-server", "analysis", "wasm"] }
```

Available features:
- `dev-server` - Development server functionality
- `analysis` - Integration with Orlint
- `wasm` - WebAssembly build support
- `desktop` - Desktop application support
- `mobile` - Mobile platform support

## See Also

- [Orbiton CLI Reference](./orbiton-cli.md) - Command-line interface documentation
- [Orlint API Reference](./orlint-api.md) - Static analysis API reference
- [Core Orbit API](./orbit-prelude.md) - Core framework APIs
- [Component Reference](./component-reference.md) - Built-in component documentation
