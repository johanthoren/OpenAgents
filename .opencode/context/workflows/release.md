# Release Workflow

## Overview
Process for building and releasing artifacts.

## Steps
1. **Version Bump**: Update `Cargo.toml`.
2. **Hermetic Build**: Run `make all` (triggers Docker builds).
3. **Verify Artifacts**: Check for static binaries and packages.
4. **Tag**: `git commit -am "Release <version>" && git tag v<version>`
