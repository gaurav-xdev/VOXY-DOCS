# 031 - Audio Capture Specification

## Purpose

Define the audio capture subsystem that interfaces with Windows Audio Session API (WASAPI) to capture microphone input and system loopback audio with minimal latency.

## Scope

- WASAPI client implementation
- Microphone capture
- System audio loopback capture
- Device enumeration and selection
- Format negotiation
- Event-driven capture (not polling)

## Responsibilities

- Capture audio from selected input devices
- Support microphone and loopback modes
- Negotiate audio format with device
- Deliver audio frames to Audio Pipeline
- Handle device changes (plug/unplug)
- Provide device enumeration

## Non-Goals

- Audio processing (VAD, STT handled by other subsystems)
- Audio output (TTS subsystem)
- Audio effects or filtering

## Architecture Position

Audio Capture is the entry point of the audio pipeline. It feeds raw audio to VAD and Wake Word detection.

## Inputs

- Device selection commands
- Capture start/stop commands
- Format preferences

## Outputs

- AudioFrame structs to Audio Pipeline
- Device change events
- Format negotiation results

## Interfaces

```rust
pub struct AudioCapture {
    pub fn enumerate_devices() -> Vec<AudioDevice>;
    pub fn select_device(id: &str) -> Result<(), CaptureError>;
    pub fn start_capture(mode: CaptureMode) -> Result<AudioStream, CaptureError>;
    pub fn stop_capture();
    pub fn get_format() -> AudioFormat;
}
```

## Data Models

### AudioDevice

| Field | Type | Description |
|-------|------|-------------|
| id | String | WASAPI device ID |
| name | String | Human-readable name |
| type | DeviceType | Microphone, Loopback, Communications |
| is_default | bool | Is default device |
| supported_formats | Vec<AudioFormat> | Supported formats |

### AudioFrame

| Field | Type | Description |
|-------|------|-------------|
| timestamp | DateTime | Capture timestamp |
| sample_rate | u32 | Sample rate (Hz) |
| channels | u16 | Channel count |
| format | SampleFormat | f32, i16, etc. |
| data | Vec<u8> | Raw audio bytes |
| duration_ms | f64 | Frame duration |

### CaptureMode

| Variant | Description |
|---------|-------------|
| `Microphone` | Capture from microphone input |
| `Loopback` | Capture system audio output |
| `Mixed` | Mix microphone and loopback |

## State Management

- Device state: Disconnected -> Connected -> Selected -> Capturing
- Capture state: Stopped -> Starting -> Running -> Stopping -> Stopped

## Threading Model

WASAPI uses a dedicated capture thread with event-driven callbacks. Audio frames are passed to Audio Pipeline via lock-free ring buffer.

## IPC Requirements

- No IPC for audio frames
- Device selection may be controlled via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `audio.device.connected` | AudioDevice | Device plugged in |
| `audio.device.disconnected` | device_id | Device removed |
| `audio.device.selected` | device_id | Device selected |
| `audio.capture.started` | format | Capture started |
| `audio.capture.stopped` | reason | Capture stopped |
| `audio.capture.error` | CaptureError | Capture error |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `AC001` | Device not found | Enumerate, select default |
| `AC002` | Format negotiation failed | Try fallback formats |
| `AC003` | Device lost during capture | Stop, notify, wait for reconnect |
| `AC004` | WASAPI initialization failed | Retry, alert |
| `AC005` | Permission denied | Request consent, alert user |

## Retry Strategy

- Device init: 3 retries, 500ms delay
- Format negotiation: Try all supported formats
- Reconnect: Infinite retry, 1s intervals

## Recovery Strategy

- Device lost: Switch to default device
- Format failure: Use system default format
- Permission failure: Prompt user

## Security Requirements

- Microphone access requires Windows privacy consent
- Loopback capture requires no special permissions
- Audio data is not persisted without explicit user action

## Configuration

```json
{
  "audio_capture": {
    "preferred_sample_rate": 16000,
    "preferred_channels": 1,
    "preferred_format": "f32",
    "buffer_duration_ms": 20,
    "device_change_polling_ms": 1000,
    "fallback_to_default": true,
    "loopback_mode": "include"
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Capture latency | < 20ms |
| Device enumeration | < 100ms |
| Device switch | < 50ms |
| Buffer underrun | 0 |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Ring buffer size | 1MB |
| Concurrent captures | 2 |
| Device poll interval | 1s |

## Benchmarks

- `bench_capture_latency`: Frame delivery latency
- `bench_device_enum`: Enumeration speed
- `bench_device_switch`: Switch time

## Testing Requirements

- Device plug/unplug tests
- Format negotiation tests
- Permission denial tests
- Buffer overflow tests

## Logging Requirements

- Device changes at INFO level
- Capture start/stop at INFO level
- Errors at ERROR level
- Format at DEBUG level

## Telemetry

- Capture duration
- Device switch frequency
- Buffer levels
- Underrun count

## Cross References

- [030_AUDIO_PIPELINE_SPEC.md](030_AUDIO_PIPELINE_SPEC.md)
- [033_VAD_SPEC.md](033_VAD_SPEC.md)
- [032_WAKE_WORD_SPEC.md](032_WAKE_WORD_SPEC.md)

## References

- WASAPI Documentation (Microsoft)
- Windows Core Audio APIs
- CPAL Audio Library
