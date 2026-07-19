SECTION 2.3: Rust Streaming Audio Implementation Blueprint

1. Purpose

To define the low-level Rust implementation for capturing, processing, and rendering audio on Windows using windows-rs and crossbeam lock-free queues, ensuring the strict latency budgets outlined in the blueprint are physically met.

2. Responsibilities

Thread Scheduling: Elevate critical audio threads to bypass Windows OS thread throttling.

Memory Management: Pre-allocate all memory arrays on startup. Zero dynamic memory allocation (Vec::push) allowed inside the hot audio loop.

Model Context Protocol (MCP) Audio Bridging: Allow the LLM to trigger OS sounds (e.g., Siri-like "listening" chimes) concurrently with TTS.

3. Architecture

A. Thread Priority Manipulation

To prevent Windows from starving the audio processing loop when the CPU is under heavy load (e.g., compiling code), the capture and playback threads must be elevated to THREAD_PRIORITY_TIME_CRITICAL.

B. Lock-Free Communication

Mutexes are strictly forbidden in the audio loop. We use crossbeam::channel::bounded and std::sync::atomic.

4. Interfaces

A. Rust Audio Kernel Crate (core/src/audio/engine.rs)

use windows::Win32::Media::Audio::{IAudioClient, AUDCLNT_SHAREMODE_EXCLUSIVE};
use windows::Win32::System::Threading::{SetThreadPriority, GetCurrentThread, THREAD_PRIORITY_TIME_CRITICAL};
use crossbeam::channel::{bounded, Sender, Receiver};
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::Arc;

pub struct VoxyAudioEngine {
    capture_client: IAudioClient,
    render_client: IAudioClient,
    interrupt_flag: Arc<AtomicBool>,
}

impl VoxyAudioEngine {
    /// Initializes WASAPI in Exclusive Mode and elevates thread priority.
    pub fn start_hot_loop(mic_id: &str, speaker_id: &str) -> Result<(), String> {
        unsafe {
            // Elevate Thread Priority
            SetThreadPriority(GetCurrentThread(), THREAD_PRIORITY_TIME_CRITICAL);
        }

        // Lock-free ring buffer for passing PCM frames to AI models
        let (tx, rx) = bounded::<Vec<f32>>(1024); 
        
        std::thread::spawn(move || {
            loop {
                // 1. Read 10ms chunk from WASAPI
                // 2. Apply WebRTC AEC
                // 3. Send to ring buffer
                // tx.send(clean_pcm).unwrap();
            }
        });
        
        Ok(())
    }

    /// Flushes TTS buffers instantly upon user barge-in
    pub fn trigger_barge_in(&self) {
        self.interrupt_flag.store(true, Ordering::SeqCst);
        // Flush render buffers via WASAPI Reset()
    }
}


5. Data Flow (Predictive LLM to TTS Streaming)

To beat Siri, we cannot wait for a full sentence.

Llama.cpp streams tokens: ["I", "can", "help", "with", "that", "."]

A Rust background task aggregates tokens until it hits a valid punctuation mark or chunk boundary (e.g., ["I can help"]).

This partial string is immediately dispatched to the StyleTTS2 ONNX model.

StyleTTS2 generates the audio for "I can help" and pushes it to the WASAPI render buffer.

While "I can help" is playing, Llama.cpp is already generating the next tokens, and StyleTTS2 is synthesizing "with that."

6. Trade-offs

WASAPI Exclusive Mode vs OS Convenience: Exclusive mode means other apps cannot record the user's microphone while VOXY is running. We accept this trade-off for the massive reduction in latency. If users complain, we will implement a virtual audio driver loopback.

VAD Aggressiveness vs False Positives: Setting Silero VAD to be highly sensitive allows for instant barge-in, but might accidentally trigger if a dog barks loudly in the room.

7. Future Extension Points

Speaker Diarization: Integrating a lightweight pyannote-style ONNX model to reject voices that do not match the primary user's voice print, ensuring VOXY isn't hijacked by a TV playing in the background.