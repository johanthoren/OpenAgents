# General Rust Standards

## Error Handling
- **Library**: Use `thiserror` for typed, enumerable errors. Avoid `anyhow` in libraries.
- **Binary**: Map typed errors to integer exit codes.
- **Safety**: No panics in production code.

## Dependency Management
- **Feature Gating**: Gate heavy dependencies (e.g., crypto, serialization) behind Cargo features.
- **Lean Core**: Keep the default feature set minimal.

## Module Structure
- **Prelude**: Export a `prelude` module with common types and traits for easy consumption.
