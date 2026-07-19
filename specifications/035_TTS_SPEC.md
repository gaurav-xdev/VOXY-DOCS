# 035 - TTS (Text-to-Speech) Specification

## Purpose

Define the Text-to-Speech subsystem that converts text responses from the Cognitive Engine into natural-sounding speech audio for playback to the user.

## Scope

- Text-to-speech synthesis
- Multiple voice and provider support
- Streaming audio output
- Voice selection and customization
- SSML support
- Barge-in compatibility

## Responsibilities

- Synthesize text into audio
- Support multiple TTS providers
- Stream audio chunks for low latency
- Handle voice selection
- Support SSML for advanced control
- Coordinate with Barge-In for interruption

## Non-Goals

- Audio playback (Audio Pipeline handles output)
- Voice cloning (future enhancement)
- Real-time voice conversion

## Architecture Position

TTS receives text from Cognitive Engine and produces audio for the Audio Pipeline to output.

## Inputs

- Text strings from Cognitive Engine
- Voice selection preferences
- SSML markup
- Speed/pitch preferences

## Outputs

- Audio chunks to Audio Pipeline
- Synthesis status events
- Error events

## Interfaces

```rust
pub struct TTS {
    pub fn synthesize(request: TTSRequest) -> Result<AudioStream, TTSError>;
    pub fn list_voices() -> Vec<Voice>;
    pub fn preview_voice(voice_id: &str) -> Result<AudioStream, TTSError>;
    pub fn stop_synthesis();
}
```

## Data Models

### TTSRequest

| Field | Type | Description |
|-------|------|-------------|
| text | String | Text to synthesize |
| voice_id | String | Selected voice |
| speed | f32 | Speed multiplier (0.5-2.0) |
| pitch | f32 | Pitch adjustment |
| ssml | bool | Text is SSML |
| stream | bool | Stream chunks |

### Voice

| Field | Type | Description |
|-------|------|-------------|
| id | String | Voice identifier |
| name | String | Human-readable name |
| language | String | Supported language |
| gender | String | Voice gender |
| provider | TTSProvider | Source provider |
| preview_url | Option<String> | Preview audio URL |

### TTSProvider

| Variant | Description |
|---------|-------------|
| `OpenAI` | OpenAI TTS API |
| `Azure` | Azure Speech Services |
| `Windows` | Windows OneCore Voices |
| `Local` | Local ONNX TTS model |

## State Management

- Synthesis state: Idle -> Synthesizing -> Streaming -> Completed -> Idle
- Barge-in state: Speaking -> Interrupted -> Idle

## Threading Model

Cloud providers use async HTTP streaming. Local models use blocking inference. Audio output runs on dedicated thread.

## IPC Requirements

- No IPC for audio
- Voice list may be queried via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `tts.started` | request_id | Synthesis started |
| `tts.chunk` | (id, chunk_index) | Audio chunk ready |
| `tts.completed` | request_id | Synthesis completed |
| `tts.stopped` | reason | Synthesis stopped |
| `tts.error` | TTSError | Synthesis error |
| `tts.interrupted` | request_id | Barge-in interrupted |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `TT001` | API timeout | Retry, fallback |
| `TT002` | Voice not found | Use default voice |
| `TT003` | SSML parse error | Fallback to plain text |
| `TT004` | Audio output failure | Retry output |
| `TT005` | Rate limit | Backoff, fallback |

## Retry Strategy

- API call: 3 retries
- Audio output: 2 retries
- Provider fallback: Immediate

## Recovery Strategy

- Cloud failure: Use Windows voices
- All failure: Text-only response

## Security Requirements

- API keys in Secret Storage
- No text logging without consent

## Configuration

```json
{
  "tts": {
    "primary_provider": "Azure",
    "fallback_provider": "Windows",
    "default_voice": "en-US-AriaNeural",
    "speed": 1.0,
    "pitch": 0.0,
    "enable_ssml": true,
    "stream_chunks": true,
    "chunk_size_ms": 200,
    "api_timeout_seconds": 15
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| First chunk latency | < 300ms |
| Full synthesis (50 tokens) | < 2s |
| Audio quality | > 4.0 MOS |
| CPU usage (local) | < 15% |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Model memory | 200MB |
| Concurrent synthesis | 1 |
| Max text length | 4000 chars |

## Benchmarks

- `bench_first_chunk`: Time to first audio |
- `bench_full_synthesis`: Total synthesis time |
- `bench_quality`: MOS scoring |

## Testing Requirements

- Quality tests with listeners
- Provider fallback tests
- SSML parsing tests
- Barge-in interruption tests

## Logging Requirements

- Synthesis events at INFO level
- Errors at ERROR level
- No text logging (privacy)

## Telemetry

- Synthesis latency
- Provider usage
- Chunk delivery rate
- Interruption frequency

## Cross References

- [030_AUDIO_PIPELINE_SPEC.md](030_AUDIO_PIPELINE_SPEC.md)
- [034_STT_SPEC.md](034_STT_SPEC.md)
- [036_BARGE_IN_SPEC.md](036_BARGE_IN_SPEC.md)

## References

- OpenAI TTS API
- Azure Speech Services TTS
- Windows OneCore Voices
- ONNX Runtime TTS
