# Testing Standards

## Strategy
- **Unit Tests**: Test core logic in `lib.rs`.
- **Integration Tests**: Test the binary with `assert_cmd` in `tests/`.
- **Property-Based Tests**: Use `proptest` for complex logic and invariant verification.

## Platform Support
- Use `#[cfg(target_os = ...)]` to handle platform-specific test cases (e.g., socket permissions).
