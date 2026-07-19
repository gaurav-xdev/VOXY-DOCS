# 034 - STT (Speech-to-Text) Specification

## Purpose

Define the Speech-to-Text subsystem that converts captured audio into transcribed text, supporting both cloud APIs and local inference models.

## Scope

- Real-time streaming speech recognition
- Cloud API integration (OpenAI, Azure, etc.)
- Local model inference (ONNX, Whisper)
- Language detection
- Partial and final transcript generation
- Punctuation and formatting

## Responsibilities

- Transcribe audio streams to text
- Support multiple STT providers
- Handle language switching
- Provide partial (interim) and final transcripts
- Manage API keys and quotas
- Fallback between providers

## Non-Goals

- Speaker diarization
- Real-time translation
- Audio preprocessing (handled by Audio Pipeline)

## Architecture Position

STT receives audio from the Audio Pipeline after wake word detection and produces text for the Cognitive Engine.

## Inputs

- Audio frames from Audio Pipeline
- Provider configuration
- Language hints
- API credentials

## Outputs

- STTResult (partial and final transcripts)
- Language detection results
- Confidence scores
- Error events

## Interfaces

```rust
pub struct STT {
    pub fn start_stream(config: STTConfig) -> Result<STTStream, STTError>;
    pub fn process_audio(stream: &STTStream, audio: AudioFrame);
    pub fn end_stream(stream: STTStream) -> Result<STTResult, STTError>;
    pub fn transcribe_file(path: &str) -> Result<STTResult, STTError>;
}
```

## Data Models

### STTResult

| Field | Type | Description |
|-------|------|-------------|
| transcript | String | Full transcript |
| is_final | bool | True if final result |
| confidence | f32 | Overall confidence |
| words | Vec<WordInfo> | Per-word timing and confidence |
| language | String | Detected language |
| duration_ms | u64 | Audio duration |

### WordInfo

| Field | Type | Description |
|-------|------|-------------|
| word | String | Word text |
| start_ms | u64 | Start time in audio |
| end_ms | u64 | End time in audio |
| confidence | f32 | Word confidence |

### STTConfig

| Field | Type | Description |
|-------|------|-------------|
| provider | STTProvider | Cloud or local |
| language | Option<String> | Expected language |
| model | String | Model identifier |
| enable_punctuation | bool | Add punctuation |
| enable_timestamps | bool | Include word timings |

### STTProvider

| Variant | Description |
|---------|-------------|
| `OpenAI` | OpenAI Whisper API |
| `Azure` | Azure Speech Services |
| `LocalWhisper` | Local Whisper ONNX model |
| `WindowsSpeech` | Windows Speech Recognition |

## State Management

- Stream state: Idle -> Active -> Processing -> Completed
- Provider health is tracked for fallback

## Threading Model

Cloud providers use async HTTP streaming. Local models use blocking inference on dedicated threads.

## IPC Requirements

- No IPC for audio/text
- Provider configuration may be updated via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `stt.partial` | STTResult | Interim transcript |
| `stt.final` | STTResult | Final transcript |
| `stt.error` | STTError | Recognition error |
| `stt.provider.switched` | (old, new) | Provider fallback |
| `stt.language.detected` | language | Language detected |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `ST001` | API timeout | Retry, then fallback |
| `ST002` | API rate limit | Backoff, fallback |
| `ST003` | Model load failure | Fallback to cloud |
| `ST004` | Audio format error | Reinitialize |
| `ST005` | Network failure | Use local model |

## Retry Strategy

- API call: 3 retries, exponential backoff
- Local inference: 1 retry
- Provider fallback: Immediate

## Recovery Strategy

- Cloud failure: Switch to local model
- Local failure: Switch to cloud
- Both failure: Alert user, stop listening

## Security Requirements

- API keys stored in Secret Storage (092)
- Audio is not logged or persisted
- Cloud API calls use HTTPS

## Configuration

```json
{
  "stt": {
    "primary_provider": "LocalWhisper",
    "fallback_provider": "OpenAI",
    "language": "auto",
    "model": "whisper-base",
    "api_timeout_seconds": 10,
    "max_retries": 3,
    "enable_partial": true,
    "partial_interval_ms": 500
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| First partial latency | < 500ms |
| Final latency (5s audio) | < 2s |
| Word error rate | < 10% |
| CPU usage (local) | < 20% |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Model memory | 500MB |
| Concurrent streams | 1 |
| API timeout | 10s |

## Benchmarks

- `bench_latency`: End-to-end transcription
- `bench_accuracy`: WER on test set
- `bench_fallback`: Provider switch time

## Testing Requirements

- Accuracy tests with standard datasets
- Provider fallback tests
- Language detection tests
- Timeout handling tests

## Logging Requirements

- Transcripts at DEBUG level (redact PII)
- Provider switches at INFO level
- Errors at ERROR level
- API calls at TRACE level

## Telemetry

- Transcription latency
- Provider usage
- Word error rate
- API quota usage

## Cross References

- [030_AUDIO_PIPELINE_SPEC.md](030_AUDIO_PIPELINE_SPEC.md)
- [035_TTS_SPEC.md](035_TTS_SPEC.md)
- [092_SECRET_STORAGE_SPEC.md](092_SECRET_STORAGE_SPEC.md)

## References

- OpenAI Whisper API
- Azure Speech Services
- ONNX Runtime Whisper
- Windows Speech Recognition
