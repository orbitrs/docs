# Orbit Compilation Process

This document provides a comprehensive overview of how Orbit applications are compiled, from parsing `.orbit` files to producing final artifacts. It is intended for contributors who need to understand the framework internals and participate in compiler development.

## Overview

The Orbit compilation process transforms single-file components (`.orbit` files) and Rust source code into optimized applications for multiple platforms. The compilation pipeline consists of several coordinated phases:

1. **Source Analysis & Discovery**: Project structure scanning and dependency resolution
2. **Parsing & AST Generation**: Multi-stage parsing of `.orbit` files and Rust code
3. **Code Generation**: Template compilation and style processing
4. **Rust Compilation**: Standard Rust compilation with generated code
5. **Platform-Specific Bundling**: Asset processing and platform optimization
6. **Optimization & Output**: Final artifact generation and optimization

## Source Structure

An Orbit project follows this structure:

```
my-orbit-app/
├── Cargo.toml          # Rust dependencies and metadata
├── orbiton.toml        # Orbit-specific configuration
├── src/
│   ├── main.rs         # Application entry point
│   ├── lib.rs          # Library exports (for libraries)
│   ├── components/     # Component modules
│   │   ├── mod.rs
│   │   ├── button.orbit      # Single-file components
│   │   ├── form.orbit
│   │   └── complex/          # Multi-file components
│   │       ├── mod.orbit
│   │       ├── logic.rs
│   │       └── styles.css
│   └── pages/          # Page components (for apps)
│       ├── mod.rs
│       ├── home.orbit
│       └── about.orbit
├── static/             # Static assets
│   ├── images/
│   ├── fonts/
│   └── favicon.ico
└── target/             # Build artifacts
    ├── debug/          # Development builds
    ├── release/        # Production builds
    └── orbit_codegen/  # Generated code cache
```

---

## Phase 1: Source Analysis & Discovery

### Project Discovery

Orbiton begins by scanning the project structure:

1. **Configuration Loading**: Parse `orbiton.toml` and `Cargo.toml`
2. **Source Tree Scanning**: Recursively find all `.orbit` and `.rs` files
3. **Dependency Graph Construction**: Build component dependency tree
4. **Asset Inventory**: Catalog static assets and their relationships

### Dependency Resolution

```rust
// Internal representation of component dependencies
struct ComponentGraph {
    components: HashMap<ComponentId, ComponentNode>,
    dependencies: HashMap<ComponentId, Vec<ComponentId>>,
    imports: HashMap<ComponentId, Vec<ImportStatement>>,
}

struct ComponentNode {
    file_path: PathBuf,
    component_type: ComponentType,
    exports: Vec<Symbol>,
    imports: Vec<ImportStatement>,
}
```

### Configuration Merging

Orbiton merges configuration from multiple sources:

1. **Default Configuration**: Built-in framework defaults
2. **Global Config**: `~/.orbiton/config.toml` (user defaults)
3. **Project Config**: `orbiton.toml` (project-specific settings)
4. **Command Line Args**: Override any setting via CLI flags

---

## Phase 2: Parsing & AST Generation

### Multi-Stage Parsing Pipeline

The parsing process is divided into specialized stages:

#### Stage 1: Lexical Analysis

```rust
// Token types for .orbit files
#[derive(Debug, Clone)]
enum OrbitToken {
    // Section delimiters
    TemplateStart,
    TemplateEnd,
    StyleStart,
    StyleEnd,
    CodeStart,
    CodeEnd,
    
    // Template tokens
    ElementStart(String),
    ElementEnd(String),
    Attribute(String, String),
    TextContent(String),
    Interpolation(String),
    Directive(String, String),
    
    // Style tokens
    Selector(String),
    Declaration(String, String),
    MediaQuery(String),
    
    // Code tokens (delegated to Rust lexer)
    RustCode(Vec<proc_macro2::TokenStream>),
}
```

#### Stage 2: Syntactic Analysis

```rust
// AST nodes for template section
#[derive(Debug, Clone)]
pub enum TemplateNode {
    Element {
        tag: String,
        attributes: Vec<Attribute>,
        children: Vec<TemplateNode>,
        directives: Vec<Directive>,
    },
    Text(String),
    Interpolation {
        expression: String,
        filters: Vec<Filter>,
    },
    Conditional {
        condition: String,
        then_branch: Box<TemplateNode>,
        else_branch: Option<Box<TemplateNode>>,
    },
    Loop {
        iterator: String,
        iterable: String,
        body: Box<TemplateNode>,
    },
    Slot {
        name: Option<String>,
        default_content: Vec<TemplateNode>,
    },
}

// Style AST representation
#[derive(Debug, Clone)]
pub struct StyleAST {
    rules: Vec<CssRule>,
    media_queries: Vec<MediaQuery>,
    imports: Vec<ImportRule>,
    scoped: bool,
}

#[derive(Debug, Clone)]
pub struct CssRule {
    selector: Selector,
    declarations: Vec<Declaration>,
    nested_rules: Vec<CssRule>,
}
```

#### Stage 3: Semantic Analysis

During semantic analysis, the compiler:

1. **Symbol Resolution**: Resolve component references and imports
2. **Type Checking**: Validate prop types and component interfaces
3. **Scope Analysis**: Determine variable scoping for template expressions
4. **Dependency Tracking**: Build fine-grained dependency graph

```rust
// Semantic information attached to AST nodes
#[derive(Debug)]
struct SemanticInfo {
    symbol_table: SymbolTable,
    type_info: TypeInformation,
    scope_chain: ScopeChain,
    diagnostics: Vec<Diagnostic>,
}

struct SymbolTable {
    symbols: HashMap<String, Symbol>,
    imports: Vec<ImportedSymbol>,
    exports: Vec<ExportedSymbol>,
}
```

### Error Recovery & Diagnostics

The parser includes sophisticated error recovery:

```rust
#[derive(Debug, Clone)]
struct ParseError {
    kind: ErrorKind,
    span: Span,
    message: String,
    suggestions: Vec<Suggestion>,
    related_errors: Vec<RelatedError>,
}

#[derive(Debug, Clone)]
enum ErrorKind {
    SyntaxError,
    TypeError,
    UnresolvedSymbol,
    CircularDependency,
    InvalidDirective,
    MalformedExpression,
}
```

---

## Phase 3: Code Generation

### Template Code Generation

Templates are compiled into efficient Rust render functions:

```rust
// Input template:
// <div class="container">
//   <h1>{{ title }}</h1>
//   <button @click="handle_click">{{ button_text }}</button>
// </div>

// Generated Rust code:
fn render(&self) -> Result<Vec<Node>, ComponentError> {
    let mut nodes = Vec::new();
    
    // Container div
    let mut container = Node::new_element("div");
    container.add_class("container");
    container.add_class(&self.get_scoped_class("container"));
    
    // Title h1
    let mut title_element = Node::new_element("h1");
    let title_text = Node::new_text(&self.title);
    title_element.add_child(title_text);
    container.add_child(title_element);
    
    // Button
    let mut button = Node::new_element("button");
    button.add_event_listener("click", Box::new(|event| {
        self.handle_click(event);
    }));
    let button_text = Node::new_text(&self.button_text);
    button.add_child(button_text);
    container.add_child(button);
    
    nodes.push(container);
    Ok(nodes)
}
```

### Advanced Template Features

#### Conditional Rendering

```rust
// Template: <div v-if="show_content">Content</div>
// Generated:
if self.show_content.get() {
    let mut div = Node::new_element("div");
    div.add_child(Node::new_text("Content"));
    nodes.push(div);
}
```

#### List Rendering

```rust
// Template: <li v-for="item in items">{{ item.name }}</li>
// Generated:
for item in self.items.get().iter() {
    let mut li = Node::new_element("li");
    li.add_child(Node::new_text(&item.name));
    nodes.push(li);
}
```

#### Component Composition

```rust
// Template: <Button :disabled="loading" @click="submit">Submit</Button>
// Generated:
let button_props = ButtonProps {
    disabled: self.loading.get().clone(),
    label: "Submit".to_string(),
};
let button_component = Button::create(button_props, self.context.clone());
let button_node = Node::new_component(Box::new(button_component));
button_node.add_event_listener("click", Box::new(|event| {
    self.submit(event);
}));
nodes.push(button_node);
```

### Style Processing

#### Scoped CSS Generation

```rust
// Input CSS:
.container {
    padding: 20px;
    border: 1px solid #ccc;
}

.title {
    color: #333;
    font-size: 1.5em;
}

// Generated scoped CSS:
.container__scope_abc123 {
    padding: 20px;
    border: 1px solid #ccc;
}

.title__scope_abc123 {
    color: #333;
    font-size: 1.5em;
}

// Generated Rust helper:
impl MyComponent {
    fn get_scoped_class(&self, class: &str) -> String {
        format!("{}__scope_abc123", class)
    }
}
```

#### CSS Optimization

The style processor includes several optimization passes:

1. **Dead Code Elimination**: Remove unused CSS rules
2. **Property Merging**: Combine duplicate declarations
3. **Selector Optimization**: Simplify complex selectors
4. **Vendor Prefix Management**: Add/remove prefixes as needed
5. **Critical CSS Extraction**: Separate above-the-fold styles

### Build-Time Optimizations

#### Tree Shaking

Orbiton performs aggressive tree shaking:

```rust
// Component dependency analysis
struct ComponentUsageAnalysis {
    used_components: HashSet<ComponentId>,
    used_props: HashMap<ComponentId, HashSet<PropName>>,
    used_methods: HashMap<ComponentId, HashSet<MethodName>>,
    reachable_code: HashSet<CodeSpan>,
}
```

#### Bundle Splitting

For web targets, code is automatically split:

```rust
enum BundleType {
    Main,           // Core application code
    Vendor,         // Third-party dependencies
    Component(String), // Lazy-loaded components
    Shared,         // Common utilities
}
```

---

## Phase 4: Rust Compilation

### Build Script Integration

Orbiton uses a sophisticated build script to coordinate compilation:

```rust
// build.rs
fn main() {
    // 1. Check for .orbit file changes
    let orbit_files = discover_orbit_files("src/");
    
    // 2. Run code generation if needed
    if needs_regeneration(&orbit_files) {
        run_orbiton_codegen();
    }
    
    // 3. Set up cargo rerun triggers
    for file in &orbit_files {
        println!("cargo:rerun-if-changed={}", file.display());
    }
    
    // 4. Configure target-specific features
    configure_target_features();
}
```

### Generated Code Structure

Generated code is organized systematically:

```
target/orbit_codegen/
├── components/
│   ├── mod.rs              # Component registry
│   ├── button.rs           # Generated Button impl
│   ├── form.rs             # Generated Form impl
│   └── metadata.rs         # Component metadata
├── styles/
│   ├── scoped.css          # Scoped component styles
│   ├── global.css          # Global styles
│   └── critical.css        # Critical path CSS
├── assets/
│   ├── manifest.json       # Asset manifest
│   └── chunks/             # Code-split chunks
└── build_info.rs           # Build metadata
```

### Incremental Compilation

Orbiton supports incremental compilation:

```rust
#[derive(Serialize, Deserialize)]
struct CompilationCache {
    file_hashes: HashMap<PathBuf, u64>,
    dependency_graph: DependencyGraph,
    generated_artifacts: HashMap<PathBuf, PathBuf>,
    last_build_time: SystemTime,
}
```

---

## Phase 5: Platform-Specific Bundling

### Web Target Processing

For web deployment (`--target web`):

#### WASM Generation & Optimization

```bash
# Internal pipeline for web builds
# Step 1: Build for WASM target with web-specific features
cargo build --target wasm32-unknown-unknown --no-default-features --features="web" --release

# Step 2: Generate JavaScript bindings
wasm-bindgen target/wasm32-unknown-unknown/release/app.wasm --out-dir pkg/ --target web

# Step 3: Optimize WASM binary (optional)
wasm-opt pkg/app_bg.wasm -O4 -o pkg/app_bg.wasm
```

**Feature Configuration for WASM Builds:**

The Orbit framework uses conditional compilation to ensure only web-compatible dependencies are included in WASM builds:

```toml
# Key features for WASM builds
[features]
default = ["skia", "desktop"]
web = ["dep:web-sys", "web-gl", "dep:wasm-bindgen-futures"]
desktop = ["dep:glutin", "dep:glutin-winit", "dep:winit", "dep:tokio", "dep:reqwest"]
```

Desktop dependencies like `tokio`, `glutin`, and `winit` are automatically excluded when building with `--no-default-features --features="web"`.

#### Asset Processing Pipeline

```rust
struct WebAssetPipeline {
    css_processor: CssProcessor,
    js_bundler: JsBundler,
    image_optimizer: ImageOptimizer,
    font_subsetter: FontSubsetter,
}

impl WebAssetPipeline {
    fn process(&self, assets: &AssetManifest) -> Result<ProcessedAssets, Error> {
        // 1. Optimize images (compression, format conversion)
        let optimized_images = self.image_optimizer.process(&assets.images)?;
        
        // 2. Bundle and minify CSS
        let bundled_css = self.css_processor.bundle(&assets.stylesheets)?;
        
        // 3. Process JavaScript
        let js_bundle = self.js_bundler.bundle(&assets.scripts)?;
        
        // 4. Subset fonts
        let subsetted_fonts = self.font_subsetter.subset(&assets.fonts)?;
        
        Ok(ProcessedAssets {
            images: optimized_images,
            stylesheets: bundled_css,
            scripts: js_bundle,
            fonts: subsetted_fonts,
        })
    }
}
```

### Desktop Target Processing

For desktop applications (`--target desktop`):

#### Native Compilation

```rust
// Platform-specific configuration
#[cfg(target_os = "windows")]
fn configure_windows_build() {
    // Windows-specific optimizations
    embed_manifest();
    configure_icon();
    setup_dependencies();
}

#[cfg(target_os = "macos")]
fn configure_macos_build() {
    // macOS bundle creation
    create_app_bundle();
    code_sign_if_configured();
    create_dmg_if_requested();
}

#[cfg(target_os = "linux")]
fn configure_linux_build() {
    // Linux packaging
    create_appimage_if_requested();
    generate_desktop_entry();
}
```

#### Asset Embedding

```rust
// Assets can be embedded directly in the binary
const EMBEDDED_ASSETS: &[(&str, &[u8])] = &[
    ("logo.png", include_bytes!("../static/logo.png")),
    ("styles.css", include_bytes!("../target/orbit_codegen/styles/bundle.css")),
];
```

### Mobile Target Processing

For mobile platforms (`--target mobile`):

#### Cross-Compilation Setup

```rust
// Mobile-specific build configuration
struct MobileConfig {
    target_arch: MobileArch,
    min_sdk_version: u32,
    app_identifier: String,
    signing_identity: Option<String>,
}

enum MobileArch {
    iOS { device: bool, simulator: bool },
    Android { abi: Vec<AndroidAbi> },
}
```

---

## Phase 6: Optimization & Output

### Production Optimizations

#### Code Optimization

```rust
// Optimization passes applied in release builds
enum OptimizationPass {
    DeadCodeElimination,
    InlineSmallFunctions,
    ConstantFolding,
    LoopUnrolling,
    VectorizeOperations,
    CompressStrings,
}
```

#### Asset Optimization

```rust
struct AssetOptimizer {
    image_compressor: ImageCompressor,
    css_minifier: CssMinifier,
    js_minifier: JsMinifier,
    font_optimizer: FontOptimizer,
}

impl AssetOptimizer {
    fn optimize_images(&self, images: &[Image]) -> Vec<OptimizedImage> {
        images.par_iter().map(|img| {
            match img.format {
                ImageFormat::Png => self.compress_png(img),
                ImageFormat::Jpeg => self.compress_jpeg(img),
                ImageFormat::Svg => self.optimize_svg(img),
                ImageFormat::WebP => self.optimize_webp(img),
            }
        }).collect()
    }
}
```

### Build Artifacts

#### Development Build Output

```
target/debug/
├── my_app                  # Debug binary
├── deps/                   # Dependency artifacts
└── build/                  # Build script outputs
    └── my_app-*/
        └── out/            # Generated code
```

#### Production Build Output

```
dist/
├── index.html              # Entry point (web)
├── my_app.wasm            # WASM binary (web)
├── my_app.js              # JS runtime (web)
├── my_app                 # Native binary (desktop)
├── assets/
│   ├── styles.css         # Bundled styles
│   ├── images/            # Optimized images
│   └── fonts/             # Subsetted fonts
├── chunks/                # Code-split bundles
│   ├── vendor.js
│   ├── components.js
│   └── utilities.js
└── manifest.json          # Asset manifest
```

### Performance Monitoring

Orbiton includes built-in build performance monitoring:

```rust
#[derive(Debug)]
struct BuildMetrics {
    total_build_time: Duration,
    phase_timings: HashMap<BuildPhase, Duration>,
    memory_usage: MemoryUsage,
    cache_hit_rate: f64,
    artifact_sizes: HashMap<ArtifactType, u64>,
}

enum BuildPhase {
    SourceDiscovery,
    Parsing,
    CodeGeneration,
    RustCompilation,
    AssetProcessing,
    Optimization,
    Bundling,
}
```

---

## Debugging Compilation Issues

### Verbose Output

Enable detailed compilation logging:

```bash
# Maximum verbosity
orbiton build --verbose --log-level trace

# Save build logs
orbiton build --log-file build.log
```

### Intermediate Artifacts

Inspect generated code:

```bash
# Keep generated files for inspection
orbiton build --keep-generated

# Show only code generation
orbiton build --dry-run --show-generated
```

### Performance Profiling

```bash
# Profile compilation performance
orbiton build --profile-compilation

# Memory usage analysis
orbiton build --memory-profile
```

---

## Extension Points

### Custom Code Generators

Implement custom code generation:

```rust
trait CodeGenerator {
    fn generate_component(&self, ast: &ComponentAST) -> Result<String, Error>;
    fn generate_styles(&self, styles: &StyleAST) -> Result<String, Error>;
    fn supports_target(&self, target: &BuildTarget) -> bool;
}

// Register custom generator
#[no_mangle]
pub extern "C" fn register_generator() -> Box<dyn CodeGenerator> {
    Box::new(MyCustomGenerator::new())
}
```

### Build Plugins

Create build-time plugins:

```rust
trait BuildPlugin {
    fn name(&self) -> &str;
    fn before_parse(&self, source: &str) -> Result<String, Error>;
    fn after_codegen(&self, generated: &str) -> Result<String, Error>;
    fn finalize_build(&self, artifacts: &BuildArtifacts) -> Result<(), Error>;
}
```

### Asset Processors

Custom asset processing:

```rust
trait AssetProcessor {
    fn supported_extensions(&self) -> &[&str];
    fn process(&self, asset: &Asset) -> Result<ProcessedAsset, Error>;
    fn optimize(&self, asset: &ProcessedAsset) -> Result<OptimizedAsset, Error>;
}
```

---

## Testing Compilation

### Unit Testing Code Generation

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_template_compilation() {
        let template = r#"
            <div class="container">
                <h1>{{ title }}</h1>
            </div>
        "#;
        
        let ast = parse_template(template).unwrap();
        let generated = generate_render_function(&ast).unwrap();
        
        assert!(generated.contains("Node::new_element(\"div\")"));
        assert!(generated.contains("add_class(\"container\")"));
    }
}
```

### Integration Testing

```rust
#[test]
fn test_full_compilation_pipeline() {
    let temp_dir = tempdir().unwrap();
    create_test_project(&temp_dir);
    
    let result = compile_project(&temp_dir, &BuildConfig::default());
    
    assert!(result.is_ok());
    assert!(temp_dir.path().join("target/debug/test_app").exists());
}
```

---

## Performance Characteristics

### Compilation Speed

Typical compilation times (on modern hardware):

| Project Size | Cold Build | Incremental Build |
|-------------|------------|-------------------|
| Small (10 components) | 2-5 seconds | 0.5-1 second |
| Medium (50 components) | 8-15 seconds | 1-3 seconds |
| Large (200+ components) | 30-60 seconds | 3-8 seconds |

### Memory Usage

| Phase | Memory Usage |
|-------|-------------|
| Parsing | 10-50 MB |
| Code Generation | 20-100 MB |
| Rust Compilation | 100-500 MB |
| Asset Processing | 50-200 MB |

### Optimization Impact

| Optimization | Size Reduction | Build Time Impact |
|-------------|----------------|-------------------|
| Dead Code Elimination | 15-30% | +10-20% |
| CSS Minification | 20-40% | +5-10% |
| Image Optimization | 30-70% | +20-50% |
| Tree Shaking | 10-25% | +15-25% |

---

## Contributing to the Compiler

### Setting Up Development Environment

```bash
# Clone the repository
git clone https://github.com/orbitrs/orbitrs.git
cd orbitrs/sdk

# Install development dependencies
rustup component add rustfmt clippy
cargo install cargo-watch

# Run tests
cargo test --workspace

# Start development with auto-reload
cargo watch -x "test --workspace"
```

### Compiler Architecture

The compiler is organized into distinct crates:

- **`orbit-parser`**: Lexing and parsing of `.orbit` files
- **`orbit-codegen`**: Code generation and optimization
- **`orbit-build`**: Build system integration
- **`orbiton`**: CLI interface and project management

### Adding New Features

1. **Parser Changes**: Modify `orbit-parser` for new syntax
2. **Code Generation**: Update `orbit-codegen` for new constructs
3. **Tests**: Add comprehensive tests for new features
4. **Documentation**: Update this document and user guides

### Performance Guidelines

- **Incremental Processing**: Design features for incremental compilation
- **Memory Efficiency**: Use streaming parsing where possible
- **Parallelization**: Leverage parallel processing for independent tasks
- **Caching**: Cache expensive computations and intermediate results

---

## Further Reading

- [Orbiton CLI Reference](../api/orbiton-cli.md) - Command-line interface documentation
- [Component Model](./component-model.md) - Understanding Orbit components
- [Performance Optimization](../guides/performance-optimization.md) - Application optimization
- [Contributing Guide](../../CONTRIBUTING.md) - How to contribute to Orbit
- [Architecture Overview](../architecture.md) - Framework architecture

Scoped CSS is adjusted by:

1. **Generating Unique Class Names** for each component file.
2. **Rewriting CSS selectors** to include the unique scope token.
3. **Injecting a style tag** at runtime or bundling a stylesheet.

### Integration

The generated render code and scoped style info are inserted into a Rust module alongside the `struct` definition from the `<code>` section.

---

## 3. Rust Compilation

After code generation, the project uses `cargo build`:

- **Codegen Output**: Generated files are placed under `target/orbit_codegen/` or merged in-memory via build script.
- **Build Script (`build.rs`)**: Ensures codegen runs before Rust compilation.
- **Rust Compiler**: Compiles all Rust sources (including generated code) into a binary or library.

### Build Artifacts

- **Development Build**: Includes debug symbols and unminified assets.
- **Release Build**: Optimized binary with minified CSS and JS (if web target).

---

## 4. Bundling & Assets

For web targets (`orbiton build --target web`):

1. **Asset Copy**: Static files from `static/` are copied into `dist/`.
2. **CSS Bundling**: Component styles are concatenated or inlined.
3. **JavaScript Glue**: A small JS runtime is bundled to bootstrap the WASM application.
4. **WASM Processing**: The compiled `.wasm` file is optimized (e.g., `wasm-opt`) and placed in `dist/`.

For desktop targets, assets are packaged alongside the executable.

---

## 5. Build Hooks & Extensions

- **Custom Pre-build Hooks**: Developers can add scripts in `orbiton.toml` under `[build.hooks]`.
- **Post-build Actions**: Deploy scripts or asset optimization can be configured.

---

## Further Reading

- [`orbiton/src/dev_server.rs`](../../orbiton/src/dev_server.rs) for development workflow.
- [`orbit/CHANGELOG.md`](../../orbit/CHANGELOG.md) for recent codegen improvements.
- [`docs/DOCUMENTATION_PLAN.md`](/docs/DOCUMENTATION_PLAN.md) for overall documentation structure.
