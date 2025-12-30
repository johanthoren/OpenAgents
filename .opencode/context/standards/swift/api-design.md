# Swift API Design Standards

## Naming

### Follow Swift API Design Guidelines
- **Clarity at the point of use** is the primary goal.
- **Prefer clarity over brevity**. Short names are good only when clear.
- **Use grammatical English phrases** for method names.

```swift
// Good: Reads like English
users.remove(at: index)
view.addSubview(button)
array.insert(element, at: index)

// Avoid: Unclear or overly terse
users.remove(index)      // Remove what?
view.add(button)         // Add where?
array.ins(element, index) // Cryptic
```

### Naming Conventions
- **Types**: UpperCamelCase (`TranscriptionEngine`, `AudioBuffer`)
- **Functions/Properties**: lowerCamelCase (`startRecording`, `isActive`)
- **Protocols**: Noun for "what it is", -able/-ible for "what it can do"

```swift
protocol Transcribable { }      // Capability
protocol TranscriptionDelegate { }  // Role
struct Transcript { }           // Thing
```

## Construction

### Prefer Memberwise Initialization
```swift
// Swift provides this automatically for structs
struct Config {
    var timeout: TimeInterval
    var retryCount: Int
    var baseURL: URL
}

let config = Config(timeout: 30, retryCount: 3, baseURL: url)
```

### Default Parameter Values
```swift
struct Config {
    var timeout: TimeInterval = 30
    var retryCount: Int = 3
    var baseURL: URL
}

// Caller only specifies non-default values
let config = Config(baseURL: url)
```

### Builder Pattern for Complex Construction
```swift
struct Request {
    let url: URL
    let method: HTTPMethod
    let headers: [String: String]
    let body: Data?
    
    static func builder(url: URL) -> Builder {
        Builder(url: url)
    }
    
    struct Builder {
        private var url: URL
        private var method: HTTPMethod = .get
        private var headers: [String: String] = [:]
        private var body: Data?
        
        init(url: URL) { self.url = url }
        
        func method(_ method: HTTPMethod) -> Builder {
            var copy = self
            copy.method = method
            return copy
        }
        
        func header(_ key: String, _ value: String) -> Builder {
            var copy = self
            copy.headers[key] = value
            return copy
        }
        
        func body(_ data: Data) -> Builder {
            var copy = self
            copy.body = data
            return copy
        }
        
        func build() -> Request {
            Request(url: url, method: method, headers: headers, body: body)
        }
    }
}

// Usage
let request = Request.builder(url: endpoint)
    .method(.post)
    .header("Content-Type", "application/json")
    .body(jsonData)
    .build()
```

## Documentation

### Doc Comments for Public API
```swift
/// Transcribes audio data to text using the loaded Whisper model.
///
/// - Parameter audio: The audio buffer containing samples to transcribe.
/// - Returns: The transcribed text.
/// - Throws: `TranscriptionError.modelNotLoaded` if no model is loaded.
///
/// ```swift
/// let engine = TranscriptionEngine()
/// try await engine.loadModel(named: "base")
/// let text = try await engine.transcribe(audio: buffer)
/// ```
func transcribe(audio: AudioBuffer) async throws -> String
```

### Mark Sections in Large Files
```swift
// MARK: - Public API

public func startRecording() async throws { }
public func stopRecording() async { }

// MARK: - Private Helpers

private func configureAudioSession() throws { }
private func handleInterruption(_ notification: Notification) { }

// MARK: - AudioCaptureDelegate

extension Manager: AudioCaptureDelegate {
    func didReceiveAudio(_ buffer: AudioBuffer) { }
}
```

## Error Design

### Use Typed Error Enums
```swift
enum TranscriptionError: Error, LocalizedError {
    case modelNotFound(name: String)
    case modelLoadFailed(underlying: Error)
    case audioCaptureFailed
    case permissionDenied
    
    var errorDescription: String? {
        switch self {
        case .modelNotFound(let name):
            return "Model '\(name)' not found"
        case .modelLoadFailed(let error):
            return "Failed to load model: \(error.localizedDescription)"
        case .audioCaptureFailed:
            return "Audio capture failed"
        case .permissionDenied:
            return "Microphone permission denied"
        }
    }
}
```

### Recoverable vs Fatal Errors
```swift
// Recoverable: Use throws
func loadModel(named: String) async throws -> Model {
    guard let url = Bundle.main.url(forResource: named, withExtension: "mlmodel") else {
        throw TranscriptionError.modelNotFound(name: named)
    }
    return try await Model(contentsOf: url)
}

// Truly unreachable: preconditionFailure (debug) or fatalError
func process(state: State) {
    switch state {
    case .ready: handleReady()
    case .recording: handleRecording()
    @unknown default:
        preconditionFailure("Unknown state: \(state)")
    }
}
```

## Protocol Design

### Small, Focused Protocols
```swift
// Good: Single responsibility
protocol AudioSource {
    func startCapture() async throws
    func stopCapture() async
}

protocol AudioSink {
    func receive(buffer: AudioBuffer)
}

// Avoid: Kitchen sink protocol
protocol AudioManager {
    func startCapture() async throws
    func stopCapture() async
    func receive(buffer: AudioBuffer)
    func setVolume(_ volume: Float)
    func getDevices() -> [AudioDevice]
    // ... too many responsibilities
}
```

### Protocol Extensions for Defaults
```swift
protocol Timestamped {
    var timestamp: Date { get }
}

extension Timestamped {
    var age: TimeInterval {
        Date().timeIntervalSince(timestamp)
    }
    
    var isRecent: Bool {
        age < 60
    }
}
```
