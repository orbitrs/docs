# Beta Compatibility Testing Guide

**Last Updated**: May 25, 2025  
**Applicability**: All Orbit Framework developers

## Overview

This guide outlines the process for testing compatibility with the Rust beta toolchain. Regular beta compatibility testing helps identify potential issues before they affect the stable release.

## Why Test with Beta?

- **Early Warning**: Identify breaking changes before they hit stable
- **Forward Compatibility**: Ensure code works with future Rust versions
- **Smooth Upgrades**: Avoid last-minute scrambles when new Rust versions are released

## Testing Schedule

- **Automated**: Our CI runs beta toolchain tests on every PR (non-blocking)
- **Manual**: Regular scheduled reviews
  - Next scheduled review: **June 1, 2025** (Week 2 of MVP Sprint)
  - Subsequent reviews: Every two weeks during active development

## How to Test Locally

### Setting Up

1. **Install beta toolchain**:
   ```powershell
   rustup toolchain install beta
   ```

2. **Update beta toolchain**:
   ```powershell
   rustup update beta
   ```

### Running Tests

1. **Run tests with beta**:
   ```powershell
   cargo +beta test --workspace
   ```

2. **Run clippy with beta**:
   ```powershell
   cargo +beta clippy --workspace -- -D warnings
   ```

3. **Run automated check script**:
   ```powershell
   ./scripts/check-beta-compatibility.ps1
   ```

## Common Issues and Solutions

### 1. Format String Issues

**Problem**: Using `format!()` with variables passed as separate arguments.
**Fix**: Use inline format arguments (see [format-string-guidance.md](./format-string-guidance.md)).

### 2. CI Configuration Issues

**Problem**: Using features that don't exist in all packages.
**Fix**: Verify package features before using them in CI workflows.

### 3. API Deprecations

**Problem**: Using APIs that are being deprecated.
**Fix**: Follow the migration guides for deprecated APIs.

## Reporting Issues

When you encounter beta compatibility issues:

1. Update the [Beta Compatibility Status](./beta-compatibility-status.md) document
2. Create a GitHub issue with the "beta-compatibility" label
3. Fix critical issues promptly, especially near Rust release dates

## Integration with Development Workflow

- Add beta testing to your personal development workflow
- Use the pre-commit hook in `scripts/pre-commit-hook`
- Consider writing tests that specifically verify beta compatibility
- Keep the documentation updated with new findings
