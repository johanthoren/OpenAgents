# Verification Workflow

## Overview
Standard testing and linting procedure.

## Steps
1. **Unit Tests**: `cargo test --lib`
2. **Integration Tests**: `cargo test --test '*'`
3. **Property Tests**: `cargo test --test proptest`
4. **Linting**:
   - `cargo fmt -- --check`
   - `cargo clippy -- -D warnings`
