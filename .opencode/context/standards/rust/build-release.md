# Build & Release Standards

## Hermetic Builds
- **Docker**: Use Docker for reproducible builds.
- **Makefiles**: Automate build steps with a `Makefile`.

## Cross-Compilation
- Support Linux (musl) and Windows (mingw) targets via Docker.
- Produce static binaries where possible.
