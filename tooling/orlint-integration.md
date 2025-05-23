# Orlint Integration Guide

The Orlint tool is a powerful static analyzer specifically designed for Orbit applications. This guide explains how to integrate Orlint into your development workflow and get the most out of its capabilities.

## Table of Contents

- [Overview](#overview)
- [Getting Started](#getting-started)
- [Key Features](#key-features)
- [Workflow Integration](#workflow-integration)
- [Configuration](#configuration)
- [Custom Rules](#custom-rules)
- [VS Code Integration](#vs-code-integration)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

## Overview

Orlint analyzes your `.orbit` files and Rust code to identify patterns that might lead to bugs, performance issues, or maintainability problems. It helps enforce best practices and coding standards across your team.

While the `orbiton analyze` command provides a high-level interface to Orlint's capabilities, you can also use Orlint directly for more specialized analysis tasks.

## Getting Started

### Installation

Orlint is automatically installed when you install the Orbiton CLI:

```bash
cargo install orbiton
```

Alternatively, you can install Orlint separately:

```bash
cargo install orlint
```

### Basic Usage

The simplest way to use Orlint is through the `orbiton analyze` command:

```bash
orbiton analyze
```

For more direct control, you can use the `orlint` command:

```bash
# Analyze all .orbit files in the current project
orlint analyze

# Analyze specific files or directories
orlint analyze src/components/

# Fix issues automatically when possible
orlint fix src/
```

## Key Features

Orlint provides several key features for maintaining code quality:

- **Syntax Validation**: Catches syntax errors in `.orbit` files
- **Performance Analysis**: Identifies patterns that might impact performance
- **Accessibility Checking**: Ensures components meet accessibility standards
- **Best Practices**: Enforces idiomatic Orbit coding patterns
- **Style Consistency**: Checks for consistent formatting and naming
- **Auto-fixing**: Automatically fixes common issues
- **Custom Rules**: Supports creating and sharing custom lint rules
- **VS Code Integration**: Provides real-time feedback in your editor

## Workflow Integration

### Git Pre-commit Hooks

Add Orlint to your git pre-commit hooks to catch issues before they enter your codebase:

```bash
# Install pre-commit
npm install --save-dev pre-commit

# Create a .pre-commit-config.yml file
cat > .pre-commit-config.yml << EOF
repos:
  - repo: local
    hooks:
      - id: orlint
        name: Orlint
        entry: orlint analyze
        language: system
        files: \\.orbit$
        pass_filenames: false
EOF
```

### CI/CD Integration

For complete instructions on integrating Orlint into CI/CD pipelines, refer to the [CI/CD Integration Guide](../../orlint/docs/ci-integration.md).

Basic GitHub Actions example:

```yaml
name: Orbit Lint

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Install Rust
      uses: actions-rs/toolchain@v1
      with:
        toolchain: stable
    - name: Install Orlint
      run: cargo install orlint
    - name: Run Orlint
      run: orlint analyze --format json --output orlint-report.json
    - name: Upload Results
      uses: actions/upload-artifact@v4
      with:
        name: orlint-report
        path: orlint-report.json
```

## Configuration

Orlint is highly configurable via an `.orlint.toml` file in your project root:

```toml
# Basic configuration
[analyzer]
include = ["src/**/*.orbit"]
exclude = ["**/tests/**", "**/node_modules/**"]

# Rule configuration
[rules]
disabled = ["perf-no-inline-styles"] # Rules to disable
warning = ["a11y-*"] # Rules to treat as warnings instead of errors

# Performance thresholds
[performance]
rendering_threshold_ms = 16 # Flag components that may render slower than this

# Accessibility settings
[accessibility]
level = "AA" # WCAG compliance level (A, AA, or AAA)
```

For complete configuration options, refer to the [Configuration Guide](../../orlint/docs/configuration.md) and [Sample Configuration](../../orlint/docs/sample-config.toml).

## Custom Rules

Orlint supports creating custom rules tailored to your project's specific needs:

```rust
use orlint::{Rule, RuleDefinition, Context, Diagnostic, Severity};

pub struct NoDeprecatedComponentsRule;

impl Rule for NoDeprecatedComponentsRule {
    fn definition(&self) -> RuleDefinition {
        RuleDefinition {
            id: "custom-no-deprecated-components".to_string(),
            description: "Prevents using deprecated components".to_string(),
            severity: Severity::Warning,
        }
    }

    fn check(&self, context: &Context) -> Vec<Diagnostic> {
        let mut diagnostics = Vec::new();
        
        // Check for imports of deprecated components
        if let Some(imports) = context.get_imports() {
            for import in imports {
                if import.path.contains("LegacyComponent") {
                    diagnostics.push(Diagnostic {
                        message: "Using deprecated component".to_string(),
                        location: import.location,
                        severity: Severity::Warning,
                        suggestions: vec!["Use NewComponent instead".to_string()],
                    });
                }
            }
        }
        
        diagnostics
    }
}
```

For detailed instructions on creating and sharing custom rules, see the [Custom Lint Rules Guide](../../orlint/docs/custom-lint-rules.md).

## VS Code Integration

Orlint integrates with Visual Studio Code to provide real-time feedback as you write code:

1. Install the Orlint VS Code extension
2. Configure the extension settings
3. Enjoy real-time linting and auto-fix capabilities

For detailed setup instructions and features, see the [VS Code Integration Guide](../../orlint/docs/vscode-integration.md).

## Troubleshooting

When encountering issues with Orlint, try these troubleshooting steps:

1. **Check for Updates**: Ensure you're using the latest version: `cargo install orlint --force`
2. **Increase Verbosity**: Run with `--verbose` flag for more information: `orlint analyze --verbose`
3. **Examine Configuration**: Verify your `.orlint.toml` file for errors
4. **Check Rule IDs**: Ensure rule IDs in your configuration are correct
5. **Isolate Problem Files**: Run Orlint on specific files to isolate issues

For more troubleshooting information, see the [Troubleshooting Guide](../../orlint/docs/troubleshooting.md).

## Resources

- [Orlint CLI Usage Guide](../../orlint/docs/cli-usage.md) - Complete command reference
- [Configuration Guide](../../orlint/docs/configuration.md) - Detailed configuration options
- [Custom Lint Rules](../../orlint/docs/custom-lint-rules.md) - Creating custom rules
- [VS Code Integration](../../orlint/docs/vscode-integration.md) - Editor integration
- [Performance Guide](../../orlint/docs/performance-guide.md) - Performance analysis features
- [CI/CD Integration](../../orlint/docs/ci-integration.md) - Continuous integration setup
- [Advanced Usage](../../orlint/docs/advanced-usage.md) - Advanced features and techniques
