# 033 - VAD (Voice Activity Detection) Specification

## Purpose

Define the Voice Activity Detection subsystem that distinguishes speech from silence and non-speech audio, optimizing downstream processing and reducing resource consumption.

## Scope

- Real-time voice activity detection
- Speech/silence segmentation
- Noise robustness
- Energy-based and ML-based detection
- Event generation for pipeline control

## Responsibilities

- Detect presence of speech in audio stream
- Segment audio into speech/non-speech regions
- Control pipeline activation (start/stop listening)
- Filter non-speech audio
- Adapt to noise environments

## Non-Goals

- Speaker identification
- Speech recognition
- Audio enhancement

## Architecture Position

VAD is the first processing stage after Audio Capture. It gates the Wake Word and STT stages.

## Inputs

- Audio frames from Audio Capture
- Noise profile (adaptive)
- Sensitivity configuration

## Outputs

- VADEvent (speech start/stop)
- Segmented audio to Wake Word
- VAD metrics

## Interfaces

```rust
pub struct VAD {
    pub fn process_frame(frame: AudioFrame) -> Option<VADEvent>;
    pub fn set_mode(mode: VADMode);
    pub fn reset();
    pub fn get_stats() -> VADStats;
}
```

## Data Models

### VADEvent

| Field | Type | Description |
|-------|------|-------------|
| timestamp | DateTime | Event time |
| type | VADEventType | SpeechStart, SpeechEnd |
| confidence | f32 | Detection confidence |

### VADMode

| Variant | Description |
|---------|-------------|
| `Normal` | Balanced sensitivity |
| `Aggressive` | High sensitivity, more false positives |
| `Quality` | Low sensitivity, fewer false positives |
| `VeryAggressive` | Maximum sensitivity |

### VADStats

| Field | Type | Description |
|-------|------|-------------|
| speech_duration_ms | u64 | Total speech detected |
| silence_duration_ms | u64 | Total silence |
| speech_ratio | f32 | Speech/total ratio |
| adaptation_level | f32 | Noise adaptation level |

## State Management

- VAD state: Idle -> SpeechDetected -> SpeechConfirmed -> SpeechEnd -> Idle
- Adaptive noise profile is maintained

## Threading Model

VAD runs on the audio capture thread. Processing must be real-time.

## IPC Requirements

- No IPC for audio processing
- Mode changes may come via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `vad.speech.start` | VADEvent | Speech started |
| `vad.speech.end` | VADEvent | Speech ended |
| `vad.mode.changed` | VADMode | Mode changed |
| `vad.adapted` | level | Noise profile adapted |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `VD001` | Processing overload | Drop frame, alert |
| `VD002` | Adaptation failure | Reset profile |

## Retry Strategy

- Frame processing: None (real-time)
- Adaptation: Continuous

## Recovery Strategy

- Overload: Reduce processing quality
- Adaptation failure: Reset to default profile

## Security Requirements

- No special security requirements

## Configuration

```json
{
  "vad": {
    "mode": "Normal",
    "frame_duration_ms": 30,
    "speech_timeout_ms": 300,
    "silence_timeout_ms": 500,
    "adaptive": true,
    "adaptation_rate": 0.01
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Detection latency | < 30ms |
| Speech detection accuracy | > 95% |
| CPU usage | < 2% |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Frame buffer | 100KB |
| Noise profile memory | 1MB |

## Benchmarks

- `bench_detection_accuracy`: Precision/recall
- `bench_latency`: Detection delay
- `bench_cpu`: CPU usage

## Testing Requirements

- Accuracy tests with labeled data
- Noise robustness tests
- Mode comparison tests

## Logging Requirements

- Speech events at DEBUG level
- Mode changes at INFO level
- Errors at ERROR level

## Telemetry

- Speech/silence ratio
- Detection latency
- Adaptation level
- False positive rate

## Cross References

- [030_AUDIO_PIPELINE_SPEC.md](030_AUDIO_PIPELINE_SPEC.md)
- [031_AUDIO_CAPTURE_SPEC.md](031_AUDIO_CAPTURE_SPEC.md)

## References

- WebRTC VAD
- Silero VAD
- Google Speech-to-Text VAD
