# VOXY Glossary — B

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## Backend

| Field | Value |
|-------|-------|
| **Category** | Architecture |
| **Definition** | The server-side or core processing component of a software system. In VOXY, the backend refers to the Rust-based engine modules that process voice input, run AI inference, and execute system commands. |
| **Purpose** | Provides the computational and logical foundation for the VOXY platform, independent of the UI presentation layer. |
| **Used By** | All engine modules |
| **Related Terms** | Frontend, Engine, Service |
| **Related Modules** | MOD-AUDIO, MOD-ASR, MOD-LLM, MOD-ACTION |
| **Related Specifications** | [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md) |
| **Architecture Notes** | VOXY's backend is composed of 20 Rust modules organized in layers. The backend communicates with the WinUI 3 frontend via IPC (gRPC over named pipes). All backend processing happens locally on the user's device. |

---

## Benchmark

| Field | Value |
|-------|-------|
| **Category** | Process / Quality |
| **Definition** | A standardized test that measures the performance of a system or component against defined criteria. VOXY uses Criterion.rs for statistical benchmarking with regression detection. |
| **Purpose** | Ensures performance targets are met and detects regressions before they reach production. |
| **Used By** | MOD-ASR, MOD-LLM, MOD-TTS, MOD-AUDIO |
| **Related Terms** | Performance Test, Latency Budget, Regression |
| **Related Modules** | All performance-critical modules |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | Benchmarks run in CI on every PR for performance-critical modules. Results are stored in target/benchmarks/ and compared against baseline. A regression >10% blocks merge. |

---

## Binding

| Field | Value |
|-------|-------|
| **Category** | Programming / Interop |
| **Definition** | A software layer that allows code written in one language to call functions or use libraries written in another language. In VOXY, bindings connect Rust to Windows APIs, ONNX Runtime, and audio libraries. |
| **Purpose** | Enables VOXY to leverage platform-specific capabilities and optimized libraries while maintaining a Rust-centric codebase. |
| **Used By** | MOD-WINAPI, MOD-ASR, MOD-LLM, MOD-AUDIO |
| **Related Terms** | FFI, Interop, Wrapper, API Projection |
| **Related Modules** | MOD-WINAPI, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md) |
| **Architecture Notes** | VOXY uses windows-rs for Win32/WinRT bindings, ort for ONNX Runtime, and whisper-rs for Whisper.cpp. All bindings are wrapped in safe Rust abstractions with error handling. |

---

## Buffer (Audio)

| Field | Value |
|-------|-------|
| **Category** | Audio / Data Structure |
| **Definition** | A contiguous block of memory used to store audio samples temporarily during capture, processing, or playback. VOXY uses lock-free ring buffers for real-time audio streaming. |
| **Purpose** | Decouples audio capture rate from processing rate, enabling asynchronous operation between the audio hardware and AI inference pipelines. |
| **Used By** | MOD-AUDIO, MOD-ASR, MOD-WAKE, MOD-TTS |
| **Related Terms** | Ring Buffer, Sample, Frame, Latency |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY uses crossbeam channels and custom ring buffers for audio data flow. Buffers are sized to hold 100-200ms of audio at the target sample rate, balancing latency against processing efficiency. |

---

## Bus (Event Bus)

| Field | Value |
|-------|-------|
| **Category** | Architecture / Pattern |
| **Definition** | A messaging infrastructure that enables publish-subscribe communication between decoupled components. VOXY's Event Bus uses Tokio broadcast channels for async event distribution. |
| **Purpose** | Enables loose coupling between modules, allowing components to communicate without direct dependencies. |
| **Used By** | All modules |
| **Related Terms** | Pub/Sub, Message Queue, Event-Driven Architecture |
| **Related Modules** | MOD-EVENT |
| **Related Specifications** | [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md), [08_MODULE_TEMPLATE.md](08_MODULE_TEMPLATE.md) |
| **Architecture Notes** | The Event Bus supports typed events through generics. Subscribers receive events asynchronously via Tokio channels. Backpressure is handled through bounded channels with configurable buffer sizes. |

---

*End of 11_GLOSSARY_B.md*
