# Windows Linking Issues and Solutions

## Problem

When building the OrbitRS project on Windows, you may encounter linker errors related to missing Registry API functions from the `advapi32.lib` library. These errors typically look like:

```
undefined reference to `__imp_RegOpenKeyExW'
undefined reference to `__imp_RegQueryInfoKeyW'
undefined reference to `__imp_RegCloseKey'
undefined reference to `__imp_RegEnumKeyExW'
undefined reference to `__imp_RegQueryValueExW'
```

These functions are part of the Windows Registry API and are required by the Skia ICU (International Components for Unicode) library, which is used for text rendering.

## Cause

The Skia library's ICU component uses the Windows Registry API to access system font information and other locale data on Windows. When linking, the linker needs to be explicitly told to include the `advapi32.lib` library, which contains these Registry API functions.

## Solutions

There are several ways to solve this issue:

### 1. Using build.rs (Recommended)

Add a `build.rs` file to your project:

```rust
fn main() {
    #[cfg(all(target_os = "windows", target_env = "msvc"))]
    {
        println!("cargo:rustc-link-lib=dylib=advapi32");
        println!("cargo:rustc-link-search=native=C:\\Windows\\System32");
    }
}
```

This script will run during the build process and tell Cargo to link against `advapi32.lib` when building on Windows with the MSVC toolchain.

### 2. Using .cargo/config.toml

Add or modify `.cargo/config.toml` in your project's root:

```toml
[target.x86_64-pc-windows-msvc]
rustflags = ["-C", "link-arg=/DEFAULTLIB:advapi32.lib"]
```

This configuration tells Cargo to pass the `/DEFAULTLIB:advapi32.lib` argument to the linker when building for 64-bit Windows with MSVC.

### 3. Using Environment Variables

Set the `LIB` environment variable before running Cargo:

```powershell
$env:LIB="$env:LIB;$env:WindowsSdkDir\Lib\$env:WindowsSDKVersion\um\x64"
$env:RUSTFLAGS="-C link-arg=advapi32.lib"
cargo build
```

## In CI Environments

When running in CI environments like GitHub Actions, it's best to use a combination of the above approaches:

1. Add a `build.rs` file to ensure dependencies are properly linked
2. Configure the `.cargo/config.toml` file as a fallback
3. Set environment variables to ensure the linker can find the libraries

For an example implementation, see our CI workflow in `.github/workflows/ci.yml`.

## Related Components

This issue specifically affects:
- Skia renderer components
- Text rendering with international character support
- Font management systems
