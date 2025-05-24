# Getting Started with Orbit UI Framework

This guide will help you set up your development environment and create your first Orbit application.

## Prerequisites

- Rust toolchain (1.70.0 or newer) - [rustup.rs](https://rustup.rs/)
- Cargo package manager
- Git

## Installation

First, install the Orbit CLI tool:

```bash
cargo install orbiton
```

This will install the `orbiton` command-line tool, which you'll use to create and manage Orbit projects.

Verify the installation by checking the version:

```bash
orbiton --version
```

<!-- Placeholder for screenshot: Orbiton CLI version output -->

## Creating Your First Orbit Application

Create a new Orbit project:

```bash
orbiton new my-app
cd my-app
```

<!-- Placeholder for screenshot: Output of orbiton new command and directory change -->

This will create a new directory called `my-app` with the following structure:

```
my-app/
├── .gitignore
├── Cargo.toml
├── orbiton.toml
├── src/
│   ├── main.rs
│   └── app.orbit
└── README.md
```

<!-- TODO: Add screenshot of project directory structure after `orbiton new` -->

<!-- Placeholder for screenshot: Basic project structure after creating a new app -->

## Running the Development Server

Start the development server:

```bash
orbiton dev
```

<!-- TODO: Add screenshot of browser at http://localhost:3000 showing the running app -->

<!-- Placeholder for screenshot: Orbiton dev server running and app accessible in browser -->

## Understanding the Project Structure

### `Cargo.toml`

This file is the manifest for Rust's package manager, Cargo. It defines the dependencies, scripts, and metadata for your Orbit project.

### `orbiton.toml`

This is the configuration file for Orbiton, specifying settings and options for your Orbit application.

### `src/main.rs`

The main entry point for your Orbit application, where the Rust code execution begins.

### `src/app.orbit`

This is your first Orbit component.

```html
<template>
  <div class="container">
    <h1>{{ message }}</h1>
    <button @click="greet">Greet</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: "Hello, Orbit!"
    };
  },
  methods: {
    greet() {
      alert("Hello from your Orbit app!");
    }
  }
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  font-family: sans-serif;
}
</style>
```

<!-- TODO: Add screenshot of generated component file in IDE -->

<!-- Placeholder for screenshot: Example of a simple component and its output in the browser -->

## Next Steps

- Learn more about [State Management](../core-concepts/state-management.md)
- Explore [Event Handling](../core-concepts/events.md)
- Check out the [Component API](../api/component-api.md)
- Browse the [Orbit Component Library](../api/orbit-kit-components.md)

## Troubleshooting

### Common Installation Issues

#### Cargo Install Fails

- **Rust Not Found:** Ensure Rust is correctly installed and configured in your system's PATH. You can verify this by running `rustc --version`. If it's not found, visit the [official Rust installation page](https://www.rust-lang.org/tools/install).
- **Orbiton CLI Installation Fails:**
  - Check your internet connection.
  - Ensure `cargo` is working correctly (`cargo --version`).
  - If you encounter compilation errors during `cargo install orbiton_cli`, ensure you have the necessary build tools for your platform (e.g., C++ build tools on Windows, `build-essential` on Debian/Ubuntu).
  - Try updating Rust and Cargo to their latest versions: `rustup update`.
- **`orbiton` command not found after installation:**
  - Ensure that Cargo's bin directory (usually `~/.cargo/bin/` on Linux/macOS or `%USERPROFILE%\.cargo\bin` on Windows) is in your system's PATH. You might need to restart your terminal or shell session for changes to take effect.

### `orbiton new` Fails

- **Permissions:** Ensure you have write permissions in the directory where you're trying to create the new project.
- **Invalid Project Name:** Project names should typically be valid Rust crate names (e.g., no spaces, special characters other than hyphens or underscores).
- **Template Fetching Issues (if applicable in future versions):** If `orbiton new` fetches templates from a remote source, ensure your internet connection is stable.

### `orbiton dev` Issues

- **Port Already in Use:** If the default port (e.g., 3000) is in use, `orbiton dev` might fail to start. You can specify a different port if the CLI supports it (check `orbiton dev --help`).
  - Example (if supported): `orbiton dev --port 3001`
- **Compilation Errors:** The terminal running `orbiton dev` will display any Rust or Orbit compilation errors. Address these errors in your `.orbit` files or Rust code.
- **Websocket Connection Fails (for Hot Module Replacement):**
  - Check your browser's console for errors.
  - Ensure no browser extensions are interfering with WebSocket connections.
  - Firewall or proxy settings might also block WebSocket connections.
- **Changes Not Reflecting (HMR not working):**
  - Verify HMR is enabled (usually by default).
  - Ensure you are editing the correct files within the `src` directory.
  - Some complex changes, especially in `main.rs` or build configurations, might require a manual restart of the dev server.

### Build Errors and Environment Issues

#### Rust Compiler Errors

If the application fails to compile during `orbiton dev` or `orbiton build`:

```
error[E0382]: use of moved value
``` 

**Solution**:
1. Ensure your Rust toolchain is up to date: `rustup update`
2. Verify component code syntax matches the `.orbit` template.
3. Run `cargo clean` before rebuilding.

#### Missing Assets or Static Files

If resources (CSS, images) are not served:

- Check `static/` directory for renamed or missing files.
- Ensure file paths in `index.html` are correct.
- Verify your `orbiton dev` run directory is project root.

### Screenshot Placeholders

<!-- TODO: Add screenshot of project directory structure after `orbiton new` -->

<!-- TODO: Add screenshot of browser at http://localhost:3000 showing the running app -->

<!-- TODO: Add screenshot of generated component file in IDE -->

### General Troubleshooting Steps

- **Check the Orbiton CLI Version:** Ensure you are using a version of Orbiton CLI compatible with your project or the documentation you are following. Use `orbiton --version`.
- **Consult `orbiton --help`:** For any command, running `orbiton <command> --help` (e.g., `orbiton new --help`) can provide useful information about its options and usage.
- **Review `orbiton.toml`:** Your project's `orbiton.toml` file contains configuration that might affect builds and the dev server.
- **Look for `.log` files:** Orbiton or related tools might generate log files with more detailed error information.
- **Community and Official Channels:** If you're stuck, consider seeking help from the Orbit community forums, Discord server, or GitHub issues for Orbiton CLI if available.

### Platform-Specific Issues

#### Windows

**Windows Defender or Antivirus Interference:**
- Some antivirus software may flag the Rust compiler or generated binaries as suspicious.
- Add your project directory and `~/.cargo/` to your antivirus exclusions.
- Windows Defender may quarantine `orbiton.exe` during installation - restore it if necessary.

**Visual Studio Build Tools:**
- Orbiton requires MSVC build tools for Windows.
- Install "C++ build tools" via Visual Studio Installer.
- Alternative: Install via `winget install Microsoft.VisualStudio.2022.BuildTools`.
