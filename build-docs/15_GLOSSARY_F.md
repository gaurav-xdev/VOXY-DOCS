# VOXY Glossary — F

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## Feature Flag

| Field | Value |
|-------|-------|
| **Category** | Development / Deployment |
| **Definition** | A mechanism for enabling or disabling functionality at runtime without deploying new code. VOXY uses feature flags for gradual rollouts, A/B testing, and emergency toggles. |
| **Purpose** | Reduces deployment risk by allowing features to be activated/deactivated without code changes, and enables experimentation with user cohorts. |
| **Used By** | MOD-CONFIG, MOD-TELEM, MOD-UI |
| **Related Terms** | Toggle, A/B Test, Gradual Rollout, Canary |
| **Related Modules** | MOD-CONFIG |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) |
| **Architecture Notes** | VOXY feature flags are stored in the local configuration database and checked at runtime. Flags have three states: disabled, enabled-for-percentage, and fully-enabled. Changes take effect without restart for most flags. |

---

## FFI (Foreign Function Interface)

| Field | Value |
|-------|-------|
| **Category** | Programming / Interop |
| **Definition** | A mechanism that allows code written in one programming language to call functions written in another. VOXY uses FFI extensively for Windows API calls and ML library integration. |
| **Purpose** | Enables VOXY to leverage C/C++ libraries (ONNX Runtime, Whisper.cpp, Windows APIs) from Rust code. |
| **Used By** | MOD-WINAPI, MOD-ASR, MOD-LLM, MOD-AUDIO |
| **Related Terms** | Binding, Interop, ABI, extern "C" |
| **Related Modules** | MOD-WINAPI, MOD-ASR |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md), [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md) |
| **Architecture Notes** | VOXY uses bindgen for generating FFI bindings from C headers and windows-rs for Win32/WinRT. All FFI calls are wrapped in safe Rust abstractions. Unsafe blocks require SAFETY comments per [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md). |

---

## Frame (Audio)

| Field | Value |
|-------|-------|
| **Category** | Audio / Data |
| **Definition** | A fixed-size block of audio samples representing a short time window (typically 10-30ms). Frames are the basic unit of processing for VAD, ASR, and audio effects. |
| **Purpose** | Enables efficient batch processing of audio data while maintaining low latency. |
| **Used By** | MOD-AUDIO, MOD-ASR, MOD-WAKE, MOD-VAD |
| **Related Terms** | Sample, Buffer, Window, Hop Size |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY processes audio in 20ms frames at 16kHz (320 samples per frame). Frames overlap by 50% (10ms hop size) for spectral analysis. The audio pipeline maintains a frame buffer that feeds the VAD and ASR engines. |

---

## Fallback

| Field | Value |
|-------|-------|
| **Category** | Reliability / Pattern |
| **Definition** | A secondary mechanism that activates when the primary mechanism fails. VOXY implements fallback chains for AI models, audio backends, and command execution. |
| **Purpose** | Ensures system functionality degrades gracefully rather than failing completely when components are unavailable. |
| **Used By** | MOD-ASR, MOD-LLM, MOD-AUDIO, MOD-ACTION |
| **Related Terms** | Graceful Degradation, Backup, Redundancy, Circuit Breaker |
| **Related Modules** | MOD-ACTION, MOD-ASR |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (REL-005) |
| **Architecture Notes** | VOXY fallback chains: ASR (Whisper.cpp -> Vosk -> Windows Speech), LLM (local ONNX -> Candle -> cloud with consent), Audio (WASAPI -> ASIO -> DirectSound). Each fallback is logged for telemetry. |

---

## Fuzzing

| Field | Value |
|-------|-------|
| **Category** | Security / Testing |
| **Definition** | An automated testing technique that provides invalid, unexpected, or random data as input to a program to find crashes, memory leaks, or security vulnerabilities. |
| **Purpose** | Discovers edge cases and vulnerabilities that manual testing misses, particularly for audio parsing and input sanitization. |
| **Used By** | MOD-AUDIO, MOD-ASR, MOD-SEC |
| **Related Terms** | Property-Based Testing, Chaos Engineering, AFL, libFuzzer |
| **Related Modules** | MOD-AUDIO, MOD-ASR |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY uses cargo-fuzz with libFuzzer for fuzzing audio decoders and input parsers. Fuzzing runs in CI for 5 minutes per target on every release build. Targets include: audio format parsing, transcript sanitization, and configuration parsing. |

---

*End of 15_GLOSSARY_F.md*
