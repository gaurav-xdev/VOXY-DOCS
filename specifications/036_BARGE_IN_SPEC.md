# 036 - Barge-In Specification

## Purpose

Define the Barge-In subsystem that enables users to interrupt TTS playback with new voice commands, managing the transition from output to input seamlessly.

## Scope

- TTS interruption detection
- Audio input monitoring during output
- State transition management
- Partial utterance handling
- Visual feedback coordination

## Responsibilities

- Monitor for user interruption during TTS
- Stop TTS playback on interruption
- Transition pipeline to listening state
- Handle partial utterances
- Coordinate with UI for visual feedback

## Non-Goals

- TTS synthesis (TTS subsystem)
- Audio capture (Audio Capture subsystem)
- Wake word detection during barge-in (optional)

## Architecture Position

Barge-In sits between TTS and the Audio Pipeline, monitoring both output and input simultaneously.

## Inputs

- TTS playback status
- Audio input from capture
- VAD events during playback
- User interaction events

## Outputs

- Interruption events to Audio Pipeline
- TTS stop commands
- State transition signals

## Interfaces

```rust
pub struct BargeIn {
    pub fn enable();
    pub fn disable();
    pub fn is_enabled() -> bool;
    pub fn on_tts_start();
    pub fn on_tts_end();
    pub fn process_input(audio: AudioFrame) -> Option<BargeInEvent>;
}
```

## Data Models

### BargeInEvent

| Field | Type | Description |
|-------|------|-------------|
| timestamp | DateTime | Interruption time |
| type | BargeInType | Voice, Key, Gesture |
| confidence | f32 | Detection confidence |

### BargeInType

| Variant | Description |
|---------|-------------|
| `Voice` | Speech detected during TTS |
| `Key` | Keyboard interrupt key pressed |
| `Gesture` | UI gesture (tap, click) |

## State Management

- BargeIn state: Disabled -> Enabled -> Monitoring -> Interrupted -> Reset

## Threading Model

Barge-In runs on the audio capture thread for minimal latency. TTS coordination is async.

## IPC Requirements

- No IPC for audio processing
- UI events may come via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `bargein.interrupted` | BargeInEvent | Interruption detected |
| `bargein.enabled` | {} | Barge-in enabled |
| `bargein.disabled` | {} | Barge-in disabled |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `BI001` | False interruption | Resume TTS if confidence low |
| `BI002` | State race | Synchronize with mutex |

## Retry Strategy

- None (real-time)

## Recovery Strategy

- False positive: Resume TTS from interruption point
- State race: Re-evaluate state

## Security Requirements

- No special requirements

## Configuration

```json
{
  "barge_in": {
    "enabled": true,
    "voice_sensitivity": 0.8,
    "min_interruption_ms": 200,
    "resume_on_false_positive": true,
    "visual_feedback": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Interruption detection | < 100ms |
| TTS stop latency | < 50ms |
| False positive rate | < 2% |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Audio buffer | 100KB |

## Benchmarks

- `bench_detection`: Interruption detection time
- `bench_stop_latency`: TTS stop time
- `bench_false_positive`: False rate

## Testing Requirements

- Interruption detection tests
- False positive tests
- State transition tests
- Resume tests

## Logging Requirements

- Interruptions at INFO level
- False positives at DEBUG level

## Telemetry

- Interruption count
- Detection latency
- False positive rate

## Cross References

- [030_AUDIO_PIPELINE_SPEC.md](030_AUDIO_PIPELINE_SPEC.md)
- [035_TTS_SPEC.md](035_TTS_SPEC.md)
- [033_VAD_SPEC.md](033_VAD_SPEC.md)

## References

- Voice User Interface Design
- Conversational AI Patterns
