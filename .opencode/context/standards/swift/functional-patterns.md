# Functional Patterns in Swift

## Philosophy
Embrace functional programming principles adapted to Swift's value semantics.
Clojure-inspired, but idiomatic Swift.

## Core Principles

### 1. Pure Functions by Default
- Core logic functions should be pure: no I/O, no mutation of external state.
- Side effects (I/O, logging, network) belong at the edges (App.swift, top-level orchestration).

```swift
// Pure: same input always produces same output
func calculateTotal(items: [Item]) -> Decimal {
    items.reduce(0) { $0 + $1.price }
}

// Impure: side effects belong at the edges
func processOrder(items: [Item]) async throws {
    let total = calculateTotal(items: items)  // Pure calculation
    try await paymentService.charge(total)    // Side effect at edge
    logger.info("Order processed")            // Side effect at edge
}
```

### 2. Data Pipelines
- Prefer higher-order functions over imperative loops.
- Use `.map()`, `.filter()`, `.reduce()`, `.compactMap()` for transformations.

```swift
// Preferred
let activeUserNames = users
    .filter { $0.isActive }
    .map { $0.name }
    .sorted()

// Avoid
var activeUserNames: [String] = []
for user in users {
    if user.isActive {
        activeUserNames.append(user.name)
    }
}
activeUserNames.sort()
```

### 3. Fluent Builders
- For complex object construction, use mutating methods that return `Self`.
- Swift's value semantics make this safe without Rust's ownership transfer.

```swift
struct RequestBuilder {
    private var timeout: TimeInterval = 30
    private var headers: [String: String] = [:]
    
    func timeout(_ value: TimeInterval) -> Self {
        var copy = self
        copy.timeout = value
        return copy
    }
    
    func header(_ key: String, _ value: String) -> Self {
        var copy = self
        copy.headers[key] = value
        return copy
    }
}

// Usage
let request = RequestBuilder()
    .timeout(60)
    .header("Authorization", "Bearer token")
```

### 4. Higher-Order Functions
- Accept closures for customization (sorting, filtering, mapping).
- Use generics with protocol constraints.

```swift
func process<T: Sequence>(
    items: T,
    transform: (T.Element) -> String
) -> [String] {
    items.map(transform)
}
```

### 5. Optional and Result as First-Class Values
- Use combinators (`.map()`, `.flatMap()`, `??`) over `if let` when appropriate.
- Propagate errors with `try` and `throws`.

```swift
// Preferred: Combinator style for simple transforms
let userName = user?.profile?.displayName ?? "Anonymous"

// Preferred: if let for complex logic or multiple uses
if let user = currentUser {
    updateUI(for: user)
    logActivity(user.id)
}

// Result type for explicit error handling
func parse(_ data: Data) -> Result<Config, ParseError> {
    // ...
}
```

## Anti-Patterns to Avoid

### Unnecessary Reference Types
- Swift structs use copy-on-write; copies are cheap until mutation.
- Don't reach for `class` just to avoid "copying".

```swift
// Preferred: Value type
struct User {
    var name: String
    var email: String
}

// Avoid: Reference type without good reason
class User {
    var name: String
    var email: String
}
```

### Mutation Leakage
- Mutation is acceptable *within* a function, but APIs should appear immutable.
- Use `let` by default; only use `var` when mutation is needed.

```swift
// Preferred: Immutable by default
let users = fetchUsers()
let activeUsers = users.filter { $0.isActive }

// Avoid: Unnecessary mutation
var users = fetchUsers()
users = users.filter { $0.isActive }  // Why var?
```

### Force Unwrapping
- Avoid `!` except in tests or truly invariant conditions.
- Prefer `guard let`, `if let`, or `??` with sensible defaults.

```swift
// Preferred
guard let url = URL(string: urlString) else {
    throw ConfigError.invalidURL(urlString)
}

// Avoid in production code
let url = URL(string: urlString)!
```
