# General Swift Standards

## Error Handling
- **Typed Errors**: Use `Error` protocol with enums for typed, enumerable errors.
- **Binary**: Map typed errors to appropriate exit codes in CLI tools.
- **Safety**: No `fatalError()` or `preconditionFailure()` in production code. Use throwing functions instead.

```swift
// Preferred: Typed error enum
enum TranscriptionError: Error {
    case modelNotFound(String)
    case audioCaptureFailed(underlying: Error)
    case permissionDenied
}

// Avoid: fatalError in production paths
fatalError("This should never happen") // Only acceptable in truly unreachable code
```

## Type Selection
- **Prefer `struct`**: Use structs by default. They're value types with copy-on-write semantics.
- **Use `class` when**: Identity matters, inheritance is needed, or reference semantics are required.
- **Use `actor` when**: Mutable state needs concurrent access protection.
- **Use `enum`**: For finite sets of related values, especially errors.

## Dependency Management
- **Lean Dependencies**: Keep the dependency count minimal.
- **License Compliance**: Only MIT, Apache 2.0, or similarly permissive licenses.
- **Conditional Dependencies**: Use SPM conditions for platform-specific dependencies.

## Module Structure
- **Public API Surface**: Export only what consumers need. Use `internal` (default) or `private` liberally.
- **File Organization**: One primary type per file, named after the type.
