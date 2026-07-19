# 030 - Audio Pipeline Specification

## Purpose

Define the audio pipeline orchestration subsystem that coordinates all audio processing stages from capture to output, managing the flow of audio data through VAD, wake word detection, STT, and TTS.

## Scope

- Audio pipeline topology and data flow
- Stage coordination and buffering
- Pipeline state management
- Audio format negotiation
- Latency budgeting

## Responsibilities

- Coordinate audio capture, processing, and output
- Manage pipeline state (idle, listening, processing, speaking)
- Buffer audio between stages
- Handle pipeline errors and recovery
- Provide pipeline diagnostics

## Non-Goals

- Audio device management (Audio Capture subsystem)
- Speech recognition algorithms (STT subsystem)
- Voice synthesis (TTS subsystem)
- Wake word model (Wake Word subsystem)

## Architecture Position

The Audio Pipeline sits above individual audio subsystems, orchestrating their interactions.

## Pipeline Topology

```
[Audio Capture] -> [VAD] -> [Wake Word] -> [STT] -> [Cognitive Engine]
                                                          |
[TTS] <- [Barge-In] <------------------------------------+
```

## Inputs

- Raw audio frames from Audio Capture
- TTS requests from Cognitive Engine
- Pipeline control commands
- Configuration updates

## Outputs

- Transcribed text to Cognitive Engine
- Synthesized audio to audio output
- Pipeline state events
- Latency metrics

## Interfaces

```rust
pub struct AudioPipeline {
    pub fn initialize() -> Result<PipelineHandle, PipelineError>;
    pub fn start_listening() -> Result<(), PipelineError>;
    pub fn stop_listening() -> Result<(), PipelineError>;
    pub fn synthesize(request: TTSRequest) -> Result<AudioStream, PipelineError>;
    pub fn get_state() -> PipelineState;
    pub fn get_latency_budget() -> LatencyReport;
}
```

## Data Models

### PipelineState

| Variant | Description |
|---------|-------------|
| `Idle` | No audio activity |
| `Listening` | Capturing audio, VAD active |
| `WakeDetected` | Wake word triggered, STT active |
| `Processing` | STT transcribing |
| `Speaking` | TTS output active |
| `BargeIn` | User interrupted TTS |

### LatencyReport

| Stage | Budget | Actual |
|-------|--------|--------|
| Capture | 50ms | measured |
| VAD | 30ms | measured |
| Wake Word | 100ms | measured |
| STT | 300ms | measured |
| TTS | 200ms | measured |

## State Management

Pipeline state is centralized and protected by a state machine. State transitions are atomic and broadcast via Event Bus.

## Threading Model

Audio processing runs on real-time-priority threads. Orchestration runs on Tokio runtime.

## IPC Requirements

- No IPC for audio data (in-process)
- Control commands may come via IPC from external tools

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `audio.pipeline.state` | PipelineState | State transition |
| `audio.pipeline.latency` | LatencyReport | Latency measurements |
| `audio.pipeline.error` | PipelineError | Pipeline error |
| `audio.pipeline.buffer_status` | BufferLevel | Buffer health |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `AP001` | Stage initialization failure | Retry once, then disable stage |
| `AP002` | Buffer overflow | Drop oldest frames, alert |
| `AP003` | Format mismatch | Reinitialize with negotiated format |
| `AP004` | Latency budget exceeded | Alert, continue with degraded quality |

## Retry Strategy

- Stage init: 1 retry
- Format negotiation: 3 retries

## Recovery Strategy

- Stage failure: Bypass stage if non-critical
- Buffer overflow: Reduce quality, increase compression
- Latency exceed: Alert, do not stop

## Security Requirements

- Audio data is not encrypted in-process
- Audio capture requires user consent (Windows privacy)
- Loopback capture requires appropriate capabilities

## Configuration

```json
{
  "audio_pipeline": {
    "sample_rate": 16000,
    "channels": 1,
    "format": "f32",
    "buffer_size_ms": 100,
    "latency_budget_ms": 500,
    "enable_barge_in": true,
    "stt_timeout_seconds": 10,
    "tts_timeout_seconds": 30
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| End-to-end latency | < 500ms |
| Pipeline throughput | Real-time (no drop) |
| State transition | < 10ms |
| Buffer underrun | 0 |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Buffer size | 10MB |
| Concurrent streams | 2 (input + output) |
| Latency budget | 500ms total |

## Benchmarks

- `bench_pipeline_latency`: End-to-end latency
- `bench_buffer_stability`: No underruns under load
- `bench_state_transition`: State change latency

## Testing Requirements

- State machine property tests
- Latency budget tests
- Buffer overflow recovery tests
- Stage failure injection tests

## Logging Requirements

- State transitions at INFO level
- Latency exceed at WARN level
- Errors at ERROR level
- Buffer status at DEBUG level

## Telemetry

- Pipeline state duration
- Per-stage latency
- Buffer levels
- Drop rate

## Cross References

- [031_AUDIO_CAPTURE_SPEC.md](031_AUDIO_CAPTURE_SPEC.md)
- [032_WAKE_WORD_SPEC.md](032_WAKE_WORD_SPEC.md)
- [033_VAD_SPEC.md](033_VAD_SPEC.md)
- [034_STT_SPEC.md](034_STT_SPEC.md)
- [035_TTS_SPEC.md](035_TTS_SPEC.md)
- [036_BARGE_IN_SPEC.md](036_BARGE_IN_SPEC.md)

## References

- WASAPI Audio Pipeline Design
- Real-Time Audio Processing
- Windows Audio Architecture
