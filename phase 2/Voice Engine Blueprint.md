VOXY Master Blueprint: Voice Engine

Covers: Voice Engine, Audio Pipeline, Latency Budgets

1. Purpose

To deliver Siri-class, sub-500ms conversational latency by bypassing the Windows audio mixer and utilizing heavily optimized, hardware-accelerated streaming inference pipelines.

2. Responsibilities

Microphone Management: Exclusive hardware access, buffer management.

VAD & Barge-in: Instant interruption handling.

Streaming STT/TTS: Chunked audio processing.

3. Architecture

Based on Phase 1, we use WASAPI Exclusive Mode, WebRTC AEC, Silero VAD, streaming Whisper.cpp, and StyleTTS2.

sequenceDiagram
    participant User
    participant WASAPI
    participant AEC_VAD
    participant STT (Whisper)
    participant Core (Router/LLM)
    participant TTS (StyleTTS2)

    User->>WASAPI: Speaks "Stop music"
    WASAPI->>AEC_VAD: 10ms chunk
    AEC_VAD->>STT: Speech detected
    STT->>Core: Stream tokens ("Stop", "music")
    Core->>TTS: Stream tokens ("Stopping", "now")
    TTS->>WASAPI: Stream audio chunks
    WASAPI->>User: Audio playback starts (Total < 500ms)
    
    User->>WASAPI: Interrupts!
    WASAPI->>AEC_VAD: Detects speech despite TTS playing
    AEC_VAD->>Core: Barge-in Event
    Core->>TTS: CANCEL_TOKEN
    TTS--xWASAPI: Flushes buffer immediately


4. Interfaces

IAudioCapture: open_stream(device_id, format), read_frames(buffer).

IVoicePipeline: on_speech_start(), on_speech_end(), interrupt().

5. Data Flow (Latency Budget)

Capture (WASAPI): 10ms

VAD (Silero): 5ms

STT Chunking (Whisper): 150ms

LLM Time-to-First-Token (Llama.cpp): 150ms

TTS First-Chunk (StyleTTS2): 150ms

Total Latency to first audio out: ~465ms (Meets < 500ms criteria).

6. Failure Modes

Exclusive Mode Blocked: Another app (OBS) holds the mic exclusively.

AEC Failure: TTS echoes back into the mic, causing VOXY to talk to itself.

VAD False Positives: Coughs or dog barks triggering the STT loop.

7. Recovery Strategy

Blocked Mic: Fallback to WASAPI Shared Mode automatically (adds ~30ms latency, logs a warning).

AEC Failure: If output audio matches input tightly, trigger a dynamic threshold increase on the VAD.

False Positives: If STT returns [BLANK] or [NOISE], silently abort the pipeline without engaging the LLM.

8. Trade-offs

Exclusive Mode: Breaks legacy apps that don't support audio routing well, but is mandatory for our strict latency budget.

9. Future Extension Points

Multi-microphone beamforming and speaker identification (diarization) to ignore background voices and only respond to the primary user's voice profile.