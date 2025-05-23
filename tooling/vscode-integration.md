---
title: VS Code Integration
description: How to integrate Orlint with Visual Studio Code for an enhanced Orbit development experience.
---

# Orlint VS Code Extension

The Orlint VS Code extension significantly enhances the development experience for Orbit UI projects by providing real-time linting, syntax highlighting, and other useful features directly within the editor.

## Overview

The "Orbit UI Tools" extension for Visual Studio Code is designed to streamline your workflow when working with `.orbit` files. It leverages the power of the Orlint analyzer to provide immediate feedback on your code.

## Features

The extension offers a comprehensive set of features:

- **Syntax Highlighting**: Provides clear and distinct highlighting for `.orbit` file syntax, improving readability.
- **Real-time Linting**: Uses the Orlint (Orbit Analyzer) CLI tool to analyze your code as you type or upon saving, highlighting potential issues directly in the editor.
- **Quick Fixes**: Offers suggestions and quick fixes for common linting errors, helping you resolve issues faster.
- **Hover Information**: Displays helpful information about Orbit components and their props when you hover over them.
- **Go-to-Definition**: Allows you to quickly navigate to the definition of components or variables.
- **Find References**: Helps you find all usages of a specific component or variable within your project.
- **Auto-completion**: Provides intelligent suggestions for Orbit components, props, and events as you type.

## Requirements

To use the Orlint VS Code extension effectively, ensure you have the following:

- **Orlint (Orbit Analyzer) CLI**: The `orlint` command-line tool must be installed and accessible in your system's PATH.
- **Visual Studio Code**: Version 1.85.0 or newer.

## Installation

You can install the extension via the Visual Studio Code Marketplace or manually.

### From Visual Studio Code Marketplace (Recommended)

1.  Open Visual Studio Code.
2.  Navigate to the **Extensions** view (you can use the shortcut `Ctrl+Shift+X` or `Cmd+Shift+X`).
3.  Search for "Orbit UI Tools".
4.  Click the "Install" button.

### Manual Installation (from VSIX)

If you have downloaded a `.vsix` package of the extension (e.g., from a release page):

1.  Download the `.vsix` file from the [latest Orlint releases](https://github.com/orbitrs/orlint/releases) (Note: This link is illustrative; replace with the actual release page if different).
2.  Open Visual Studio Code.
3.  Go to the **Extensions** view.
4.  Click on the "..." (More Actions) menu in the top-right corner of the Extensions view sidebar.
5.  Select "Install from VSIX..."
6.  Choose the downloaded `.vsix` file and click "Install".

## Configuration

The Orlint VS Code extension can be configured through your VS Code `settings.json` file or the settings UI. Access this by going to **File > Preferences > Settings** (or `Code > Preferences > Settings` on macOS) and searching for "orbit".

Available configuration options:

```json
{
  "orbit.analyzer.enable": true, // Enables or disables the analyzer. Default: true
  "orbit.analyzer.path": "orlint", // Path to the orlint executable. Default: "orlint" (assumes it's in PATH)
  "orbit.analyzer.rules": [], // Array of specific rule IDs to enable. Empty array means all default rules are active.
  "orbit.analyzer.configPath": "", // Path to a custom .orlint.toml configuration file. If empty, Orlint searches for it in the workspace root.
  "orbit.analyzer.validateOnSave": true, // Lint files on save. Default: true
  "orbit.analyzer.validateOnType": false, // Lint files as you type. Default: false (can be resource-intensive)
  "orbit.analyzer.severities": { // Customize the severity level for specific rules
    "component-naming": "warning", // Example: Set 'component-naming' rule to report as a warning
    "prop-type-required": "error"  // Example: Set 'prop-type-required' rule to report as an error
  }
}
```

**Key Configuration Notes:**

-   **`orbit.analyzer.path`**: If `orlint` is not in your system's PATH, you'll need to provide the full path to the executable here.
-   **`orbit.analyzer.configPath`**: Useful if your Orlint configuration file (`.orlint.toml`) is not located at the root of your workspace.
-   **`orbit.analyzer.validateOnType`**: Enabling this provides the most immediate feedback but might impact performance on larger files or slower machines.

## Usage

Once installed and enabled, the extension automatically activates when you open `.orbit` files.

-   **Linting Issues**: Detected issues will be highlighted with squiggly underlines directly in your code. You can also view a comprehensive list of problems in the **Problems** panel (View > Problems).
-   **Hover Information**: Hovering over Orbit components or their props will show relevant documentation or type information.
-   **Quick Fixes**: When available, a lightbulb icon will appear next to lines with issues. Clicking it (or using `Ctrl+.` / `Cmd+.`) will show available quick fixes.

### Available Commands

The extension also provides commands accessible via the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`):

-   **`Orbit: Analyze Current File`**: Manually triggers an analysis of the currently active `.orbit` file.
-   **`Orbit: Analyze Workspace`**: Triggers an analysis of all `.orbit` files within the current workspace.

## Building from Source

If you wish to build the extension from its source code (e.g., for development or contributions):

1.  **Clone the Repository**:
    ```bash
    git clone https://github.com/orbitrs/orlint.git # Or your fork
    cd orlint/vscode
    ```

2.  **Install Dependencies**:
    ```bash
    npm install
    ```

3.  **Build the Extension**:
    ```bash
    npm run package
    ```
    This command typically uses a bundler like Webpack (as indicated by `webpack.config.js` in the directory structure) to compile the TypeScript source code (from the `src/` directory) and package it into a `.vsix` file located in the `vscode` or a `dist` subdirectory.

## Troubleshooting

-   **Extension not activating**:
    -   Ensure you have `.orbit` files in your workspace.
    -   Check the VS Code Output panel (select "Orbit UI Tools" from the dropdown) for any error messages.
-   **Linting not working**:
    -   Verify that `orlint` is installed and accessible from your terminal by running `orlint --version`.
    -   If `orlint` is installed but not in PATH, set the `orbit.analyzer.path` configuration correctly.
    -   Check for an `.orlint.toml` configuration file in your project; errors in this file can prevent Orlint from running.

For further assistance or to report issues, please refer to the [Orlint GitHub repository issues page](https://github.com/orbitrs/orlint/issues).
