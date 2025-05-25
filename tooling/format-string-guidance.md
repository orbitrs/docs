# Format String Best Practices for Beta Compatibility

**Last Updated**: May 25, 2025  
**Applicable To**: All Rust code in Orbit Framework

## Overview

Recent changes in Rust beta toolchains have introduced stricter linting for format strings. This guide helps developers maintain compatibility between stable and beta Rust toolchains.

## Common Issues

### 1. Uninlined Format Arguments

#### Problem

Using `format!()` with variables passed as arguments:

```rust
// Deprecated pattern
let error_message = format!("Failed to create device: {}", error);
```

This pattern triggers the `uninlined_format_args` warning in beta Rust.

#### Solution

Use inline format arguments:

```rust
// Recommended pattern
let error_message = format!("Failed to create device: {error}");
```

### 2. String Concatenation vs. Format

#### Problem

Using format for simple concatenation:

```rust
// Inefficient
let path = format!("{}/filename.txt", base_dir);
```

#### Solution

Use string concatenation or the `join` method:

```rust
// More efficient
let path = base_dir.to_string() + "/filename.txt";
// Or
use std::path::PathBuf;
let path = PathBuf::from(base_dir).join("filename.txt");
```

## Examples from Our Codebase

### Fixed Format Strings in WGPU Renderer

```rust
// Before
.map_err(|e| Error::Renderer(format!("Failed to create device: {}", e)))?;

// After
.map_err(|e| Error::Renderer(format!("Failed to create device: {e}")))?;
```

## Automated Checks

We've added a script to check for potential format string issues:

```powershell
./scripts/check-beta-compatibility.ps1
```

This script scans the codebase for deprecated format string patterns and other beta compatibility issues.

## CI Integration

Our CI pipeline runs tests and clippy on both stable and beta toolchains. Beta failures are non-blocking but are reported to help identify compatibility issues early.

## When to Address Beta Compatibility

- **During code review**: Address simple format string issues
- **During scheduled maintenance**: Address more complex issues
- **Before stable release**: Address all beta warnings if the next stable Rust release is imminent

## Additional Resources

- [Rust Format Syntax Documentation](https://doc.rust-lang.org/std/fmt/index.html)
- [Clippy Lints: uninlined_format_args](https://rust-lang.github.io/rust-clippy/master/index.html#uninlined_format_args)
