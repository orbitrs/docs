# WGPU Rendering System

## Overview

The WGPU rendering system in the Orbit UI Framework provides high-performance 3D graphics capabilities using the WebGPU API through the WGPU Rust implementation. This document outlines the key components of the rendering system and how to use them in your applications.

## Key Components

### WgpuRenderer

The `WgpuRenderer` is the core of the 3D rendering system, implementing the `Renderer` trait. It handles:

- GPU device and queue management
- Surface configuration
- Command encoding and submission
- Frame rendering

### Shader System

The shader system provides tools for managing and using WGSL and SPIR-V shaders:

- `Shader`: Represents a compiled shader module with an entry point
- `ShaderSet`: A collection of shaders for different pipeline stages
- `ShaderManager`: Handles shader compilation, caching, and provides common shaders

### Mesh System

The mesh system allows you to create and manipulate 3D geometry:

- `Vertex`: A 3D vertex with position and color attributes
- `Mesh`: A 3D mesh with vertex and index buffers
- `MeshPrimitives`: Common 3D primitives like cubes, spheres, and planes

### Camera System

The camera system provides 3D camera functionality:

- `Camera`: Defines a 3D camera with perspective projection
- `CameraController`: Interactive camera controller for movement and rotation
- `CameraUniform`: Uniform buffer for sending camera data to shaders

## Usage

### Creating a WGPU Renderer

```rust
use orbit::renderer::{create_renderer, RendererType};

// Create a WGPU renderer
let mut renderer = create_renderer(RendererType::Wgpu)?;
```

### Setting Up a 3D Scene

```rust
// Create a camera
let camera = Camera::new(
    cgmath::Point3::new(0.0, 1.5, 5.0), // position
    cgmath::Point3::new(0.0, 0.0, 0.0), // target
    cgmath::Vector3::unit_y(),          // up vector
    16.0 / 9.0,                         // aspect ratio
    45.0,                               // field of view in degrees
    0.1,                                // near clipping plane
    100.0,                              // far clipping plane
);

// Create basic primitives
let cube = MeshPrimitives::cube(&device, 1.0)?;
let sphere = MeshPrimitives::sphere(&device, 1.0, 32)?;
let plane = MeshPrimitives::plane(&device, 10.0)?;
```

### Creating Shaders

```rust
let mut shader_manager = ShaderManager::new(device.clone());

// Create a basic shader set
let shader_set = shader_manager.create_basic_3d_shader_set()?;

// Or load custom shaders
let vertex_shader = shader_manager.load_wgsl(vertex_shader_source, "Custom Vertex", "vs_main")?;
let fragment_shader = shader_manager.load_wgsl(fragment_shader_source, "Custom Fragment", "fs_main")?;
```

### Handling Camera Movement

```rust
// Create a camera controller
let mut camera_controller = CameraController::new(camera, 3.0);

// Process keyboard input
camera_controller.process_keyboard("w", true); // Move forward
camera_controller.process_keyboard("a", true); // Move left

// Process mouse movement
camera_controller.process_mouse(delta_x, delta_y);

// Update camera
let dt = 0.016; // time delta in seconds
camera_controller.update(dt);
```

## Integration with Component System

The WGPU renderer integrates with the Orbit component system through the `Renderer` trait:

```rust
impl Renderer for WgpuRenderer {
    fn render(&mut self, root: &Node) -> Result<(), Error> {
        // Rendering implementation
    }
    
    fn name(&self) -> &str {
        "WGPU Renderer"
    }
}
```

Components can specify 3D content through node attributes:

```rust
fn render(&self) -> Result<Vec<Node>, ComponentError> {
    let mut node = Node::new(None);
    node.add_attribute("renderer".to_string(), "wgpu".to_string());
    node.add_attribute("type".to_string(), "3d".to_string());
    // Add more 3D-specific attributes
    Ok(vec![node])
}
```

## Composite Rendering

For UIs that need both 2D and 3D elements, the `CompositeRenderer` allows seamless integration:

```rust
// Create a composite renderer
let composite = CompositeRenderer::default()?;

// Or create with specific renderers
let composite = CompositeRenderer::new(
    create_renderer(RendererType::Skia)?,
    create_renderer(RendererType::Wgpu)?,
);
```

The composite renderer will automatically use the appropriate renderer for different parts of the UI.

## Performance Considerations

- Use appropriate LOD (Level of Detail) for meshes
- Minimize shader compilation during runtime
- Use instancing for repeated geometry
- Consider offscreen rendering for complex scenes
- Implement frustum culling for large scenes

## Best Practices

1. Initialize the WGPU renderer on a background thread
2. Pre-compile shaders at application startup
3. Use the `MeshPrimitives` for common shapes
4. Implement camera controls with smooth motion
5. Use the `CompositeRenderer` for mixed 2D/3D UIs
