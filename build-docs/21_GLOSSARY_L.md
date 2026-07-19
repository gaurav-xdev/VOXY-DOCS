# VOXY Glossary — L

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## LLM (Large Language Model)

| Field | Value |
|-------|-------|
| **Category** | AI / Technology |
| **Definition** | A neural network with billions of parameters trained on vast text corpora to understand and generate human language. VOXY runs LLMs locally for privacy and offline operation. |
| **Purpose** | Powers natural language understanding, reasoning, planning, and response generation for complex user queries. |
| **Used By** | MOD-LLM, MOD-NLU, MOD-ACTION |
| **Related Terms** | Transformer, GPT, Token, Inference, Quantization |
| **Related Modules** | MOD-LLM |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY supports multiple local LLM architectures (Llama, Mistral, Phi) via ONNX Runtime and Candle. Models are quantized to Q4/Q5 for memory efficiency. The system selects models based on available RAM and GPU memory. Tool calling enables the LLM to invoke system actions. |

---

## Latency

| Field | Value |
|-------|-------|
| **Category** | Performance / Measurement |
| **Definition** | The time delay between a user's action (speaking a command) and the system's response. VOXY measures latency at multiple points in the voice pipeline. |
| **Purpose** | Primary quality metric for voice-first interaction; low latency is essential for natural user experience. |
| **Used By** | All modules |
| **Related Terms** | Response Time, Throughput, Real-Time, Budget |
| **Related Modules** | MOD-AUDIO, MOD-ASR, MOD-ACTION |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (PERF-001 through PERF-008) |
| **Architecture Notes** | VOXY latency budget: Audio Capture (5-10ms) + VAD (10-20ms) + Wake Word (50-100ms) + ASR (100-200ms) + NLU (20-50ms) + Action (50-200ms) = <500ms total for simple commands. Each module measures and reports its latency via tracing spans. |

---

## Local AI

| Field | Value |
|-------|-------|
| **Category** | Architecture / Principle |
| **Definition** | The design principle that AI inference should run on the user's device rather than in the cloud. VOXY is local-AI-first, with cloud as an optional fallback. |
| **Purpose** | Ensures privacy (data never leaves device), offline functionality, and low latency (no network round-trip). |
| **Used By** | All AI modules |
| **Related Terms** | Edge AI, On-Device ML, Offline First, Privacy |
| **Related Modules** | MOD-LLM, MOD-ASR, MOD-NLU, MOD-WAKE |
| **Related Specifications** | [00_READ_FIRST.md](00_READ_FIRST.md), [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (PRIV-001) |
| **Architecture Notes** | VOXY ships with quantized models optimized for consumer hardware. The system detects available resources (RAM, GPU) and selects appropriate model sizes. Cloud fallback requires explicit user consent per PRIV-004. |

---

## Lock-Free

| Field | Value |
|-------|-------|
| **Category** | Concurrency / Programming |
| **Definition** | A concurrency approach that avoids traditional locking primitives (mutexes) by using atomic operations and memory ordering guarantees. Lock-free algorithms guarantee system-wide progress. |
| **Purpose** | Eliminates deadlock and priority inversion risks in real-time audio processing and high-throughput event handling. |
| **Used By** | MOD-AUDIO, MOD-EVENT |
| **Related Terms** | Atomic, Wait-Free, CAS, Race Condition |
| **Related Modules** | MOD-AUDIO, MOD-EVENT |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY uses lock-free data structures from the crossbeam crate (channels, queues) and atomic types from std::sync::atomic. The audio ring buffer uses CAS (compare-and-swap) operations for producer-consumer coordination. |

---

## Log Mel Spectrogram

| Field | Value |
|-------|-------|
| **Category** | Audio / AI |
| **Definition** | A visual representation of audio signal frequency content over time, using the mel frequency scale (perceptually motivated) and logarithmic amplitude compression. Used as input to speech recognition models. |
| **Purpose** | Converts raw audio waveforms into a feature representation that ASR models can process effectively. |
| **Used By** | MOD-ASR, MOD-WAKE |
| **Related Terms** | FFT, STFT, MFCC, Feature Extraction |
| **Related Modules** | MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY computes log mel spectrograms with 80 mel bins, 25ms window, 10ms hop, and 16kHz sample rate. The computation is performed on the audio thread using FFTW or a Rust FFT implementation. Results are cached for overlapping windows. |

---

*End of 21_GLOSSARY_L.md*