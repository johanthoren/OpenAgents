# Functional Patterns in Rust

## Philosophy
Embrace functional programming principles adapted to Rust's ownership model.
Clojure-inspired, but idiomatic Rust.

## Core Principles

### 1. Pure Functions by Default
- Core logic functions should be pure: no I/O, no mutation of external state.
- Side effects (I/O, logging) belong at the edges (main.rs, top-level orchestration).

### 2. Data Pipelines
- Prefer iterator chains over imperative loops.
- Use `.map()`, `.filter()`, `.fold()`, `.collect()` for transformations.

```rust
// Preferred
let deltas = durations.windows(2).map(|w| diff(w[0], w[1])).collect();

// Avoid
let mut deltas = Vec::new();
for i in 0..durations.len() - 1 {
    deltas.push(diff(durations[i], durations[i + 1]));
}
```

### 3. Consuming Builders
- Builder methods should take `self` (not `&mut self`) and return `Self`.
- This creates a fluent, chainable API that mirrors immutable data transformations.

```rust
pub fn add_value(mut self, key: &str, value: &str) -> Self {
    self.values.insert(key.to_string(), value.to_string());
    self
}
```

### 4. Higher-Order Functions
- Accept closures for customization (sorting, filtering, mapping).
- Use generics with trait bounds (`F: FnMut(...) -> ...`).

### 5. Option/Result as First-Class Values
- Use combinators (`.map()`, `.and_then()`, `.ok_or()`, `.map_err()`) over `match` when appropriate.
- Propagate errors with `?` operator.

## Anti-Patterns to Avoid

### Excessive Cloning
- Clojure's structural sharing doesn't exist in Rust. `.clone()` is a real cost.
- Prefer borrowing (`&T`, `&[T]`) for read-only access.
- If ownership is needed, let the caller decide whether to clone.

```rust
// Prefer: borrow when you only need to read
fn calculate_median(deltas: &[Duration]) -> f64 { ... }

// Avoid: forcing a clone on every call
fn calculate_median(deltas: Vec<Duration>) -> f64 { ... }
```

### Mutation Leakage
- Mutation is acceptable *within* a function, but APIs should appear immutable.
- Don't expose `&mut self` in public builder APIs; consume and return `Self` instead.
