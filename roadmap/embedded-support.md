# OrbitRS Embedded Support Roadmap

> Document Version: 1.0  
> Created: May 22, 2025  
> Status: Planning  

## Overview

This document outlines the architectural changes and implementation strategy required to extend OrbitRS to support embedded systems, including microcontrollers like the Raspberry Pi Pico (RP2040) and other similar platforms. The goal is to maintain compatibility with existing desktop and web platforms while adding support for resource-constrained embedded environments.

## Goals

1. Enable OrbitRS applications to run on microcontrollers (MCUs)
2. Maintain a unified programming model across all platforms
3. Support common embedded displays and input mechanisms
4. Optimize memory usage for constrained environments
5. Provide a path for developers to create cross-platform applications

## Target Platforms

Initial embedded support will focus on:

- Raspberry Pi Pico (RP2040)
- ESP32 family
- STM32 family
- Other Cortex-M based microcontrollers

## Architecture Redesign

### 1. Tiered Architecture

The framework will be restructured into a tiered architecture:

```
orbitrs/
├── orbitrs-core/         # no_std compatible core (tier 1)
├── orbitrs-renderer/     # Rendering abstraction layer (tier 1)
├── orbitrs-components/   # Basic component primitives (tier 1)
├── orbitrs-std/          # Standard library features (tier 2)
├── orbitrs-embedded/     # Embedded platform support (tier 2)
├── orbitrs-web/          # Web platform support (tier 2) 
├── orbitrs-desktop/      # Desktop platform support (tier 2)
└── orbitrs/              # Meta-package that re-exports all components
```

**Tier 1 (Core)**: No-std compatible, minimal dependencies, fundamental abstractions
**Tier 2 (Platform)**: Platform-specific implementations built on top of the core

### 2. Core Component Model

A simplified component model for embedded targets that preserves the programming model:

```rust
/// Core trait for all components
pub trait Component {
    /// Render the component to a target surface
    fn render(&self, renderer: &mut dyn Renderer);
    
    /// Update the component state based on input events
    fn handle_event(&mut self, event: &Event) -> bool;
    
    /// Get the dimensions of this component
    fn dimensions(&self) -> Dimensions;
}

/// Memory-efficient signal system for state management
pub struct Signal<T: 'static> {
    value: T,
    dependents: StaticVec<*mut (), 8>, // Fixed capacity for embedded targets
}
```

### 3. Renderer Abstraction

A modular rendering system with platform-specific backends:

```rust
/// Core rendering trait
pub trait Renderer {
    /// Initialize the renderer
    fn initialize(&mut self) -> Result<(), Error>;
    
    /// Create a drawing surface
    fn create_surface(&mut self, width: u32, height: u32) -> Surface;
    
    /// Draw basic primitives
    fn draw_rect(&mut self, surface: &mut Surface, rect: Rect, color: Color);
    fn draw_text(&mut self, surface: &mut Surface, text: &str, position: Point, font: &Font);
    fn draw_image(&mut self, surface: &mut Surface, image: &Image, position: Point);
    
    /// Present the surface to the display
    fn present(&mut self, surface: &Surface);
}
```

## Implementation Plan

### Phase 1: Core Architecture (3-4 months)

1. **Core Library Refactoring**
   - Split the codebase into the tiered architecture described above
   - Create `#[no_std]` compatible core crates
   - Implement memory-constrained versions of core data structures

2. **Renderer Abstraction**
   - Define a platform-agnostic rendering API
   - Create trait-based abstractions for rendering operations
   - Implement current renderer as a concrete backend

3. **Component Model Simplification**
   - Develop lightweight component abstractions
   - Implement static allocation strategies for UI trees
   - Create compile-time component definition macros

### Phase 2: Embedded Platform Support (2-3 months)

1. **Embedded Renderer Implementation**
   - Integrate with `embedded-graphics` ecosystem
   - Support common display protocols (SPI, I2C, parallel)
   - Implement optimized rendering paths for resource constraints

2. **Input System**
   - Implement GPIO-based input handling
   - Support common embedded input devices (buttons, encoders, touch interfaces)
   - Map hardware inputs to the OrbitRS event system

3. **Memory Management**
   - Implement pre-allocated memory pools for UI components
   - Develop compile-time UI tree generation
   - Create static state management primitives

### Phase 3: Platform-Specific Implementations (2-3 months per platform)

1. **RP2040 Support**
   - Integrate with rp-hal ecosystem
   - Implement RP2040-specific optimizations
   - Create comprehensive examples for common peripherals

2. **ESP32 Support**
   - Integrate with esp-rs ecosystem
   - Implement ESP32-specific optimizations (potentially leverage hardware acceleration)
   - Support ESP32's WiFi capabilities for networked applications

3. **STM32 Support**
   - Integrate with stm32-rs ecosystem
   - Implement STM32-specific optimizations
   - Support various STM32 families (F4, H7, etc.)

## Technical Challenges & Solutions

### 1. Memory Constraints

**Challenges:**
- Limited RAM (as low as 264KB on RP2040)
- No dynamic memory allocation in some environments
- Limited stack space

**Solutions:**
- Static allocation for UI component trees
- Pre-allocated memory pools
- Compile-time UI generation
- Incremental rendering for complex UIs

```rust
// Example of compile-time UI generation
#[orbitrs::embedded_component]
fn counter() -> impl Component {
    static COUNT: Signal<u32> = Signal::new(0);
    
    view! {
        <Container>
            <Label text={format_static!("Count: {}", COUNT.get())} />
            <Button on_press={|| COUNT.update(|c| *c += 1)} text="+" />
        </Container>
    }
}
```

### 2. Display Limitations

**Challenges:**
- Limited resolution and color depth
- Slow refresh rates
- Diverse display interfaces

**Solutions:**
- Simplified rendering primitives
- Display-specific optimizations
- Incremental display updates
- Hardware-accelerated paths where available

```rust
// Example embedded renderer for SSD1306 OLED display
pub struct Ssd1306Renderer<D: DisplayInterface> {
    display: ssd1306::Ssd1306<D>,
    buffer: [u8; SSD1306_BUFFER_SIZE],
}

impl<D: DisplayInterface> Renderer for Ssd1306Renderer<D> {
    // Implementation details...
}
```

### 3. Cross-Platform Compatibility

**Challenges:**
- Maintaining unified programming model
- Supporting different hardware capabilities
- Balancing abstraction with performance

**Solutions:**
- Feature-based conditional compilation
- Progressive enhancement pattern
- Platform capability detection
- Shared core abstractions

```rust
// Example of conditional compilation
#[cfg(feature = "embedded")]
pub use orbitrs_embedded::platform::*;

#[cfg(feature = "desktop")]
pub use orbitrs_desktop::platform::*;

#[cfg(feature = "web")]
pub use orbitrs_web::platform::*;
```

## Example Usage

### Minimal Counter Application for RP2040

```rust
#![no_std]
#![no_main]

use embedded_hal::digital::v2::OutputPin;
use orbitrs_embedded::{prelude::*, display::Ssd1306};
use rp2040_hal::{pac, sio::Sio, Clock, Watchdog};
use panic_halt as _;

#[orbitrs::embedded_component]
fn counter_app() -> impl Component {
    // Static allocation for state
    static COUNT: Signal<i32> = Signal::new(0);
    
    view! {
        <Container>
            <Label text={format_static!("Count: {}", COUNT.get())} />
            <Row>
                <Button 
                    on_press={|| COUNT.update(|c| *c -= 1)}
                    text="-" 
                />
                <Button 
                    on_press={|| COUNT.update(|c| *c += 1)}
                    text="+" 
                />
            </Row>
        </Container>
    }
}

#[rp2040_hal::entry]
fn main() -> ! {
    // Initialize RP2040 peripherals
    let mut pac = pac::Peripherals::take().unwrap();
    let core = pac::CorePeripherals::take().unwrap();
    let mut watchdog = Watchdog::new(pac.WATCHDOG);
    let sio = Sio::new(pac.SIO);
    
    // Configure clocks
    let clocks = rp2040_hal::clocks::init_clocks_and_plls(
        12_000_000,
        pac.XOSC,
        pac.CLOCKS,
        pac.PLL_SYS,
        pac.PLL_USB,
        &mut pac.RESETS,
        &mut watchdog,
    ).ok().unwrap();
    
    // Configure pins for SPI
    let pins = rp2040_hal::gpio::Pins::new(
        pac.IO_BANK0,
        pac.PADS_BANK0,
        sio.gpio_bank0,
        &mut pac.RESETS,
    );
    
    // Set up SPI for display
    let spi = rp2040_hal::Spi::<_, _, 8>::new(pac.SPI0);
    let spi = spi.init(
        &mut pac.RESETS,
        clocks.peripheral_clock.freq(),
        1_000_000u32.Hz(),
        &embedded_hal::spi::MODE_0,
    );
    
    // Set up display
    let dc = pins.gpio8.into_push_pull_output();
    let cs = pins.gpio9.into_push_pull_output();
    let reset = pins.gpio12.into_push_pull_output();
    
    // Initialize display
    let display_interface = display_interface_spi::SPIInterface::new(spi, dc, cs);
    let display = ssd1306::Ssd1306::new(
        display_interface,
        ssd1306::size::DisplaySize128x64,
        ssd1306::rotation::DisplayRotation::Rotate0,
    ).into();
    
    // Create embedded renderer
    let renderer = orbitrs_embedded::renderer::Ssd1306Renderer::new(display);
    
    // Create application with embedded backend
    let mut app = orbitrs_embedded::OrbitApp::new(counter_app, renderer);
    
    // Input pins for buttons
    let button_plus = pins.gpio16.into_pull_up_input();
    let button_minus = pins.gpio17.into_pull_up_input();
    
    // Main loop
    let mut last_plus = true;
    let mut last_minus = true;
    
    loop {
        // Poll button inputs
        let current_plus = button_plus.is_high().unwrap();
        let current_minus = button_minus.is_high().unwrap();
        
        // Handle button press events
        if last_plus && !current_plus {
            app.dispatch_event(Event::ButtonPress(Button::Plus));
        }
        
        if last_minus && !current_minus {
            app.dispatch_event(Event::ButtonPress(Button::Minus));
        }
        
        last_plus = current_plus;
        last_minus = current_minus;
        
        // Update and render the application
        app.update();
        app.render();
        
        // Optional: add delay to reduce power consumption
        cortex_m::asm::delay(clocks.system_clock.freq().0 / 100);
    }
}
```

## Build System Integration

### Cargo Features

```toml
# Example Cargo.toml
[dependencies]
orbitrs-core = { version = "0.1.0", default-features = false }
orbitrs-renderer = { version = "0.1.0", default-features = false }
orbitrs-components = { version = "0.1.0", default-features = false }

# Conditional platform dependencies
orbitrs-embedded = { version = "0.1.0", optional = true }
orbitrs-desktop = { version = "0.1.0", optional = true }
orbitrs-web = { version = "0.1.0", optional = true }

[features]
default = ["std"]
std = ["orbitrs-desktop"]
web = ["orbitrs-web"]
embedded = ["orbitrs-embedded"]
rp2040 = ["embedded", "dep:rp2040-hal"]
esp32 = ["embedded", "dep:esp32-hal"]
stm32f4 = ["embedded", "dep:stm32f4xx-hal"]
```

### CI Integration

The CI pipeline will need to be extended to support building and testing for embedded targets:

```yaml
# New embedded testing job
embedded-test:
  name: Test Embedded Targets
  runs-on: ubuntu-latest
  needs: check
  steps:
    - uses: actions/checkout@v4
    
    - name: Install Rust toolchain
      uses: dtolnay/rust-toolchain@stable
      with:
        targets: thumbv6m-none-eabi
    
    - name: Install cargo-embed
      uses: taiki-e/install-action@v2
      with:
        tool: cargo-embed
    
    - name: Build for RP2040
      run: cargo build --no-default-features --features rp2040 --target thumbv6m-none-eabi
    
    # Optional: run tests in emulation
    - name: Run embedded tests in emulation
      run: cargo test --no-default-features --features rp2040 --target thumbv6m-none-eabi
```

## Release Strategy

The embedded support will be released in phases:

1. **Alpha Release**: Initial architecture with RP2040 support
2. **Beta Release**: Support for multiple platforms with basic UI components
3. **Stable Release**: Full embedded platform support with production-ready examples

Each release will include:
- Documentation for specific platforms
- Example applications targeting common hardware
- Pre-built binaries for popular development boards
- Integration guides for common embedded ecosystems

## Documentation Strategy

Embedded support documentation will include:

1. **Platform Guides**
   - Hardware setup instructions
   - Display configuration
   - Input device integration
   - Memory optimization techniques

2. **Porting Guides**
   - How to adapt existing OrbitRS applications for embedded targets
   - Progressive enhancement strategies
   - Feature detection and capability-based rendering

3. **Reference Projects**
   - Complete examples for different hardware configurations
   - Benchmarks and performance metrics
   - Memory usage analysis

## Conclusion

Extending OrbitRS to support embedded systems requires significant architectural changes but offers the exciting possibility of using the same framework across the entire spectrum of computing devices - from tiny microcontrollers to desktop applications and web interfaces.

This document outlines a comprehensive approach to implementing embedded support while maintaining compatibility with existing platforms. The development will proceed in phases, focusing first on core architectural changes followed by platform-specific implementations.

## References

- [Embedded Rust Book](https://docs.rust-embedded.org/book/)
- [rp-hal Documentation](https://docs.rs/rp-hal/latest/rp_hal/)
- [embedded-graphics Documentation](https://docs.rs/embedded-graphics/latest/embedded_graphics/)
- [no_std Rust Guide](https://docs.rust-embedded.org/book/intro/no-std.html)
