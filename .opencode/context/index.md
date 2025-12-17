# Context Index

## Quick Map

### Core Standards (Language-Agnostic)
Path: `.opencode/context/core/standards/{file}`
```
code        → standards/code.md       [critical] implement, refactor, architecture
docs        → standards/docs.md       [critical] write docs, README, documentation
tests       → standards/tests.md      [critical] write tests, testing, TDD → deps: code
patterns    → standards/patterns.md   [high]     error handling, security, validation
analysis    → standards/analysis.md   [high]     analyze, investigate, debug
```

### Rust Standards (Load alongside core for Rust projects)
Path: `.opencode/context/standards/rust/{file}`
```
rust-general        → rust/rust-general.md        [high] rust, cargo, thiserror → deps: code, patterns
functional-patterns → rust/functional-patterns.md [high] rust, iterators, builders → deps: code
testing             → rust/testing.md             [high] rust tests, proptest, assert_cmd → deps: tests
api-design          → rust/api-design.md          [high] rust API, builder pattern, doctests → deps: docs
cli                 → rust/cli.md                 [high] rust CLI, clap, stdout/stderr → deps: code
build-release       → rust/build-release.md       [medium] rust build, docker, cross-compile
nagios              → rust/nagios.md              [medium] nagios plugin, monitoring, exit codes
```

### Core Workflows
Path: `.opencode/context/core/workflows/{file}`
```
delegation  → workflows/delegation.md     [high]   delegate, task tool, subagent
review      → workflows/review.md         [high]   review code, audit → deps: code, patterns
breakdown   → workflows/task-breakdown.md [high]   break down, 4+ files → deps: delegation
sessions    → workflows/sessions.md       [medium] session management, cleanup
```

### Project Workflows
Path: `.opencode/context/workflows/{file}`
```
new-project  → workflows/new-project.md  [medium] new rust project, scaffold, cargo new
release      → workflows/release.md      [medium] release, version bump, tag
verification → workflows/verification.md [medium] verify, lint, clippy, fmt
```

## Loading Instructions

**For common tasks, use quick map above. For keyword matching, scan triggers.**

**Format:** `id → path [priority] triggers → deps: dependencies`

**Dependencies:** Load dependent contexts alongside main context for complete guidelines.

**Rust Projects:** When working on Rust code, load relevant `standards/rust/*` files alongside core standards.

## Priority / Precedence

**More specific overrides less specific.** When standards conflict:

1. **Language-specific standards** (e.g., `standards/rust/*`) override core standards
2. **Project-specific standards** (if present) override language-specific standards
3. **Core standards** provide defaults when no specific guidance exists

Example: Core says "immutability always" but Rust standards say "mutation acceptable within functions if API appears immutable" → Follow Rust standard.

## Categories

**Core Standards** - Language-agnostic defaults for code quality, testing, documentation
**Rust Standards** - Rust-specific patterns (takes precedence over core for Rust projects)
**Core Workflows** - Process templates for delegation, review, task breakdown
**Project Workflows** - Development lifecycle workflows
**System** - Documentation and guides
