# 064 - Screen Capture Specification

## Purpose

Define the Screen Capture subsystem that captures the Windows desktop and application windows using the Windows Graphics Capture API for hardware-accelerated, privacy-respecting screen capture.

## Scope

- Windows Graphics Capture API integration
- Full screen capture
- Window-specific capture
- Region capture
- Capture session management
- Frame rate control

## Responsibilities

- Capture screen content
- Support window-specific capture
- Manage capture sessions
- Provide frames to Vision Runtime
- Handle capture errors
- Respect user privacy settings

## Non-Goals

- Video recording (future enhancement)
- Screen sharing
- Remote desktop

## Architecture Position

Screen Capture provides visual input to Vision Runtime, OCR, and Grounding Engine.

## Inputs

- Capture requests from Automation Runtime
- Window selection
- Region specifications

## Outputs

- Captured frames
- Capture status events
- Error reports

## Interfaces

```rust
pub struct ScreenCapture {
    pub fn capture_full_screen() -> Result<Frame, CaptureError>;
    pub fn capture_window(window_id: WindowId) -> Result<Frame, CaptureError>;
    pub fn capture_region(region: Region) -> Result<Frame, CaptureError>;
    pub fn start_session(config: CaptureConfig) -> Result<CaptureSession, CaptureError>;
    pub fn stop_session(session: CaptureSession);
}
```

## Data Models

### Frame

| Field | Type | Description |
|-------|------|-------------|
| data | Vec<u8> | Raw pixel data (BGRA) |
| width | u32 | Frame width |
| height | u32 | Frame height |
| timestamp | DateTime | Capture time |
| source | CaptureSource | Screen, Window, Region |

### CaptureConfig

| Field | Type | Description |
|-------|------|-------------|
| target | CaptureTarget | What to capture |
| fps | f32 | Capture frame rate |
| format | PixelFormat | BGRA, RGBA |

## State Management

- Capture sessions tracked
- No persistent state

## Threading Model

Capture runs on dedicated thread with Direct3D device.

## IPC Requirements

- No IPC for capture

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `capture.frame` | Frame | Frame captured |
| `capture.session.started` | session | Session started |
| `capture.session.stopped` | session | Session stopped |
| `capture.error` | CaptureError | Capture error |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `SC001` | Access denied | Request permission |
| `SC002` | Window closed | Stop capture |
| `SC003` | Device lost | Recreate device |

## Retry Strategy

- Capture: 2 retries
- Session: 1 retry

## Recovery Strategy

- Access denied: Request user consent
- Device lost: Recreate

## Security Requirements

- Capture requires user consent
- UAC screens cannot be captured
- Secure desktop cannot be captured

## Configuration

```json
{
  "screen_capture": {
    "default_fps": 30,
    "format": "BGRA",
    "max_resolution": "1920x1080",
    "enable_hardware_acceleration": true,
    "privacy_check": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Capture latency | < 50ms |
| Frame rate | 30 FPS |
| Memory per frame | < 8MB |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Concurrent sessions | 5 |
| Max resolution | 4K |
| Frame buffer | 100MB |

## Benchmarks

- `bench_capture`: Single capture time
- `bench_fps`: Sustained frame rate
- `bench_memory`: Memory usage

## Testing Requirements

- Capture accuracy tests
- Privacy boundary tests
- Error handling tests

## Logging Requirements

- Sessions at INFO level
- Errors at ERROR level

## Telemetry

- Capture frequency
- Average latency
- Error rate

## Cross References

- [060_AUTOMATION_RUNTIME_SPEC.md](060_AUTOMATION_RUNTIME_SPEC.md)
- [070_VISION_RUNTIME_SPEC.md](070_VISION_RUNTIME_SPEC.md)
- [065_OCR_SPEC.md](065_OCR_SPEC.md)

## References

- Windows Graphics Capture API
- Direct3D 11
- Windows Privacy Settings
