# Compilation Process

This document outlines how Orbit applications are compiled, from parsing `.orbit` files to producing final artifacts. It is intended for contributors who need to understand the framework internals.

## Overview

1. **Source Structure**: An Orbit project contains:
   - Rust code (`*.rs`) under `src/`
   - Single-file components (`*.orbit`) under `src/`
   - Static assets under `static/`

2. **Compilation Phases**:
   1. **Parsing**
   2. **Template/Style Code Generation**
   3. **Rust Compilation**
   4. **Bundling & Assets**

---

## 1. Parsing `.orbit` Files

Orbiton reads each `.orbit` file and splits it into three sections:

- `<template>`: HTML-like structure
- `<style>`: Scoped CSS
- `<code lang="rust">`: Component logic

The parser produces an **Abstract Syntax Tree (AST)** for each section. Key steps:

1. **Tokenization**: Identify tags, code blocks, and text nodes.
2. **AST Construction**: Build separate ASTs:
   - **Template AST**: Nodes for elements, attributes, directives.
   - **Style AST**: CSS rules, selectors, declarations.
   - **Code AST**: Rust tokens (leveraging the Rust compiler's parser).

<!-- Placeholder: diagram of `.orbit` parsing -->

---

## 2. Code Generation

### Template => Rust Render Function

The template AST is transformed into a Rust `fn render(&self) -> VirtualDomNode`:

- Elements become calls to `html!` or equivalent macros.
- Interpolations (`{{ expr }}`) generate expressions in the render function.
- Directives (`@click`, `:class`) map to event bindings and attribute logic.

### Style Scoping

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
