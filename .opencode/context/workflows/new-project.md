# New Project Workflow

## Overview
Standard procedure for scaffolding new Rust projects.

## Steps
1. **Initialize**:
   - CLI: `cargo new --bin <name>`
   - Lib: `cargo new --lib <name>`
2. **Structure**:
   - Create `src/lib.rs` (even for CLIs).
   - Setup `tests/` directory.
3. **Dependencies**:
   - Add `thiserror`, `clap`, `log`.
   - Add `assert_cmd`, `predicates`, `proptest` to `dev-dependencies`.
4. **Build Setup**:
   - Create `Dockerfile.linux` and `Dockerfile.windows`.
   - Create `Makefile`.
