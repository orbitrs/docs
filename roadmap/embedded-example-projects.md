# Example Projects for OrbitRS on Embedded Platforms

This document provides example implementation projects for OrbitRS on various embedded platforms.

## 1. RP2040 Weather Station

### Description
A simple weather station application using a Raspberry Pi Pico, SSD1306 OLED display, and BME280 temperature/humidity/pressure sensor.

### Hardware Components
- Raspberry Pi Pico (RP2040)
- SSD1306 128x64 OLED display (I2C or SPI)
- BME280 environmental sensor
- Optional: 2 buttons for UI navigation

### Code Structure

#### `main.rs`
```rust
#![no_std]
#![no_main]

use embedded_hal::digital::v2::InputPin;
use orbitrs_embedded::{prelude::*, display::Ssd1306};
use rp2040_hal::{
    clocks::init_clocks_and_plls,
    gpio::{bank0::Gpio20, bank0::Gpio21, Pin, PushPullOutput},
    pac,
    sio::Sio,
    spi::{self, Spi},
    Clock, Watchdog,
};
use bme280::BME280;
use panic_halt as _;

// Static variables for sensor data
static TEMPERATURE: Signal<f32> = Signal::new(0.0);
static HUMIDITY: Signal<f32> = Signal::new(0.0);
static PRESSURE: Signal<f32> = Signal::new(0.0);
static CURRENT_PAGE: Signal<usize> = Signal::new(0);

#[orbitrs::embedded_component]
fn weather_app() -> impl Component {
    view! {
        <Container>
            {
                match *CURRENT_PAGE.get() {
                    0 => view! {
                        <Label 
                            text={format_static!(32, "Temp: {:.1}°C", *TEMPERATURE.get())}
                        />
                    },
                    1 => view! {
                        <Label 
                            text={format_static!(32, "Humidity: {:.1}%", *HUMIDITY.get())}
                        />
                    },
                    _ => view! {
                        <Label 
                            text={format_static!(32, "Press: {:.0} hPa", *PRESSURE.get())}
                        />
                    }
                }
            }
            <Row>
                <Button 
                    on_press={|| CURRENT_PAGE.update(|p| *p = (*p + 2) % 3)}
                    text="Prev" 
                />
                <Button 
                    on_press={|| CURRENT_PAGE.update(|p| *p = (*p + 1) % 3)}
                    text="Next" 
                />
            </Row>
        </Container>
    }
}

#[rp2040_hal::entry]
fn main() -> ! {
    // Initialize the RP2040
    let mut pac = pac::Peripherals::take().unwrap();
    let core = pac::CorePeripherals::take().unwrap();
    let mut watchdog = Watchdog::new(pac.WATCHDOG);
    let sio = Sio::new(pac.SIO);
    
    // Configure the clocks
    let clocks = init_clocks_and_plls(
        12_000_000,
        pac.XOSC,
        pac.CLOCKS,
        pac.PLL_SYS,
        pac.PLL_USB,
        &mut pac.RESETS,
        &mut watchdog,
    ).ok().unwrap();
    
    // Configure GPIO pins
    let pins = rp2040_hal::gpio::Pins::new(
        pac.IO_BANK0,
        pac.PADS_BANK0,
        sio.gpio_bank0,
        &mut pac.RESETS,
    );
    
    // Configure the SPI for display
    let spi = Spi::<_, _, 8>::new(pac.SPI0);
    let spi = spi.init(
        &mut pac.RESETS,
        clocks.peripheral_clock.freq(),
        1_000_000u32.Hz(),
        &embedded_hal::spi::MODE_0,
    );
    
    // Configure display pins
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
    
    // Configure the I2C for sensor
    let i2c = pac.I2C1;
    let sda_pin = pins.gpio2.into_function::<rp2040_hal::gpio::FunctionI2C>();
    let scl_pin = pins.gpio3.into_function::<rp2040_hal::gpio::FunctionI2C>();
    
    let i2c = rp2040_hal::I2C::i2c1(
        i2c,
        sda_pin,
        scl_pin,
        100.kHz(),
        &mut pac.RESETS,
        &clocks.peripheral_clock,
    );
    
    // Initialize BME280 sensor
    let mut bme280 = BME280::new_primary(i2c);
    bme280.init().unwrap();
    
    // Create the OrbitRS application
    let mut app = orbitrs_embedded::OrbitApp::new(
        renderer,
        weather_app,
        (128, 64),
    ).unwrap();
    
    // Configure button inputs
    let button_prev = pins.gpio16.into_pull_up_input();
    let button_next = pins.gpio17.into_pull_up_input();
    
    // Create button handler
    let mut button_handler = orbitrs_embedded::input::GpioButtonHandler::new(&mut [
        (ButtonId::Left, button_prev),
        (ButtonId::Right, button_next),
    ]);
    
    app.add_input_handler(button_handler).unwrap();
    
    // Sample delay
    let mut delay = cortex_m::delay::Delay::new(core.SYST, clocks.system_clock.freq().to_Hz());
    
    // Main loop
    let mut last_sample = 0;
    loop {
        // Update the application
        app.update().unwrap();
        
        // Read sensor data every second
        if last_sample > 100 {
            // Read sensor
            let measurements = bme280.measure().unwrap();
            
            // Update signal values
            TEMPERATURE.set(measurements.temperature);
            HUMIDITY.set(measurements.humidity);
            PRESSURE.set(measurements.pressure / 100.0); // Convert to hPa
            
            last_sample = 0;
            
            // Force redraw
            app.invalidate();
        }
        
        last_sample += 1;
        delay.delay_ms(10);
    }
}
```

#### `Cargo.toml`
```toml
[package]
name = "orbitrs-weather-station"
version = "0.1.0"
edition = "2021"

[dependencies]
orbitrs-core = { version = "0.1.0", default-features = false }
orbitrs-renderer = { version = "0.1.0", default-features = false }
orbitrs-components = { version = "0.1.0", default-features = false }
orbitrs-embedded = { version = "0.1.0", default-features = false }

# Hardware support
rp2040-hal = "0.8.0"
embedded-hal = "0.2.7"
embedded-graphics = "0.7.1"

# Display driver
display-interface = "0.4.1"
display-interface-spi = "0.4.1"
ssd1306 = "0.7.1"

# Sensor
bme280 = "0.4.3"

# RP2040 dependencies
cortex-m = "0.7.7"
cortex-m-rt = "0.7.2"
panic-halt = "0.2.0"

[features]
default = []

[profile.release]
opt-level = "s"
lto = true
codegen-units = 1
debug = true
```

## 2. ESP32 Smart Home Control Panel

### Description
A touchscreen-based smart home control panel using an ESP32 and ILI9341 TFT display with touch capability.

### Hardware Components
- ESP32 development board
- ILI9341 320x240 TFT display with XPT2046 touch controller
- Optional: DHT22 temperature/humidity sensor for room monitoring

### Code Structure

#### `main.rs`
```rust
#![no_std]
#![no_main]

use esp32_hal::{
    clock::ClockControl,
    gpio::IO,
    peripherals::Peripherals,
    prelude::*,
    spi::{Spi, SpiMode},
    timer::TimerGroup,
    Rtc,
};
use orbitrs_embedded::{prelude::*, display::Ili9341};
use panic_halt as _;

// Smart home device states
static LIVING_ROOM_LIGHT: Signal<bool> = Signal::new(false);
static KITCHEN_LIGHT: Signal<bool> = Signal::new(false);
static BEDROOM_LIGHT: Signal<bool> = Signal::new(false);
static THERMOSTAT_TEMP: Signal<f32> = Signal::new(21.0);
static CURRENT_TAB: Signal<usize> = Signal::new(0);

#[orbitrs::embedded_component]
fn home_control_app() -> impl Component {
    view! {
        <Container>
            <TabBar
                selected={*CURRENT_TAB.get()}
                on_change={|idx| CURRENT_TAB.set(idx)}
                tabs={["Lights", "Climate", "Status"]}
            />
            
            {
                match *CURRENT_TAB.get() {
                    0 => view! {
                        <Card title="Lighting Control">
                            <Row>
                                <Label text="Living Room" />
                                <Switch 
                                    checked={*LIVING_ROOM_LIGHT.get()}
                                    on_change={|v| LIVING_ROOM_LIGHT.set(v)}
                                />
                            </Row>
                            <Row>
                                <Label text="Kitchen" />
                                <Switch 
                                    checked={*KITCHEN_LIGHT.get()}
                                    on_change={|v| KITCHEN_LIGHT.set(v)}
                                />
                            </Row>
                            <Row>
                                <Label text="Bedroom" />
                                <Switch 
                                    checked={*BEDROOM_LIGHT.get()}
                                    on_change={|v| BEDROOM_LIGHT.set(v)}
                                />
                            </Row>
                        </Card>
                    },
                    1 => view! {
                        <Card title="Climate Control">
                            <Row>
                                <Label text={format_static!(32, "Temperature: {:.1}°C", *THERMOSTAT_TEMP.get())} />
                            </Row>
                            <Row>
                                <Button 
                                    text="-"
                                    on_press={|| THERMOSTAT_TEMP.update(|t| *t -= 0.5)}
                                />
                                <ProgressBar 
                                    value={((*THERMOSTAT_TEMP.get() - 15.0) / 15.0 * 100.0) as u8}
                                />
                                <Button 
                                    text="+"
                                    on_press={|| THERMOSTAT_TEMP.update(|t| *t += 0.5)}
                                />
                            </Row>
                        </Card>
                    },
                    _ => view! {
                        <Card title="System Status">
                            <Label text="All systems operational" />
                            <ProgressBar value={100} />
                        </Card>
                    }
                }
            }
        </Container>
    }
}

#[esp32_hal::entry]
fn main() -> ! {
    // Initialize ESP32
    let peripherals = Peripherals::take().unwrap();
    let system = peripherals.SYSTEM.split();
    let clocks = ClockControl::boot_defaults(system.clock_control).freeze();
    
    // Configure GPIO
    let io = IO::new(peripherals.GPIO, peripherals.IO_MUX);
    
    // Configure SPI for display
    let spi = Spi::new(
        peripherals.SPI2,
        io.pins.gpio19, // SCK
        io.pins.gpio23, // MOSI
        io.pins.gpio25, // MISO
        io.pins.gpio22, // CS
        60u32.MHz(),
        SpiMode::Mode0,
        &clocks,
    );
    
    // Configure display pins
    let dc = io.pins.gpio21.into_push_pull_output();
    let rst = io.pins.gpio18.into_push_pull_output();
    
    // Initialize display
    let display_interface = display_interface_spi::SPIInterface::new(spi, dc, rst);
    let display = ili9341::Ili9341::new(
        display_interface,
        ili9341::Orientation::Portrait,
    ).into();
    
    // Create embedded renderer
    let renderer = orbitrs_embedded::renderer::Ili9341Renderer::new(display);
    
    // Configure touch controller
    let touch_spi = Spi::new(
        peripherals.SPI3,
        io.pins.gpio14, // SCK
        io.pins.gpio13, // MOSI
        io.pins.gpio12, // MISO
        io.pins.gpio15, // CS
        2u32.MHz(),
        SpiMode::Mode0,
        &clocks,
    );
    
    let touch_irq = io.pins.gpio27.into_pull_up_input();
    
    // Initialize touch controller
    let mut touch_controller = xpt2046::XPT2046::new(
        touch_spi,
        io.pins.gpio15.into_push_pull_output(),
    );
    
    // Create touch handler
    let touch_handler = orbitrs_embedded::input::TouchHandler::new(
        touch_controller,
        touch_irq,
        |x, y| (320 - y as u32, x as u32), // Coordinate transformation
    );
    
    // Create the OrbitRS application
    let mut app = orbitrs_embedded::OrbitApp::new(
        renderer,
        home_control_app,
        (320, 240),
    ).unwrap();
    
    app.add_input_handler(touch_handler).unwrap();
    
    // Create timer for periodic tasks
    let mut timer0 = TimerGroup::new(peripherals.TIMG0, &clocks).timer0;
    timer0.start(1u64.MHz());
    
    // Main loop
    loop {
        // Update the application
        app.update().unwrap();
        
        // Handle WiFi/MQTT communication periodically
        if timer0.wait() {
            // Placeholder for smart home communication
            // This would involve sending MQTT messages based on UI state
        }
    }
}
```

#### `Cargo.toml`
```toml
[package]
name = "orbitrs-smarthome-panel"
version = "0.1.0"
edition = "2021"

[dependencies]
orbitrs-core = { version = "0.1.0", default-features = false }
orbitrs-renderer = { version = "0.1.0", default-features = false }
orbitrs-components = { version = "0.1.0", default-features = false }
orbitrs-embedded = { version = "0.1.0", default-features = false, features = ["touch"] }

# Hardware support
esp32-hal = "0.11.0"
embedded-hal = "0.2.7"
embedded-graphics = "0.7.1"

# Display driver
display-interface = "0.4.1"
display-interface-spi = "0.4.1"
ili9341 = { version = "0.5.0", default-features = false }

# Touch controller
xpt2046 = "0.3.0"

# ESP32 dependencies
panic-halt = "0.2.0"
esp-println = { version = "0.5.0", features = ["esp32"] }

[profile.release]
opt-level = "s"
lto = true
```

## 3. STM32F4 Fitness Tracker

### Description
A wearable fitness tracker application using an STM32F4 Discovery board with accelerometer for step counting.

### Hardware Components
- STM32F4 Discovery board
- SSD1306 OLED display
- LIS3DSH accelerometer (built into the STM32F4 Discovery)

### Code Structure

#### `main.rs`
```rust
#![no_std]
#![no_main]

use cortex_m::peripheral::NVIC;
use stm32f4xx_hal::{
    gpio::{gpioa, Output, PushPull, EPin},
    i2c::I2c,
    pac,
    prelude::*,
    spi::{Spi, Mode},
};
use orbitrs_embedded::{prelude::*, display::Ssd1306};
use lis3dsh::{Lis3dsh, Scale, DataRate};
use rtic::app;
use panic_halt as _;

// Application state
static STEP_COUNT: Signal<u32> = Signal::new(0);
static CALORIES: Signal<u32> = Signal::new(0);
static ACTIVE_MINUTES: Signal<u32> = Signal::new(0);
static CURRENT_VIEW: Signal<u8> = Signal::new(0);

#[orbitrs::embedded_component]
fn fitness_app() -> impl Component {
    view! {
        <Container>
            <Row>
                <Button 
                    text="Steps"
                    on_press={|| CURRENT_VIEW.set(0)}
                    highlighted={*CURRENT_VIEW.get() == 0}
                />
                <Button 
                    text="Cals"
                    on_press={|| CURRENT_VIEW.set(1)}
                    highlighted={*CURRENT_VIEW.get() == 1}
                />
                <Button 
                    text="Active"
                    on_press={|| CURRENT_VIEW.set(2)}
                    highlighted={*CURRENT_VIEW.get() == 2}
                />
            </Row>
            
            {
                match *CURRENT_VIEW.get() {
                    0 => view! {
                        <Container>
                            <LargeText text={format_static!(16, "{}", *STEP_COUNT.get())} />
                            <Label text="steps today" />
                            <ProgressBar value={((*STEP_COUNT.get() as f32 / 10000.0) * 100.0) as u8} />
                        </Container>
                    },
                    1 => view! {
                        <Container>
                            <LargeText text={format_static!(16, "{}", *CALORIES.get())} />
                            <Label text="calories burned" />
                            <ProgressBar value={((*CALORIES.get() as f32 / 2000.0) * 100.0) as u8} />
                        </Container>
                    },
                    _ => view! {
                        <Container>
                            <LargeText text={format_static!(16, "{}", *ACTIVE_MINUTES.get())} />
                            <Label text="active minutes" />
                            <ProgressBar value={((*ACTIVE_MINUTES.get() as f32 / 30.0) * 100.0) as u8} />
                        </Container>
                    }
                }
            }
        </Container>
    }
}

#[app(device = stm32f4xx_hal::pac)]
mod app {
    use super::*;
    
    #[shared]
    struct Shared {
        accelerometer: Lis3dsh<Spi<pac::SPI1>>,
    }
    
    #[local]
    struct Local {
        app: OrbitApp<Ssd1306Renderer<I2c<pac::I2C1>>>,
        led: EPin<Output<PushPull>>,
        last_reading: (f32, f32, f32),
        activity_level: u8,
    }
    
    #[init]
    fn init(ctx: init::Context) -> (Shared, Local, init::Monotonics) {
        // Get access to device peripherals
        let rcc = ctx.device.RCC.constrain();
        let clocks = rcc.cfgr.use_hse(8.MHz()).sysclk(48.MHz()).freeze();
        
        // Configure GPIO
        let gpioe = ctx.device.GPIOE.split();
        let led = gpioe.pe3.into_push_pull_output().erase();
        
        let gpioa = ctx.device.GPIOA.split();
        let gpioe = ctx.device.GPIOE.split();
        
        // Setup accelerometer on SPI1
        let sck = gpioa.pa5.into_alternate();
        let miso = gpioa.pa6.into_alternate();
        let mosi = gpioa.pa7.into_alternate();
        let cs = gpioe.pe3.into_push_pull_output();
        
        let spi = Spi::new(
            ctx.device.SPI1,
            (sck, miso, mosi),
            Mode {
                polarity: stm32f4xx_hal::spi::Polarity::IdleLow,
                phase: stm32f4xx_hal::spi::Phase::CaptureOnFirstTransition,
            },
            1.MHz(),
            &clocks,
        );
        
        // Configure accelerometer
        let mut accelerometer = Lis3dsh::new(spi, cs).unwrap();
        accelerometer.set_scale(Scale::G2).unwrap();
        accelerometer.set_datarate(DataRate::Hz400).unwrap();
        
        // Configure I2C for OLED display
        let scl = gpiob.pb6.into_alternate_open_drain();
        let sda = gpiob.pb7.into_alternate_open_drain();
        
        let i2c = I2c::new(ctx.device.I2C1, (scl, sda), 400.kHz(), &clocks);
        
        // Initialize display
        let interface = display_interface_i2c::I2CInterface::new(i2c);
        let display = ssd1306::Ssd1306::new(
            interface,
            ssd1306::size::DisplaySize128x32,
            ssd1306::rotation::DisplayRotation::Rotate0,
        ).into();
        
        // Create renderer
        let renderer = orbitrs_embedded::renderer::Ssd1306Renderer::new(display);
        
        // Create app
        let mut app = orbitrs_embedded::OrbitApp::new(
            renderer,
            fitness_app,
            (128, 32),
        ).unwrap();
        
        // Configure buttons for input
        let button_a = gpioa.pa0.into_pull_up_input();
        let button_b = gpioa.pa1.into_pull_up_input();
        let button_c = gpioa.pa2.into_pull_up_input();
        
        // Create button handler
        let button_handler = orbitrs_embedded::input::GpioButtonHandler::new(&[
            (ButtonId::Custom(0), button_a),
            (ButtonId::Custom(1), button_b),
            (ButtonId::Custom(2), button_c),
        ]);
        
        app.add_input_handler(button_handler).unwrap();
        
        // Configure timer interrupt
        let mut timer = ctx.device.TIM2.counter(&clocks);
        timer.start(100.millis()).unwrap();
        timer.listen(stm32f4xx_hal::timer::Event::Update);
        
        unsafe {
            NVIC::unmask(pac::Interrupt::TIM2);
        }
        
        (
            Shared { accelerometer },
            Local { 
                app,
                led,
                last_reading: (0.0, 0.0, 0.0),
                activity_level: 0,
            },
            init::Monotonics()
        )
    }
    
    #[idle]
    fn idle(_: idle::Context) -> ! {
        loop {
            cortex_m::asm::wfi();
        }
    }
    
    #[task(binds = TIM2, priority = 1, shared = [accelerometer], local = [app, led, last_reading, activity_level])]
    fn timer_tick(mut ctx: timer_tick::Context) {
        // Read accelerometer
        let accel_data = ctx.shared.accelerometer.lock(|accel| {
            accel.read_accel().unwrap()
        });
        
        // Calculate acceleration magnitude for step detection
        let (x, y, z) = accel_data;
        let magnitude = (x*x + y*y + z*z).sqrt();
        let (last_x, last_y, last_z) = *ctx.local.last_reading;
        let last_magnitude = (last_x*last_x + last_y*last_y + last_z*last_z).sqrt();
        
        // Simple step detection algorithm
        let delta = (magnitude - last_magnitude).abs();
        if delta > 0.5 && *ctx.local.activity_level < 255 {
            *ctx.local.activity_level += 1;
        } else if *ctx.local.activity_level > 0 {
            *ctx.local.activity_level -= 1;
        }
        
        // Detect steps based on activity level
        if *ctx.local.activity_level > 10 {
            STEP_COUNT.update(|s| *s += 1);
            ctx.local.led.set_high();
        } else {
            ctx.local.led.set_low();
        }
        
        // Update calorie estimation (very rough estimate)
        CALORIES.set((*STEP_COUNT.get() * 4) / 100);
        
        // Update active minutes (when activity level is high)
        if *ctx.local.activity_level > 50 {
            ACTIVE_MINUTES.update(|m| {
                if *m < 1440 { // Max minutes in a day
                    *m += 1;
                }
            });
        }
        
        // Update UI
        ctx.local.app.update().unwrap();
        
        // Store current reading
        *ctx.local.last_reading = (x, y, z);
    }
}
```

#### `Cargo.toml`
```toml
[package]
name = "orbitrs-fitness-tracker"
version = "0.1.0"
edition = "2021"

[dependencies]
orbitrs-core = { version = "0.1.0", default-features = false }
orbitrs-renderer = { version = "0.1.0", default-features = false }
orbitrs-components = { version = "0.1.0", default-features = false }
orbitrs-embedded = { version = "0.1.0", default-features = false }

# Hardware support
stm32f4xx-hal = { version = "0.14.0", features = ["stm32f407"] }
embedded-hal = "0.2.7"
embedded-graphics = "0.7.1"

# Display driver
display-interface = "0.4.1"
display-interface-i2c = "0.4.1"
ssd1306 = "0.7.1"

# Accelerometer driver
lis3dsh = "0.2.0"

# RTIC framework
cortex-m = "0.7.7"
cortex-m-rt = "0.7.2"
panic-halt = "0.2.0"
rtic = { version = "1.0.0", features = ["thumbv7-backend"] }

[profile.release]
opt-level = "s"
lto = true
debug = true
```

## Integration Considerations

All of these examples showcase different approaches to integrating OrbitRS with embedded platforms:

1. **RP2040 Example**: Shows basic integration with a simple microcontroller and minimal peripherals.
2. **ESP32 Example**: Demonstrates more advanced features like touchscreen support and wireless connectivity.
3. **STM32F4 Example**: Illustrates integration with RTIC for a real-time application with sensor processing.

Key implementation patterns demonstrated:

- Static allocation of UI component state
- Signal-based reactive state management
- Integration with various display technologies
- Hardware-specific input handling
- Sensor data processing and visualization

Each example uses the same OrbitRS programming model but adapts it to the specific constraints and capabilities of the target hardware platform.
