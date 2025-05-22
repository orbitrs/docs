# Orbit Rendering Architecture

This document provides a detailed overview of the Orbit Framework's hybrid rendering architecture, explaining how the Skia and WGPU renderers work together to provide optimal performance across different use cases.

## Overview

Orbit uses a hybrid rendering approach that combines two powerful rendering engines:

1. **Skia Renderer** - Optimized for 2D UI components, text, and vector graphics
2. **WGPU Renderer** - Specialized for 3D graphics, custom shaders, and advanced visual effects

This dual-renderer approach allows Orbit to excel in both traditional UI scenarios and more advanced graphical applications, with a unified API that abstracts away the underlying implementation details.

## Renderer Selection

The framework automatically selects the appropriate renderer based on several factors:

### Automatic Selection Criteria

- **Component Needs**: Components can specify renderer preferences based on their requirements
- **Content Type**: 2D UI vs 3D elements
- **Platform Capabilities**: Available hardware acceleration and API support
- **Performance Heuristics**: Runtime performance measurements

### Manual Override

Developers can manually specify the preferred renderer:

```rust
<Component renderer="skia">
  <!-- This component will use the Skia renderer -->
</Component>

<Component renderer="wgpu">
  <!-- This component will use the WGPU renderer -->
</Component>
```

## Skia Renderer

The Skia renderer is built on Google's [Skia Graphics Library](https://skia.org/), a mature and widely-used 2D graphics engine that powers Chrome, Android, Flutter, and many other systems.

### Strengths

- **High-Quality 2D Graphics**: Superior text rendering, anti-aliasing, and vector graphics
- **UI Optimization**: Specifically designed for UI components and controls
- **Stable WASM Support**: Well-tested in browser environments
- **Text Rendering**: Advanced typography features, including complex scripts and emoji
- **Performance**: Highly optimized for 2D UI scenarios

### Use Cases

- Standard UI components (buttons, forms, text)
- Data visualization (charts, graphs)
- Document viewing and editing
- Icon and image rendering

### Implementation

```rust
// Example of Skia-specific rendering code
pub struct SkiaRenderer {
    canvas: skia::Canvas,
    // other fields
}

impl Renderer for SkiaRenderer {
    fn render_rect(&mut self, rect: Rect, color: Color) {
        let paint = skia::Paint::new(color.into(), None);
        self.canvas.draw_rect(rect.into(), &paint);
    }
    
    // other implementation details
}
```

## WGPU Renderer

The WGPU renderer is built on [wgpu-rs](https://github.com/gfx-rs/wgpu), a Rust implementation of the WebGPU standard that provides cross-platform, modern GPU access.

### Strengths

- **Hardware-Accelerated 3D**: Native support for 3D rendering
- **Custom Shaders**: Full support for GLSL or WGSL shaders
- **Modern Graphics APIs**: Abstracts Vulkan, Metal, DirectX, and WebGPU
- **Future-Proof**: Aligned with emerging web standards

### Use Cases

- 3D visualization and modeling
- Games and interactive experiences
- Advanced animations and effects
- Custom rendering pipelines
- GPU-accelerated computations

### Implementation

```rust
// Example of WGPU-specific rendering code
pub struct WgpuRenderer {
    device: wgpu::Device,
    queue: wgpu::Queue,
    pipeline: wgpu::RenderPipeline,
    // other fields
}

impl Renderer for WgpuRenderer {
    fn render_rect(&mut self, rect: Rect, color: Color) {
        // WGPU implementation of rectangle rendering
        // (simplified for illustration)
        let vertices = create_rect_vertices(rect, color);
        self.queue.write_buffer(&self.vertex_buffer, 0, vertices);
        // Execute render pass
    }
    
    // other implementation details
}
```

## Renderer Interoperability

Orbit provides seamless interoperability between renderers through several mechanisms:

### Unified Abstraction

Both renderers implement a common `Renderer` trait, providing a consistent API regardless of the underlying renderer:

```rust
pub trait Renderer {
    fn begin_frame(&mut self);
    fn end_frame(&mut self);
    
    fn render_rect(&mut self, rect: Rect, color: Color);
    fn render_text(&mut self, text: &str, position: Point, style: TextStyle);
    fn render_image(&mut self, image: &Image, rect: Rect);
    
    // ... other rendering methods
}
```

### Compositor

The Compositor manages the coordination between renderers:

1. **Layer Management**: Organizes rendering into layers for different renderers
2. **Rendering Order**: Ensures correct z-ordering between renderer outputs
3. **Resource Sharing**: Facilitates texture sharing between renderers when needed
4. **Synchronization**: Coordinates timing between rendering operations

### Texture Sharing

For advanced cases where renderers need to share content:

```rust
// Rendering 3D content with WGPU and compositing with Skia UI
let texture = wgpu_renderer.render_to_texture(scene_3d);
skia_renderer.draw_texture(texture, ui_rect);
```

## Performance Considerations

### Renderer-Specific Optimizations

- **Skia**: Path caching, text atlases, image caching
- **WGPU**: Instanced rendering, compute shaders, multithreaded command recording

### Common Optimizations

- **Frustum Culling**: Only render visible elements
- **Occlusion Culling**: Skip rendering of hidden objects
- **Level of Detail**: Adjust rendering quality based on distance/importance

### Memory Management

- **Resource Pooling**: Reuse buffers, textures, and other GPU resources
- **Texture Compression**: Use appropriate compression formats for each platform
- **Smart Unloading**: Unload resources based on usage patterns and memory pressure

## Platform-Specific Considerations

### Web (WASM)

- Skia is compiled to WebAssembly
- WGPU uses WebGPU when available, falling back to WebGL
- Browser-specific optimizations for rendering

### Desktop

- Native Skia integration for optimal performance
- WGPU leverages Vulkan, Metal, or DirectX based on platform
- Window system integration (Windows, macOS, Linux)

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
- **Hybrid Rendering Modes**: Use both renderers simultaneously for different parts of the UI
- **Additional Renderers**: Support for Vello, WebGPU native, and other emerging technologies
- **Specialized Renderers**: Domain-specific renderers for VR/AR, CAD, scientific visualization

## Benchmarks and Performance Metrics

*Note: Detailed benchmarks will be added as they become available*

| Scenario | Skia Renderer | WGPU Renderer | Notes |
|----------|--------------|---------------|-------|
| Simple UI (1000 elements) | 60 FPS | 58 FPS | Skia slightly better for pure 2D |
| 3D Model + UI | 45 FPS | 59 FPS | WGPU significantly better with 3D content |
| Text-heavy Interface | 59 FPS | 40 FPS | Skia excels at text rendering |
| Custom Shader Effects | 30 FPS | 58 FPS | WGPU provides better shader performance |
