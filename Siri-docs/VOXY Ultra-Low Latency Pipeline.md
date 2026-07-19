SECTION 2.2: Beating Siri: VOXY's Sub-400ms Voice Pipeline

1. Purpose

To design a voice pipeline that achieves "human-level" conversational latency (Time-To-First-Audio $< 400\text{ms}$) natively on Windows. This eliminates the "walkie-talkie" feel of traditional AI voice agents.

2. Responsibilities

Zero-Copy Audio Buffering: Move audio frames from the hardware driver to the neural nets without memory allocations.

Streaming Inference: Pipeline STT -> LLM -> TTS so that output generation overlaps with input processing.

Instant Barge-in: Allow the user to interrupt VOXY mid-sentence with $< 50\text{ms}$ reaction time.

3. Architecture

A. The VOXY Streaming Audio Pipeline

This architecture utilizes a single, lock-free ring buffer. All AI models operate on sliding windows over this buffer.

graph TD
    subgraph Audio Capture [Windows Kernel]
        A[Microphone Driver] -->|WASAPI Exclusive| B(Rust Audio Thread)
    end

    subgraph Lock-Free Ring Buffer [10 Seconds of Audio]
        B --> C[(Raw Audio Buffer)]
        E[WebRTC AEC & RNNoise] -->|Transforms| C
    end

    subgraph Neural Pipeline [Inference Threads]
        C --> D{openWakeWord ONNX}
        D -->|Triggered| F[Silero VAD]
        F -->|Active Speech| G[Whisper.cpp Streaming]
        G -->|Token Stream| H[Llama.cpp / Phi-3]
        H -->|Token Stream| I[StyleTTS2 ONNX]
    end

    subgraph Playback [Windows Kernel]
        I -->|Audio Chunks| J(Rust Playback Thread)
        J -->|WASAPI Exclusive| K[Speaker Driver]
        J -.->|AEC Reference| E
    end


B. The Mathematical Latency Budget

To guarantee a Siri-beating experience, we enforce strict latency budgets for each chunk of processing. Let $T_{total}$ be the Time-To-First-Audio.


$$T_{total} = T_{mic} + T_{vad} + T_{stt} + T_{llm} + T_{tts} + T_{spk}$$


Where:

$T_{mic}$ (Capture Buffer): $10\text{ms}$

$T_{vad}$ (Silero): $5\text{ms}$

$T_{stt}$ (Whisper Partial transcription): $150\text{ms}$

$T_{llm}$ (Time to first LLM token): $100\text{ms}$

$T_{tts}$ (Time to render first 500ms TTS chunk): $80\text{ms}$

$T_{spk}$ (Playback Buffer): $10\text{ms}$

$T_{total} = 355\text{ms}$ (Well below the $500\text{ms}$ human perception threshold for "instant" response).

4. Data Flow (The "Barge-In" Interruption)

VOXY is currently speaking via TTS.

The user speaks over VOXY.

WebRTC AEC subtracts the TTS audio from the incoming mic audio.

Silero VAD detects sudden human vocal energy.

The Audio Thread broadcasts a VOICE_INTERRUPT event.

The TTS buffer is instantly flushed via AtomicBool flags.

The LLM context is updated with [User Interrupted].

5. Failure Modes

Hardware Exclusive Mode Blocked: Another application (like a video game or Zoom) has taken exclusive control of the microphone.

Buffer Underrun: The TTS engine cannot generate audio fast enough to keep the playback buffer full, resulting in robotic stuttering/crackling.

6. Recovery Strategy

Fallback to Shared Mode: If IAudioClient::Initialize fails with AUDCLNT_E_EXCLUSIVE_MODE_NOT_ALLOWED, VOXY instantly falls back to WASAPI Shared Mode. This adds $\approx 30\text{ms}$ of latency but guarantees functionality.

Adaptive TTS Buffering: If an underrun occurs, the Rust playback thread increases the minimum TTS pre-buffer size from $100\text{ms}$ to $250\text{ms}$ for the remainder of the session.