# Swift 6 Concurrency Standards

## Philosophy
Swift 6 enforces strict concurrency checking at compile time. Embrace the actor model
and let the compiler catch data races before they happen.

## Core Concepts

### Actor Isolation
- **`@MainActor`**: Use for all UI-related state and updates.
- **Custom actors**: Use for mutable state that needs concurrent access.
- **`nonisolated`**: Mark methods that don't touch actor state.

```swift
@MainActor
@Observable
class AppState {
    var isRecording = false
    var transcriptCount = 0
    
    nonisolated func formatStatus() -> String {
        // Pure function, no actor state needed
        "Ready"
    }
}
```

### Sendable
- All types crossing actor boundaries must be `Sendable`.
- Value types (structs, enums) with `Sendable` properties are implicitly `Sendable`.
- Use `@unchecked Sendable` sparingly and only when you can guarantee thread safety.

```swift
// Implicitly Sendable (all properties are Sendable)
struct TranscriptSegment {
    let timestamp: Date
    let text: String
    let confidence: Double
}

// Explicitly Sendable class (requires careful implementation)
final class AudioBuffer: @unchecked Sendable {
    private let lock = NSLock()
    private var samples: [Float] = []
    
    func append(_ newSamples: [Float]) {
        lock.lock()
        defer { lock.unlock() }
        samples.append(contentsOf: newSamples)
    }
}
```

### Async/Await
- Prefer `async/await` over callbacks and completion handlers.
- Use `Task` for launching concurrent work from synchronous contexts.
- Use `TaskGroup` for structured concurrency with multiple parallel operations.

```swift
// Preferred: async/await
func fetchTranscript(id: String) async throws -> Transcript {
    let data = try await networkService.fetch(endpoint: .transcript(id))
    return try decoder.decode(Transcript.self, from: data)
}

// Launching from synchronous context
func startRecording() {
    Task { @MainActor in
        await audioManager.startCapture()
        isRecording = true
    }
}
```

## Patterns

### Main Actor Isolation for UI State
```swift
@MainActor
@Observable
class ViewModel {
    var items: [Item] = []
    var isLoading = false
    var error: Error?
    
    func load() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            items = try await service.fetchItems()
        } catch {
            self.error = error
        }
    }
}
```

### Background Work with Actor
```swift
actor TranscriptionEngine {
    private var model: WhisperModel?
    
    func loadModel(named: String) async throws {
        model = try await WhisperModel.load(named: named)
    }
    
    func transcribe(audio: AudioBuffer) async throws -> String {
        guard let model else {
            throw TranscriptionError.modelNotLoaded
        }
        return try await model.transcribe(audio)
    }
}
```

### Bridging Callback APIs
```swift
func requestPermission() async -> Bool {
    await withCheckedContinuation { continuation in
        AVCaptureDevice.requestAccess(for: .audio) { granted in
            continuation.resume(returning: granted)
        }
    }
}
```

## Anti-Patterns

### Avoid Data Races
```swift
// BAD: Mutable state without protection
class Counter {
    var count = 0  // Data race!
    func increment() { count += 1 }
}

// GOOD: Actor protection
actor Counter {
    var count = 0
    func increment() { count += 1 }
}
```

### Avoid Blocking the Main Actor
```swift
// BAD: Heavy work on main actor
@MainActor
func processLargeFile() {
    let data = heavyComputation()  // Blocks UI!
    updateUI(with: data)
}

// GOOD: Offload heavy work
@MainActor
func processLargeFile() async {
    let data = await Task.detached {
        heavyComputation()
    }.value
    updateUI(with: data)
}
```
