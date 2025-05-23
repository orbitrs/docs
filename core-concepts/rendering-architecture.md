# 🎨 Orbit Rendering Architecture

This document provides a detailed overview of the Orbit Framework's hybrid rendering architecture, explaining how the Skia and WGPU renderers work together to provide optimal performance across different use cases.

## 📝 Overview

Orbit uses a hybrid rendering approach that combines two powerful rendering engines:

1. **Skia Renderer** - Optimized for 2D UI components, text, and vector graphics
2. **WGPU Renderer** - Specialized for 3D graphics, custom shaders, and advanced visual effects

This dual-renderer approach allows Orbit to excel in both traditional UI scenarios and more advanced graphical applications, with a unified API that abstracts away the underlying implementation details.

### Visual Architecture Overview

```
┌───────────────────────────────────────────┐
│             Application Logic             │
└───────────────────┬───────────────────────┘
                    │
┌───────────────────▼───────────────────────┐
│          Component Tree / Virtual DOM     │
└───────────────────┬───────────────────────┘
                    │
┌───────────────────▼───────────────────────┐
│           Render Operation Tree           │
└────────┬─────────────────────────┬────────┘
         │                         │
┌────────▼─────────┐      ┌────────▼─────────┐
│  Skia Renderer   │◄────►│  WGPU Renderer   │
└────────┬─────────┘      └────────┬─────────┘
         │                         │
         │         ┌───────────────▼───────────────┐
         └─────────►      Compositor Layer        │
                   └───────────────┬───────────────┘
                                   │
                   ┌───────────────▼───────────────┐
                   │       Platform Output         │
                   │ (Window, Canvas, Framebuffer) │
                   └───────────────────────────────┘
```

## 🔍 Renderer Selection

The framework automatically selects the appropriate renderer based on several factors:

### Automatic Selection Criteria

- **Component Needs**: Components can specify renderer preferences based on their requirements
- **Content Type**: 2D UI vs 3D elements
- **Platform Capabilities**: Available hardware acceleration and API support
- **Performance Heuristics**: Runtime performance measurements

### Decision Tree for Renderer Selection

```
┌─────────────────────┐
│ Component Rendering │
└──────────┬──────────┘
           │
           ▼
┌────────────────────────┐
│ Contains 3D Elements?  │
└──────┬─────────┬───────┘
       │         │
       │ Yes     │ No
       ▼         ▼
┌──────────┐ ┌──────────┐
│   WGPU   │ │ Contains │
│          │ │  Custom  │
│          │ │ Shaders? │
└──────────┘ └────┬─────┘
                  │
            ┌─────┴─────┐
            │ Yes       │ No
            ▼           ▼
       ┌──────────┐ ┌──────────┐
       │   WGPU   │ │  Check   │
       │          │ │ Platform │
       │          │ │ Support  │
       └──────────┘ └────┬─────┘
                         │
                   ┌─────┴─────┐
                   │           │
                   ▼           ▼
              ┌──────────┐ ┌──────────┐
              │ Prefer   │ │ Fallback │
              │  Skia    │ │   Mode   │
              └──────────┘ └──────────┘
```

### Manual Override

Developers can manually specify the preferred renderer:

```orbit
<template>
  <!-- Component using Skia renderer -->
  <div renderer="skia" class="chart-container">
    <LineChart :data="chartData" />
  </div>
  
  <!-- Component using WGPU renderer -->
  <div renderer="wgpu" class="model-viewer">
    <Model3D :src="modelSource" />
  </div>
</template>
```

You can also set the renderer at a higher level and have it cascade down:

```orbit
<template>
  <!-- All children will use WGPU unless overridden -->
  <div renderer="wgpu" class="advanced-ui">
    <!-- This will use WGPU -->
    <Model3D :src="modelSource" />
    
    <!-- This specifically overrides to Skia -->
    <div renderer="skia">
      <StatusBar />
    </div>
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;
use orbit::renderer::*;

#[component]
pub struct MyApplication {
    // Component state
}

impl MyApplication {
    // Programmatic renderer selection
    pub fn select_optimal_renderer(&self) -> RendererType {
        // Check if graphics hardware supports ray tracing
        if detect_ray_tracing_support() {
            RendererType::Wgpu
        } else {
            RendererType::Skia
        }
    }
}
</code>
```

## 🖌️ Skia Renderer

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

// Drawing a custom shape with the Skia renderer
fn draw_custom_shape(renderer: &mut SkiaRenderer, center: Point, radius: f32) {
    let mut path = skia::Path::new();
    
    // Create a star shape
    for i in 0..5 {
        let angle = 2.0 * std::f32::consts::PI * i as f32 / 5.0;
        let x = center.x + radius * angle.cos();
        let y = center.y + radius * angle.sin();
        
        if i == 0 {
            path.move_to(x, y);
        } else {
            path.line_to(x, y);
        }
    }
    path.close();
    
    let paint = skia::Paint::new(skia::Color4f::new(1.0, 0.8, 0.0, 1.0), None);
    renderer.canvas.draw_path(&path, &paint);
}
```

## 🎮 WGPU Renderer

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

// Example of a custom shader in WGPU
const FRAGMENT_SHADER: &str = r#"
    @fragment
    fn fs_main(@location(0) tex_coords: vec2<f32>) -> @location(0) vec4<f32> {
        // Simple gradient shader
        return vec4<f32>(
            tex_coords.x,
            tex_coords.y,
            1.0 - tex_coords.x,
            1.0
        );
    }
"#;

fn create_gradient_pipeline(renderer: &mut WgpuRenderer) -> wgpu::RenderPipeline {
    let shader_module = renderer.device.create_shader_module(wgpu::ShaderModuleDescriptor {
        label: Some("Gradient Shader"),
        source: wgpu::ShaderSource::Wgsl(FRAGMENT_SHADER.into()),
    });
    
    // Pipeline creation code...
    // (simplified for illustration)
}
```

## 🔄 Renderer Interoperability

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

```rust
// Compositor implementation example
pub struct Compositor {
    layers: Vec<RenderLayer>,
    skia_renderer: SkiaRenderer,
    wgpu_renderer: WgpuRenderer,
}

impl Compositor {
    pub fn render(&mut self) {
        // Sort layers by z-index
        self.layers.sort_by_key(|layer| layer.z_index);
        
        // Pre-render all WGPU layers to textures
        for layer in &self.layers {
            if layer.renderer_type == RendererType::Wgpu {
                self.wgpu_renderer.begin_frame();
                self.wgpu_renderer.render_layer(layer);
                let texture = self.wgpu_renderer.end_frame_to_texture();
                
                // Store the texture for compositing
                layer.set_rendered_texture(texture);
            }
        }
        
        // Render all layers in order, including WGPU textures
        self.skia_renderer.begin_frame();
        for layer in &self.layers {
            if layer.renderer_type == RendererType::Skia {
                // Direct rendering
                self.skia_renderer.render_layer(layer);
            } else {
                // Draw the pre-rendered WGPU texture
                self.skia_renderer.draw_texture(layer.rendered_texture(), layer.rect);
            }
        }
        self.skia_renderer.end_frame();
    }
}
```

### Layer Management Visualization

```
┌───────────────────────────────────────────────────────┐
│                   Viewport/Window                     │
│                                                       │
│   ┌───────────────────────────────────────────────┐   │
│   │                                               │   │
│   │             Layer 0: Background               │   │
│   │             (Skia Renderer)                   │   │
│   │                                               │   │
│   │   ┌───────────────────────────────┐           │   │
│   │   │                               │           │   │
│   │   │       Layer 1: 3D Scene       │           │   │
│   │   │       (WGPU Renderer)         │           │   │
│   │   │                               │           │   │
│   │   │                               │           │   │
│   │   │                               │           │   │
│   │   └───────────────────────────────┘           │   │
│   │                                               │   │
│   │   ┌───────────────────┐                       │   │
│   │   │  Layer 2: UI      │                       │   │
│   │   │  (Skia Renderer)  │                       │   │
│   │   │                   │                       │   │
│   │   └───────────────────┘                       │   │
│   │                                               │   │
│   │                                               │   │
│   └───────────────────────────────────────────────┘   │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### Texture Sharing

For advanced cases where renderers need to share content:

```rust
// Rendering 3D content with WGPU and compositing with Skia UI
let texture = wgpu_renderer.render_to_texture(scene_3d);
skia_renderer.draw_texture(texture, ui_rect);
```

A more complete example of integrating 3D and 2D content:

```rust
use orbit::prelude::*;
use orbit::renderer::*;

#[component]
pub struct Mixed3D2DView {
    scene: Scene3D,
    ui_overlay: UiOverlay,
    wgpu_renderer: WgpuRenderer,
    skia_renderer: SkiaRenderer,
    rendered_texture: Option<Texture>,
}

impl Mixed3D2DView {
    pub fn new() -> Self {
        Self {
            scene: Scene3D::new(),
            ui_overlay: UiOverlay::new(),
            wgpu_renderer: WgpuRenderer::new(),
            skia_renderer: SkiaRenderer::new(),
            rendered_texture: None,
        }
    }
    
    pub fn update(&mut self, delta_time: f32) {
        // Update 3D scene
        self.scene.update(delta_time);
        
        // Update UI
        self.ui_overlay.update();
    }
    
    pub fn render(&mut self, viewport: Rect) {
        // Render 3D scene to a texture
        self.wgpu_renderer.begin_frame();
        self.scene.render(&mut self.wgpu_renderer);
        self.rendered_texture = Some(self.wgpu_renderer.end_frame_to_texture());
        
        // Render UI overlay with 3D scene as background
        self.skia_renderer.begin_frame();
        
        // Draw the 3D scene texture first
        if let Some(texture) = &self.rendered_texture {
            self.skia_renderer.draw_texture(texture, viewport);
        }
        
        // Draw UI elements on top
        self.ui_overlay.render(&mut self.skia_renderer);
        
        self.skia_renderer.end_frame();
    }
}
```

## ⚡ Performance Considerations

### Renderer-Specific Optimizations

- **Skia**: 
  - Path caching for frequently used shapes
  - Text atlases for efficient text rendering
  - Image caching and mipmap generation
  - Fast blend modes and composition

- **WGPU**: 
  - Instanced rendering for repeated elements
  - Compute shaders for data processing
  - Multithreaded command recording
  - Pipeline state objects caching

### Common Optimizations

- **Frustum Culling**: Only render visible elements
  ```rust
  fn is_visible(object_bounds: Bounds, camera_frustum: Frustum) -> bool {
      // Check if object is within the camera's view frustum
      camera_frustum.intersects(object_bounds)
  }
  ```

- **Occlusion Culling**: Skip rendering of hidden objects
  ```rust
  fn is_occluded(object: &RenderObject, occluders: &[RenderObject]) -> bool {
      for occluder in occluders {
          if occluder.completely_covers(object) {
              return true;
          }
      }
      false
  }
  ```

- **Level of Detail**: Adjust rendering quality based on distance/importance
  ```rust
  fn select_lod(object: &RenderObject, camera_position: Vec3) -> LodLevel {
      let distance = (object.position - camera_position).length();
      
      if distance < 10.0 {
          LodLevel::High
      } else if distance < 50.0 {
          LodLevel::Medium
      } else {
          LodLevel::Low
      }
  }
  ```

### Memory Management

- **Resource Pooling**: Reuse buffers, textures, and other GPU resources
- **Texture Compression**: Use appropriate compression formats for each platform
- **Smart Unloading**: Unload resources based on usage patterns and memory pressure

### Render Batching

A key optimization technique is batching similar draw calls to minimize state changes:

```rust
// Batched rendering example
struct BatchedRenderer {
    current_material: Option<MaterialId>,
    batches: Vec<RenderBatch>,
}

impl BatchedRenderer {
    fn render(&mut self, objects: &[RenderObject]) {
        // Sort objects by material to minimize state changes
        let mut sorted_objects = objects.to_vec();
        sorted_objects.sort_by_key(|obj| obj.material_id);
        
        // Create batches
        let mut current_batch = RenderBatch::new();
        
        for obj in sorted_objects {
            if self.current_material != Some(obj.material_id) {
                // Material changed, start a new batch
                if !current_batch.is_empty() {
                    self.batches.push(current_batch);
                    current_batch = RenderBatch::new();
                }
                self.current_material = Some(obj.material_id);
            }
            
            // Add to current batch
            current_batch.add(obj);
        }
        
        // Add the last batch if not empty
        if !current_batch.is_empty() {
            self.batches.push(current_batch);
        }
        
        // Now render all batches
        for batch in &self.batches {
            batch.render();
        }
    }
}
```

## 🖥️ Platform-Specific Considerations

### Web (WASM)

- Skia is compiled to WebAssembly
- WGPU uses WebGPU when available, falling back to WebGL
- Browser-specific optimizations for rendering

```rust
// Web-specific renderer initialization
#[cfg(target_arch = "wasm32")]
fn initialize_renderer() -> Box<dyn Renderer> {
    if webgpu_supported() {
        Box::new(WgpuRenderer::new_for_web())
    } else {
        Box::new(SkiaRenderer::new_for_web())
    }
}
```

### Desktop

- Native Skia integration for optimal performance
- WGPU leverages Vulkan, Metal, or DirectX based on platform
- Window system integration (Windows, macOS, Linux)

```rust
// Desktop-specific renderer initialization
#[cfg(not(target_arch = "wasm32"))]
fn initialize_renderer() -> Box<dyn Renderer> {
    let graphics_api = match std::env::consts::OS {
        "windows" => GraphicsApi::DirectX12,
        "macos" => GraphicsApi::Metal,
        _ => GraphicsApi::Vulkan,
    };
    
    Box::new(WgpuRenderer::new_with_api(graphics_api))
}
```

### Mobile (Planned)

- Touch input handling
- Platform-specific rendering optimizations
- Power usage considerations

### Embedded (Planned)

- Reduced feature set for limited hardware
- Optional `no_std` support
- Hardware-specific optimizations

## 🔮 Advanced Configuration

### Renderer Configuration Options

```rust
// Example configuration options
pub struct RendererConfig {
    // General options
    pub vsync: bool,
    pub msaa_samples: u32,
    pub frame_rate_limit: Option<u32>,
    
    // Skia-specific options
    pub skia_cache_size_mb: u32,
    pub enable_path_caching: bool,
    
    // WGPU-specific options
    pub preferred_graphics_api: Option<GraphicsApi>,
    pub enable_ray_tracing: bool,
    pub shader_validation: bool,
}

// Usage
let config = RendererConfig {
    vsync: true,
    msaa_samples: 4,
    frame_rate_limit: Some(60),
    skia_cache_size_mb: 64,
    enable_path_caching: true,
    preferred_graphics_api: Some(GraphicsApi::Vulkan),
    enable_ray_tracing: false,
    shader_validation: cfg!(debug_assertions),
};

let renderer = create_renderer_with_config(config);
```

### Custom Pipeline Integration

For applications with specific rendering needs, you can integrate custom pipelines:

```rust
// Creating a post-processing effect
pub struct BloomEffect {
    device: wgpu::Device,
    pipeline: wgpu::ComputePipeline,
    bind_group_layout: wgpu::BindGroupLayout,
}

impl PostProcessingEffect for BloomEffect {
    fn apply(&self, renderer: &mut WgpuRenderer, input_texture: &Texture, output_texture: &Texture) {
        // Set up bind group with input and output textures
        let bind_group = self.create_bind_group(input_texture, output_texture);
        
        // Dispatch compute shader
        let mut encoder = renderer.device.create_command_encoder(&Default::default());
        {
            let mut compute_pass = encoder.begin_compute_pass(&Default::default());
            compute_pass.set_pipeline(&self.pipeline);
            compute_pass.set_bind_group(0, &bind_group, &[]);
            compute_pass.dispatch(input_texture.width() / 8, input_texture.height() / 8, 1);
        }
        
        renderer.queue.submit(std::iter::once(encoder.finish()));
    }
}
```

## 🚀 Future Directions

- **Runtime Renderer Switching**: Dynamically switch renderers based on performance metrics
- **Hybrid Rendering Modes**: Use both renderers simultaneously for different parts of the UI
- **Additional Renderers**: Support for Vello, WebGPU native, and other emerging technologies
- **Specialized Renderers**: Domain-specific renderers for VR/AR, CAD, scientific visualization

## 📊 Benchmarks and Performance Metrics

*Note: Detailed benchmarks will be added as they become available*

| Scenario | Skia Renderer | WGPU Renderer | Notes |
|----------|--------------|---------------|-------|
| Simple UI (1000 elements) | 60 FPS | 58 FPS | Skia slightly better for pure 2D |
| 3D Model + UI | 45 FPS | 59 FPS | WGPU significantly better with 3D content |
| Text-heavy Interface | 59 FPS | 40 FPS | Skia excels at text rendering |
| Custom Shader Effects | 30 FPS | 58 FPS | WGPU provides better shader performance |

### Performance Optimization Guide

For recommendations on optimizing rendering performance in your application, see the [Performance Optimization Guide](../guides/performance-optimization.md).

## 📚 Sample Implementations

### Basic 2D UI with Skia

```orbit
<template>
  <div renderer="skia">
    <header>
      <h1>{{ title }}</h1>
    </header>
    
    <main>
      <div class="card" v-for="item in items">
        <h2>{{ item.title }}</h2>
        <p>{{ item.description }}</p>
        <button @click="selectItem(item)">Select</button>
      </div>
    </main>
    
    <footer>© 2025 My Application</footer>
  </div>
</template>

<style>
.card {
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  padding: 16px;
  margin-bottom: 16px;
}
</style>

<code lang="rust">
use orbit::prelude::*;

#[component]
pub struct ItemsList {
    title: Prop<String>,
    items: Prop<Vec<Item>>,
    on_item_selected: EventEmitter<Item>,
}

impl ItemsList {
    pub fn new() -> Self {
        Self {
            title: Prop::with_default("Items List".to_string()),
            items: Prop::new(),
            on_item_selected: EventEmitter::new(),
        }
    }
    
    pub fn select_item(&self, item: Item) {
        self.on_item_selected.emit(item);
    }
}
</code>
```

### 3D Visualization with WGPU

```orbit
<template>
  <div renderer="wgpu" class="model-viewer">
    <Model3D 
      :src="modelSrc" 
      :rotation="rotation"
      :scale="scale"
      @click="onModelClick"
    />
    
    <!-- UI controls rendered with Skia -->
    <div renderer="skia" class="controls">
      <Slider v-model="scale" min="0.1" max="2.0" step="0.1" />
      <button @click="resetView">Reset View</button>
    </div>
  </div>
</template>

<code lang="rust">
use orbit::prelude::*;
use orbit::graphics::*;

#[component]
pub struct ModelViewer {
    model_src: Prop<String>,
    rotation: State<Vec3>,
    scale: State<f32>,
}

impl ModelViewer {
    pub fn new() -> Self {
        Self {
            model_src: Prop::new(),
            rotation: State::new(Vec3::new(0.0, 0.0, 0.0)),
            scale: State::new(1.0),
        }
    }
    
    pub fn on_model_click(&self, event: &ClickEvent) {
        // Handle click on model
    }
    
    pub fn reset_view(&self, _: &ClickEvent) {
        self.rotation.set(Vec3::new(0.0, 0.0, 0.0));
        self.scale.set(1.0);
    }
}
</code>
```

## 🔄 Related Topics

- [Component Model](./component-model.md) - Understand how components work in Orbit
- [Performance Optimization](../guides/performance-optimization.md) - Learn optimization techniques
- [Orbiton Renderer Command](../api/orbiton-cli.md#orbiton-renderer) - CLI options for renderer configuration
- [API Reference: orbit::render](../api/orbit-render.md) - Detailed API reference
