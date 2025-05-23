# `orbit::render` API Reference

This document provides a comprehensive reference for the rendering system in the Orbit UI Framework.

## Overview

The `orbit::render` module provides a hybrid rendering architecture that combines different rendering backends (Skia, WGPU) to deliver optimal performance across various platforms and use cases. The framework automatically selects the appropriate renderer based on content requirements while maintaining a consistent API.

## Core Types

### `Renderer`

```rust
pub trait Renderer: Send + 'static {
    /// Initialize the renderer
    fn initialize(&mut self, config: RendererConfig) -> Result<(), RendererError>;
    
    /// Begin a new frame with the specified dimensions
    fn begin_frame(&mut self, width: u32, height: u32) -> Result<(), RendererError>;
    
    /// End the current frame and present it
    fn end_frame(&mut self) -> Result<(), RendererError>;
    
    /// Render a rectangle with the specified position, size and color
    fn render_rect(&mut self, rect: Rect, color: Color) -> Result<(), RendererError>;
    
    /// Render text with the specified position, content, and style
    fn render_text(&mut self, text: &str, position: Point, style: TextStyle) -> Result<(), RendererError>;
    
    /// Render an image at the specified position
    fn render_image(&mut self, image: &Image, rect: Rect) -> Result<(), RendererError>;
    
    /// Render a component node and its children
    fn render_component(&mut self, component: &dyn Component) -> Result<(), RendererError>;
    
    /// Clean up resources
    fn cleanup(&mut self) -> Result<(), RendererError>;
}
```

### `RendererType`

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum RendererType {
    /// Skia-based renderer
    Skia,
    /// WGPU-based renderer
    Wgpu,
    /// Automatic selection based on content
    Auto,
}
```

### `RendererConfig`

```rust
#[derive(Debug, Clone)]
pub struct RendererConfig {
    /// Type of renderer to use
    pub renderer_type: RendererType,
    /// Whether to enable antialiasing
    pub antialiasing: bool,
    /// Target frames per second
    pub target_fps: Option<u32>,
    /// Custom renderer options as key-value pairs
    pub options: HashMap<String, Value>,
}
```

### `RendererError`

```rust
#[derive(Debug, thiserror::Error)]
pub enum RendererError {
    #[error("Initialization error: {0}")]
    Init(String),
    
    #[error("Rendering error: {0}")]
    Render(String),
    
    #[error("Resource error: {0}")]
    Resource(String),
    
    #[error("Pipeline error: {0}")]
    Pipeline(String),
    
    #[error("Platform error: {0}")]
    Platform(String),
}
```

## Renderer Implementations

### `SkiaRenderer`

```rust
pub struct SkiaRenderer {
    // Internal fields
}
```

The Skia renderer uses Google's Skia graphics library for high-quality 2D rendering, optimized for UI components, text, and vector graphics.

#### Methods

```rust
impl SkiaRenderer {
    /// Create a new Skia renderer with default settings
    pub fn new() -> Self;
    
    /// Create a new Skia renderer with custom settings
    pub fn with_config(config: SkiaConfig) -> Self;
    
    /// Access the underlying Skia canvas
    pub fn canvas(&mut self) -> &mut skia::Canvas;
    
    // Additional Skia-specific methods
}

impl Renderer for SkiaRenderer {
    // Implementation of the Renderer trait
}
```

#### Strengths

- High-quality 2D graphics with superior antialiasing
- Excellent text rendering with advanced typography support
- Optimized for UI components and controls
- Stable WebAssembly support
- Mature and well-tested codebase

#### Use Cases

- Standard UI components (buttons, forms, text)
- Data visualization (charts, graphs)
- Document viewing and editing
- Icon and image rendering

### `WgpuRenderer`

```rust
pub struct WgpuRenderer {
    // Internal fields
}
```

The WGPU renderer uses the WebGPU API for hardware-accelerated rendering, optimized for 3D content, custom shaders, and advanced visual effects.

#### Methods

```rust
impl WgpuRenderer {
    /// Create a new WGPU renderer
    pub fn new() -> Self;
    
    /// Create a new WGPU renderer with custom settings
    pub fn with_config(config: WgpuConfig) -> Self;
    
    /// Access the underlying device
    pub fn device(&self) -> &wgpu::Device;
    
    /// Access the command queue
    pub fn queue(&self) -> &wgpu::Queue;
    
    // WGPU-specific methods
}

impl Renderer for WgpuRenderer {
    // Implementation of the Renderer trait
}
```

#### Strengths

- Hardware-accelerated 3D rendering
- Full support for custom shaders (GLSL, WGSL)
- Modern graphics APIs (Vulkan, Metal, DirectX, WebGPU)
- Parallel processing capabilities
- Future-proof architecture aligned with web standards

#### Use Cases

- 3D visualization and modeling
- Games and interactive experiences
- Advanced animations and effects
- Custom rendering pipelines
- GPU-accelerated computations

## Renderer Selection

The framework automatically selects the appropriate renderer based on several factors:

### Selection Criteria

- **Component Needs**: Components can specify renderer preferences
- **Content Type**: 2D UI vs 3D elements
- **Platform Capabilities**: Available hardware acceleration and API support
- **Performance Heuristics**: Runtime performance measurements

### Manual Selection

You can manually specify the preferred renderer for a component:

```rust
// In a component template
<Component renderer="skia">
  <!-- This component will use the Skia renderer -->
</Component>

<Component renderer="wgpu">
  <!-- This component will use the WGPU renderer -->
</Component>

// In Rust code
let renderer = match content_type {
    ContentType::Standard2D => create_renderer(RendererType::Skia),
    ContentType::Interactive3D => create_renderer(RendererType::Wgpu),
    _ => create_renderer(RendererType::Auto),
};
```

## Virtual DOM

```rust
pub struct VNode {
    // Internal fields
}
```

The Virtual DOM is an in-memory representation of the component tree that enables efficient diffing and updates.

### Methods

```rust
impl VNode {
    /// Create an element node
    pub fn element(tag: &str, props: Props, children: Vec<VNode>) -> Self;
    
    /// Create a text node
    pub fn text(content: &str) -> Self;
    
    /// Create a component node
    pub fn component(component: Box<dyn Component>, props: Props) -> Self;
    
    /// Diff this node with another and generate a patch
    pub fn diff(&self, other: &VNode) -> Patch;
    
    /// Apply a patch to this node
    pub fn apply_patch(&mut self, patch: Patch);
}
```

## Renderer Compositor

```rust
pub struct RendererCompositor {
    // Internal fields
}
```

The Compositor manages the coordination between different renderers:

### Methods

```rust
impl RendererCompositor {
    /// Create a new compositor
    pub fn new() -> Self;
    
    /// Add a renderer to the compositor
    pub fn add_renderer(&mut self, renderer: Box<dyn Renderer>);
    
    /// Begin a new composite frame
    pub fn begin_frame(&mut self, width: u32, height: u32) -> Result<(), RendererError>;
    
    /// End the current frame and present the composited result
    pub fn end_frame(&mut self) -> Result<(), RendererError>;
    
    /// Render a component using the appropriate renderer
    pub fn render_component(&mut self, component: &dyn Component) -> Result<(), RendererError>;
}
```

### Composition Process

1. **Layer Management**: Organizes rendering into layers for different renderers
2. **Rendering Order**: Ensures correct z-ordering between renderer outputs
3. **Resource Sharing**: Facilitates texture sharing between renderers when needed
4. **Synchronization**: Coordinates timing between rendering operations

## Drawing Primitives

### `Rect`

```rust
#[derive(Debug, Clone, Copy)]
pub struct Rect {
    pub x: f32,
    pub y: f32,
    pub width: f32,
    pub height: f32,
}
```

### `Point`

```rust
#[derive(Debug, Clone, Copy)]
pub struct Point {
    pub x: f32,
    pub y: f32,
}
```

### `Color`

```rust
#[derive(Debug, Clone, Copy)]
pub struct Color {
    pub r: f32,
    pub g: f32,
    pub b: f32,
    pub a: f32,
}

impl Color {
    // Color creation helpers
    pub const TRANSPARENT: Self = Self { r: 0.0, g: 0.0, b: 0.0, a: 0.0 };
    pub const BLACK: Self = Self { r: 0.0, g: 0.0, b: 0.0, a: 1.0 };
    pub const WHITE: Self = Self { r: 1.0, g: 1.0, b: 1.0, a: 1.0 };
    pub const RED: Self = Self { r: 1.0, g: 0.0, b: 0.0, a: 1.0 };
    pub const GREEN: Self = Self { r: 0.0, g: 1.0, b: 0.0, a: 1.0 };
    pub const BLUE: Self = Self { r: 0.0, g: 0.0, b: 1.0, a: 1.0 };
    
    // Create a color from RGB values (0-255)
    pub fn rgb(r: u8, g: u8, b: u8) -> Self;
    
    // Create a color from RGBA values (0-255)
    pub fn rgba(r: u8, g: u8, b: u8, a: u8) -> Self;
    
    // Create a color from a hex string
    pub fn from_hex(hex: &str) -> Result<Self, ColorError>;
}
```

### `TextStyle`

```rust
#[derive(Debug, Clone)]
pub struct TextStyle {
    pub font_family: String,
    pub font_size: f32,
    pub font_weight: FontWeight,
    pub color: Color,
    pub alignment: TextAlignment,
    pub line_height: Option<f32>,
}
```

### `Image`

```rust
#[derive(Debug, Clone)]
pub struct Image {
    // Internal fields
}

impl Image {
    /// Load an image from a file path
    pub fn from_path<P: AsRef<Path>>(path: P) -> Result<Self, ImageError>;
    
    /// Load an image from memory
    pub fn from_memory(data: &[u8]) -> Result<Self, ImageError>;
    
    /// Create an image from raw RGBA data
    pub fn from_rgba(width: u32, height: u32, data: Vec<u8>) -> Result<Self, ImageError>;
    
    /// Get the dimensions of the image
    pub fn dimensions(&self) -> (u32, u32);
}
```

## Performance Considerations

### Renderer-Specific Optimizations

- **Skia**: Path caching, text atlases, image caching
- **WGPU**: Instanced rendering, compute shaders, multithreaded command recording

### Common Optimizations

The framework employs several optimization techniques:

- **Frustum Culling**: Only render visible elements
- **Occlusion Culling**: Skip rendering of hidden objects
- **Level of Detail**: Adjust rendering quality based on distance/importance
- **Batching**: Group similar draw calls to reduce state changes
- **View Clipping**: Only process elements within the viewport

### Memory Management

Orbit implements resource management strategies:

- **Resource Pooling**: Reuse buffers, textures, and other GPU resources
- **Texture Compression**: Use appropriate compression formats per platform
- **Smart Unloading**: Unload resources based on usage patterns

## Platform-Specific Considerations

### Web (WASM)

- Skia compiles to WebAssembly
- WGPU uses WebGPU when available, falling back to WebGL
- Browser-specific optimizations for rendering

### Desktop

- Native Skia integration for optimal performance
- WGPU leverages Vulkan, Metal, or DirectX based on platform
- Window system integration for Windows, macOS, and Linux

### Mobile (Planned)

- Touch input handling
- Platform-specific rendering optimizations
- Power usage considerations

### Embedded (Planned)

- Reduced feature set for limited hardware
- Optional `no_std` support
- Hardware-specific optimizations

## Future Directions

- **Runtime Renderer Switching**: Dynamically switch renderers based on performance metrics
- **Hybrid Rendering Modes**: Use multiple renderers simultaneously for different UI parts
- **Additional Renderers**: Support for Vello, WebGPU native, and other technologies
- **Specialized Renderers**: Domain-specific renderers for VR/AR, CAD, scientific visualization

## Related Documentation

- [Rendering Architecture](../core-concepts/rendering-architecture.md)
- [Performance Optimization](../guides/performance-optimization.md)
- [Component Model](../core-concepts/component-model.md)
- [Advanced Component Patterns](../core-concepts/advanced-component-patterns.md)
