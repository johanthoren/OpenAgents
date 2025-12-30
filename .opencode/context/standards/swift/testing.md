# Swift Testing Standards

## Strategy
- **Unit Tests**: Test core logic with XCTest or Swift Testing framework.
- **Integration Tests**: Test component interactions, especially async flows.
- **UI Tests**: Use XCUITest sparingly for critical user flows.

## Test Structure

### Arrange-Act-Assert Pattern
```swift
func testCalculateTotalWithMultipleItems() throws {
    // Arrange
    let items = [
        Item(name: "A", price: 10.00),
        Item(name: "B", price: 20.00),
        Item(name: "C", price: 30.00)
    ]
    
    // Act
    let total = calculateTotal(items: items)
    
    // Assert
    XCTAssertEqual(total, 60.00, accuracy: 0.01)
}
```

### Swift Testing Framework (iOS 16+, macOS 13+)
```swift
import Testing

@Test func calculateTotalWithMultipleItems() {
    let items = [
        Item(name: "A", price: 10.00),
        Item(name: "B", price: 20.00)
    ]
    
    let total = calculateTotal(items: items)
    
    #expect(total == 30.00)
}

@Test("Empty cart returns zero")
func emptyCart() {
    #expect(calculateTotal(items: []) == 0)
}
```

## Testing Async Code

### XCTest Async
```swift
func testFetchTranscript() async throws {
    let service = TranscriptionService()
    
    let transcript = try await service.fetch(id: "test-123")
    
    XCTAssertFalse(transcript.segments.isEmpty)
}
```

### Testing Actors
```swift
func testTranscriptionEngine() async throws {
    let engine = TranscriptionEngine()
    
    try await engine.loadModel(named: "tiny")
    let result = try await engine.transcribe(audio: testAudioBuffer)
    
    XCTAssertFalse(result.isEmpty)
}
```

## Test Doubles

### Protocol-Based Mocking
```swift
protocol AudioCaptureProtocol {
    func startCapture() async throws
    func stopCapture() async
}

// Production implementation
class AudioCaptureManager: AudioCaptureProtocol {
    func startCapture() async throws { /* real implementation */ }
    func stopCapture() async { /* real implementation */ }
}

// Test double
class MockAudioCapture: AudioCaptureProtocol {
    var startCaptureCalled = false
    var stopCaptureCalled = false
    
    func startCapture() async throws {
        startCaptureCalled = true
    }
    
    func stopCapture() async {
        stopCaptureCalled = true
    }
}
```

### Dependency Injection for Testability
```swift
// Testable: Dependencies injected
class TranscriptionService {
    private let audioCapture: AudioCaptureProtocol
    private let engine: TranscriptionEngine
    
    init(audioCapture: AudioCaptureProtocol, engine: TranscriptionEngine) {
        self.audioCapture = audioCapture
        self.engine = engine
    }
}

// Test
func testServiceStartsCapture() async throws {
    let mockCapture = MockAudioCapture()
    let service = TranscriptionService(
        audioCapture: mockCapture,
        engine: TestTranscriptionEngine()
    )
    
    try await service.startRecording()
    
    XCTAssertTrue(mockCapture.startCaptureCalled)
}
```

## Naming Conventions

```swift
// Good: Descriptive, states expected behavior
func testCalculateDiscount_premiumUser_returns10Percent() { }
func testValidateEmail_invalidFormat_returnsFalse() { }
func testCreateUser_emailExists_throwsDuplicateError() { }

// Avoid: Vague names
func testIt() { }
func testUser() { }
```

## What to Test

### DO Test
- Pure functions and business logic
- Error handling paths
- Edge cases (empty, nil, boundaries)
- Public API contracts
- Async state transitions

### DON'T Test
- Private implementation details
- Apple framework behavior
- Simple property access
- Generated code

## Platform-Specific Tests

```swift
#if os(macOS)
func testScreenCapturePermission() async {
    // macOS-specific test
}
#endif

#if os(iOS)
func testMicrophonePermission() async {
    // iOS-specific test
}
#endif
```
