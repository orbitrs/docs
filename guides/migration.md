# 🔄 Migration Guide

This guide helps you migrate Orbit projects between major versions, ensuring compatibility with new APIs, features, and best practices.

## 🚧 Breaking Changes

### v0.x → v1.0.0

- `.orbit` file syntax updated:
  - `<Component>` tag renamed to `<Orbit>`
  - `on:event` syntax changed to `@event`
- State management:
  - `State<T>` replaced with `use_state` hook
  - `use_store` introduced for global stores
- Renderer configuration:
  - `orbit::run::<App>` now requires `AppConfig`
  - Default render target moved to `web`; use `AppConfig::default().target(Target::Desktop)`

### v1.0.0 → v2.0.0

- Module path changes:
  - `orbitrs::prelude::*` split into granular imports
  - `orbitrs::hooks` moved to `orbitrs::state`
- CLI tool `orbiton`:
  - `orbiton new` flags renamed (`--template` → `--preset`)
  - `orbiton build` now requires `--config` for custom configs

## 🛠 Migration Steps

1. **Update Dependencies**
   ```toml
   [dependencies]
   orbitrs = "^1.0"
   orbiton = "^1.0"
   ```

2. **Refactor `.orbit` Files**
   - Replace `<Component>` tags:
     ```bash
     grep -Rl "<Component>" src/ | xargs sed -i '' 's/<Component>/<Orbit>/g'
     ```
   - Update event bindings:
     ```bash
     grep -Rl "on:" src/ | xargs sed -i '' 's/on:/@/g'
     ```

3. **Migrate State**
   - Replace `State<T>` usage with `use_state`:
     ```bash
     sed -E -i '' 's/State<([^>]+)>/use_state::<\1>/g' src/
     ```

4. **Update Renderer Initialization**
   ```diff
   // ...existing code...
   - orbit::run::<App>(orbit::AppConfig::default())
   + orbit::run::<App>(AppConfig { target: Target::Desktop, ..Default::default() })
   // ...existing code...
   ```

5. **Review CLI Scripts**
   - Adjust `orbiton` commands in CI/CD pipelines
   - Update scripts in `package.json` or shell scripts

6. **Test and Validate**
   - Run your test suite and fix any errors
   - Manually verify critical paths
   - Perform performance benchmarks

## 🔍 Additional Resources

- [Changelog](../../CHANGELOG.md)
- [API Reference](../api/README.md)
- [Core Concepts](../core-concepts/README.md)
