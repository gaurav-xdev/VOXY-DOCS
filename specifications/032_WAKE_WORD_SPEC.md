# 032 - Wake Word Specification

## Purpose

Define the wake word detection subsystem that continuously monitors audio input for a predefined activation phrase, triggering the full voice pipeline upon detection.

## Scope

- Wake word model inference
- Audio stream processing
- Detection confidence scoring
- False positive mitigation
- Multiple wake word support

## Responsibilities

- Process audio stream for wake word presence
- Provide confidence scores
- Trigger pipeline activation
- Support custom wake words
- Minimize false activations

## Non-Goals

- Speech recognition (STT subsystem)
- Voice activity detection (VAD subsystem)
- Custom model training (external process)

## Architecture Position

Wake Word sits between VAD and STT in the audio pipeline. It is the gatekeeper for voice activation.

## Inputs

- Audio frames from VAD (post-VAD audio)
- Wake word model files
- Sensitivity configuration

## Outputs

- WakeWordEvent to Audio Pipeline
- Confidence scores
- Detection metrics

## Interfaces

```rust
pub struct WakeWordEngine {
    pub fn load_model(path: &str) -> Result<(), WakeError>;
    pub fn process_frame(frame: AudioFrame) -> Option<WakeWordEvent>;
    pub fn set_sensitivity(level: f32);
    pub fn get_active_model() -> String;
}
```

## Data Models

### WakeWordEvent

| Field | Type | Description |
|-------|------|-------------|
| timestamp | DateTime | Detection time |
| model_name | String | Detected wake word model |
| confidence | f32 | Detection confidence (0.0-1.0) |
| audio_context | Vec<AudioFrame> | Preceding audio buffer |

## State Management

- Model state: Unloaded -> Loading -> Ready
- Detection state: Idle -> Listening -> Detected -> Cooldown

## Threading Model

Wake word inference runs on a dedicated thread with real-time priority. Model loading runs on blocking pool.

## IPC Requirements

- No IPC for audio processing
- Model updates may come via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `wake.detected` | WakeWordEvent | Wake word detected |
| `wake.model.loaded` | model_name | Model loaded |
| `wake.model.failed` | (name, error) | Model load failed |
| `wake.sensitivity.changed` | level | Sensitivity changed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `WW001` | Model load failure | Use default model |
| `WW002` | Inference failure | Skip frame, retry next |
| `WW003` | Invalid model format | Reject, alert |

## Retry Strategy

- Model load: 3 retries
- Inference: Skip frame, continue

## Recovery Strategy

- Model failure: Fallback to built-in model
- Inference failure: Continue processing

## Security Requirements

- Models are loaded from trusted paths only
- Custom models are validated before loading
- No network loading of models

## Configuration

```json
{
  "wake_word": {
    "model_path": "%LOCALAPPDATA%/VOXY/models/wake/",
    "default_model": "voxy_default.onnx",
    "sensitivity": 0.7,
    "min_confidence": 0.5,
    "cooldown_ms": 2000,
    "audio_context_ms": 500,
    "enable_custom_models": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Detection latency | < 200ms |
| False acceptance rate | < 1 per hour |
| False rejection rate | < 5% |
| CPU usage (idle) | < 5% |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Model memory | 50MB |
| Inference time per frame | < 10ms |
| Active models | 5 |

## Benchmarks

- `bench_detection_latency`: Time to detection
- `bench_false_positive`: False activation rate
- `bench_cpu_usage`: CPU under continuous listening

## Testing Requirements

- Detection accuracy tests
- False positive/negative tests
- Model loading tests
- Sensitivity tuning tests

## Logging Requirements

- Detections at INFO level
- Model loads at INFO level
- Errors at ERROR level
- Inference timing at TRACE level

## Telemetry

- Detection count
- Confidence distribution
- False positive rate
- Inference latency

## Cross References

- [030_AUDIO_PIPELINE_SPEC.md](030_AUDIO_PIPELINE_SPEC.md)
- [031_AUDIO_CAPTURE_SPEC.md](031_AUDIO_CAPTURE_SPEC.md)
- [033_VAD_SPEC.md](033_VAD_SPEC.md)

## References

- ONNX Runtime
- Porcupine Wake Word Engine
- Snowboy Hotword Detection
- Windows Voice Activation
