# VOXY — Technology Stack

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Technology Foundation |

---

## Purpose

This document defines every technology, library, framework, and tool used in the VOXY platform. Each entry includes selection rationale, version pinning, compatibility notes, and replacement criteria.

---

## Scope

Covers:
- Programming languages and runtimes
- Core frameworks and SDKs
- AI/ML libraries and model formats
- Audio processing libraries
- Build and development tools
- Testing frameworks
- CI/CD infrastructure

Does not cover:
- Implementation patterns (see [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md))
- Module architecture (see [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md))
- Windows API specifics (see [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md))

---

## Audience

- Principal Engineers evaluating technology choices
- Engineers adding new dependencies
- DevOps configuring build environments
- AI Coding Agents selecting appropriate libraries

---

## Technology Selection Criteria

Every technology in the VOXY stack must satisfy:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **Windows Compatibility** | Critical | Must work natively on Windows 10/11 |
| **Offline Capability** | Critical | Must function without internet connectivity |
| **Performance** | High | Must meet latency targets (<200ms for voice commands) |
| **Memory Efficiency** | High | Must operate within 8GB RAM budget for AI models |
| **License Compatibility** | Critical | Must be MIT/Apache-2.0 or permissive |
| **Maintenance Status** | High | Actively maintained, responsive maintainers |
| **Documentation Quality** | Medium | Comprehensive docs or source clarity |
| **Community Size** | Medium | Sufficient for issue resolution |
| **Security Track Record** | Critical | No unpatched CVEs, regular audits |

---

## Programming Languages

### Rust (Primary)

| Property | Value |
|----------|-------|
| **Version** | 1.85.0 (stable) |
| **Edition** | 2024 |
| **Rationale** | Memory safety without GC, zero-cost abstractions, excellent Windows FFI, async/await, cargo ecosystem |
| **Used By** | All backend modules (MOD-AUDIO through MOD-UPDATE) |
| **Replacement Criteria** | If Windows-specific async runtime limitations block critical features |

**Toolchain Components:**
- `rustc` — Compiler
- `cargo` — Build system and package manager
- `rustfmt` — Code formatting
- `clippy` — Linting
- `rust-analyzer` — LSP server

**Required Cargo Tools:**
```toml
[workspace.metadata.tools]
cargo-nextest = "0.9"
cargo-deny = "0.16"
cargo-audit = "0.21"
cargo-fuzz = "0.12"
cargo-machete = "0.7"
```

### C/C++ (Interop and Performance)

| Property | Value |
|----------|-------|
| **Compiler** | MSVC 19.40+ (Visual Studio 2022) |
| **Standard** | C++20 |
| **Rationale** | Windows COM interop, DirectML integration, ONNX Runtime C++ API, legacy Win32 APIs |
| **Used By** | MOD-WINAPI, MOD-UIAUTO, MOD-ASR (ONNX bindings), MOD-LLM (inference backends) |
| **Replacement Criteria** | If pure Rust alternatives mature for all Windows APIs |

### C# / XAML (UI Layer)

| Property | Value |
|----------|-------|
| **Runtime** | .NET 9 |
| **UI Framework** | WinUI 3 (Windows App SDK 2.0+) |
| **Rationale** | Native Windows UI, fluent design system, XAML data binding, WinRT projection |
| **Used By** | MOD-UI only |
| **Replacement Criteria** | If WinUI 4 introduces breaking changes requiring migration |

### Python (Tooling and Scripts)

| Property | Value |
|----------|-------|
| **Version** | 3.12+ |
| **Rationale** | Model conversion, dataset preprocessing, benchmarking scripts |
| **Used By** | Build scripts, model quantization, test data generation |
| **Replacement Criteria** | If all tooling can be rewritten in Rust |

---

## Core Frameworks and SDKs

### Windows App SDK 2.0+

| Property | Value |
|----------|-------|
| **Namespace** | `Microsoft.WindowsAppSDK` |
| **Rationale** | Modern Windows app packaging, WinUI 3, App Lifecycle, Notifications |
| **Used By** | MOD-UI, MOD-UPDATE |
| **Min Version** | 2.0.0 |
| **Notes** | Packaged with MSIX; unpackaged mode supported for development |

### WinRT / Windows Runtime

| Property | Value |
|----------|-------|
| **Rationale** | Modern Windows API surface, language projection, async operations |
| **Used By** | MOD-WINAPI, MOD-UIAUTO, MOD-APPCTL |
| **Access Pattern** | Rust via `windows-rs` crate; C# via built-in projection |

### .NET 9

| Property | Value |
|----------|-------|
| **Rationale** | WinUI 3 runtime, C# async/await, LINQ, System.Text.Json |
| **Used By** | MOD-UI only |
| **Runtime** | Self-contained deployment with trimming |

---

## AI / ML Stack

### ONNX Runtime 1.19+

| Property | Value |
|----------|-------|
| **Rationale** | Cross-platform inference engine, DirectML GPU acceleration, quantization support |
| **Used By** | MOD-WAKE, MOD-ASR, MOD-NLU, MOD-LLM |
| **Execution Providers** | DirectML (primary), CPU (fallback) |
| **Model Formats** | ONNX, ONNX with QDQ quantization, ONNX with INT8 |
| **Notes** | DirectML EP requires DirectX 12 compatible GPU; falls back to CPU EP automatically |

### Windows ML (Microsoft.Windows.AI.MachineLearning)

| Property | Value |
|----------|-------|
| **Rationale** | Native Windows inference, hardware-accelerated via DirectML, no external dependency |
| **Used By** | MOD-WAKE (lightweight models), MOD-NLU (intent classification) |
| **Notes** | Ships in-box with Windows 11 24H2+; redistributable for Windows 10 |

### Candle (Rust ML Framework)

| Property | Value |
|----------|-------|
| **Crate** | `candle-core`, `candle-nn`, `candle-transformers` |
| **Rationale** | Pure Rust inference, no Python dependency, GGUF support, Metal/DirectML backends |
| **Used By** | MOD-LLM (alternative inference path), MOD-KB (embeddings) |
| **Notes** | Used when ONNX Runtime does not support a model architecture |

### Tokenizers (Hugging Face)

| Property | Value |
|----------|-------|
| **Crate** | `tokenizers` (Rust port) |
| **Rationale** | Fast tokenization, BPE/WordPiece/Unigram support, pre-tokenized dataset caching |
| **Used By** | MOD-LLM, MOD-NLU |

---

## Audio Processing Stack

### CPAL (Cross-Platform Audio Library)

| Property | Value |
|----------|-------|
| **Crate** | `cpal` 0.15+ |
| **Rationale** | Low-level audio I/O, WASAPI backend on Windows, event-driven capture, minimal latency |
| **Used By** | MOD-AUDIO (capture and playback) |
| **Backend** | WASAPI (primary), ASIO (optional low-latency) |
| **Notes** | ASIO support requires `asio` feature flag and ASIO SDK |

### Symphonia

| Property | Value |
|----------|-------|
| **Crate** | `symphonia` 0.6+ |
| **Rationale** | Pure Rust media container and audio decoding (MP3, FLAC, WAV, OGG) |
| **Used By** | MOD-AUDIO (audio file playback), MOD-TTS (reference audio) |

### Rubato

| Property | Value |
|----------|-------|
| **Crate** | `rubato` 0.16+ |
| **Rationale** | High-quality asynchronous resampling (sinc interpolation) |
| **Used By** | MOD-AUDIO (sample rate conversion) |

### dasp (Digital Audio Signal Processing)

| Property | Value |
|----------|-------|
| **Crate** | `dasp` 0.12+ |
| **Rationale** | DSP primitives: frames, signals, interpolation, envelope detection |
| **Used By** | MOD-AUDIO (preprocessing), MOD-VAD (energy detection) |

---

## Speech Processing

### Whisper.cpp (Rust Bindings)

| Property | Value |
|----------|-------|
| **Crate** | `whisper-rs` 0.13+ |
| **Rationale** | Local ASR, streaming support, quantization, cross-platform |
| **Used By** | MOD-ASR (primary transcription engine) |
| **Model Size** | base (74M) or small (244M) for latency targets |
| **Notes** | Quantized to Q5_0 or Q4_0 for memory efficiency |

### Vosk (Alternative ASR)

| Property | Value |
|----------|-------|
| **Crate** | `vosk` (bindings) |
| **Rationale** | Lightweight ASR, small model footprint, offline operation |
| **Used By** | MOD-ASR (fallback on low-end hardware) |

### Piper TTS (Rust Port)

| Property | Value |
|----------|-------|
| **Crate** | `piper-rs` |
| **Rationale** | Fast, lightweight neural TTS, ONNX-based, multi-speaker |
| **Used By** | MOD-TTS (primary synthesis engine) |
| **Model Format** | ONNX with espeak-ng phonemization |

### Kokoro TTS

| Property | Value |
|----------|-------|
| **Crate** | `kokoro` (Rust bindings) |
| **Rationale** | High-quality neural TTS, minimal latency, small models |
| **Used By** | MOD-TTS (premium voice option) |

---

## Infrastructure and Utilities

### Tokio

| Property | Value |
|----------|-------|
| **Crate** | `tokio` 1.40+ (full feature set) |
| **Rationale** | Async runtime, task scheduling, I/O multiplexing, channels |
| **Used By** | All async Rust modules |
| **Runtime** | Multi-threaded scheduler (worker_threads = num_cpus) |

### Serde

| Property | Value |
|----------|-------|
| **Crate** | `serde` 1.0+, `serde_json`, `toml` |
| **Rationale** | Serialization framework, configuration parsing, IPC messages |
| **Used By** | MOD-CONFIG, MOD-EVENT, MOD-TELEM |

### Tracing

| Property | Value |
|----------|-------|
| **Crate** | `tracing` 0.1+, `tracing-subscriber` |
| **Rationale** | Structured logging, spans, OpenTelemetry integration |
| **Used By** | MOD-TELEM, all modules for observability |

### thiserror + anyhow

| Property | Value |
|----------|-------|
| **Crate** | `thiserror` 1.0+, `anyhow` 1.0+ |
| **Rationale** | Error handling: `thiserror` for library errors, `anyhow` for application errors |
| **Used By** | All Rust modules |

### windows-rs

| Property | Value |
|----------|-------|
| **Crate** | `windows` 0.58+ |
| **Rationale** | Rust bindings for Win32 and WinRT APIs, generated from metadata |
| **Used By** | MOD-WINAPI, MOD-UIAUTO, MOD-APPCTL |

### memmap2

| Property | Value |
|----------|-------|
| **Crate** | `memmap2` 0.9+ |
| **Rationale** | Memory-mapped files for large model weights, zero-copy audio buffers |
| **Used By** | MOD-LLM, MOD-AUDIO |

### parking_lot

| Property | Value |
|----------|-------|
| **Crate** | `parking_lot` 0.12+ |
| **Rationale** | Fast synchronization primitives (Mutex, RwLock) |
| **Used By** | All modules requiring concurrent access |

### dashmap

| Property | Value |
|----------|-------|
| **Crate** | `dashmap` 6.0+ |
| **Rationale** | Concurrent hash map, lock-free reads |
| **Used By** | MOD-EVENT (handler registry), MOD-CONFIG (settings cache) |

### crossbeam

| Property | Value |
|----------|-------|
| **Crate** | `crossbeam` 0.8+ |
| **Rationale** | Lock-free data structures, channels, epoch-based memory reclamation |
| **Used By** | MOD-EVENT (high-throughput channels), MOD-AUDIO (ring buffers) |

---

## Database and Storage

### SQLite

| Property | Value |
|----------|-------|
| **Crate** | `rusqlite` 0.32+ |
| **Rationale** | Embedded database, zero configuration, ACID transactions |
| **Used By** | MOD-CONFIG (settings), MOD-KB (document metadata), MOD-TELEM (local logs) |

### HNSW (Vector Search)

| Property | Value |
|----------|-------|
| **Crate** | `hnsw` or `instant-distance` |
| **Rationale** | Approximate nearest neighbor search for embeddings |
| **Used By** | MOD-KB (vector store) |

---

## Testing Stack

### cargo-nextest

| Property | Value |
|----------|-------|
| **Rationale** | Faster test execution, better output, JUnit XML export |
| **Used By** | CI/CD pipelines, local development |

### proptest

| Property | Value |
|----------|-------|
| **Crate** | `proptest` 1.5+ |
| **Rationale** | Property-based testing, fuzz-like input generation |
| **Used By** | MOD-AUDIO, MOD-ASR, MOD-LLM |

### mockall

| Property | Value |
|----------|-------|
| **Crate** | `mockall` 0.13+ |
| **Rationale** | Mock generation for traits, dependency injection in tests |
| **Used By** | All modules with external dependencies |

### criterion

| Property | Value |
|----------|-------|
| **Crate** | `criterion` 0.5+ |
| **Rationale** | Statistical benchmarking, regression detection |
| **Used By** | Performance-critical modules (MOD-ASR, MOD-LLM, MOD-TTS) |

---

## CI/CD and DevOps

### GitHub Actions

| Property | Value |
|----------|-------|
| **Rationale** | Native GitHub integration, Windows runners, matrix builds |
| **Runners** | `windows-latest` (x64) |
| **Features** | Caching, artifacts, code signing integration |

### cargo-deny

| Property | Value |
|----------|-------|
| **Rationale** | License compliance, banned crate detection, security advisory checking |
| **Used By** | CI gates, pre-commit hooks |

### cargo-audit

| Property | Value |
|----------|-------|
| **Rationale** | RustSec advisory database scanning |
| **Used By** | CI security gates |

### Code Signing

| Property | Value |
|----------|-------|
| **Tool** | Azure Code Signing or DigiCert |
| **Rationale** | Windows SmartScreen compatibility, MSIX distribution |
| **Used By** | Release pipeline |

---

## Complete Dependency Registry

### Workspace Cargo.toml

```toml
[workspace]
members = [
    "src/audio",
    "src/wake_word",
    "src/asr",
    "src/nlu",
    "src/llm",
    "src/tts",
    "src/action",
    "src/winapi_bridge",
    "src/ui_automation",
    "src/app_control",
    "src/file_system",
    "src/process",
    "src/config",
    "src/event_bus",
    "src/telemetry",
    "src/update",
    "src/security",
    "src/knowledge_base",
    "src/plugin",
]
resolver = "2"

[workspace.dependencies]
# Async runtime
tokio = { version = "1.40", features = ["full"] }
tokio-util = "0.7"

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
toml = "0.8"

# Error handling
thiserror = "1.0"
anyhow = "1.0"

# Logging and tracing
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }

# Windows APIs
windows = { version = "0.58", features = ["Win32_Foundation", "Win32_System_Threading", "Win32_UI_WindowsAndMessaging", "Win32_Media_Audio", "Win32_System_Com", "Win32_UI_Accessibility"] }

# Audio
cpal = "0.15"
symphonia = { version = "0.6", features = ["mp3", "flac", "wav", "ogg"] }
rubato = "0.16"
dasp = "0.12"

# AI/ML
ort = { version = "2.0", features = ["directml"] }
candle-core = "0.8"
candle-nn = "0.8"
candle-transformers = "0.8"
tokenizers = "0.21"

# Concurrency
dashmap = "6.0"
parking_lot = "0.12"
crossbeam = "0.8"

# Storage
rusqlite = { version = "0.32", features = ["bundled"] }
memmap2 = "0.9"

# Testing
mockall = "0.13"
proptest = "1.5"
criterion = "0.5"

# Utilities
uuid = { version = "1.11", features = ["v4", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
regex = "1.11"
```

---

## Technology Replacement Policy

| Technology | Replacement Candidate | Trigger Condition | Timeline |
|------------|----------------------|-------------------|----------|
| ONNX Runtime | Candle (full) | Candle achieves parity on all model types | 2027 Q2 |
| Whisper.cpp | Windows Speech Recognition API | Windows Speech API reaches WER parity | 2026 Q4 |
| CPAL | Windows Audio Session API (direct) | If CPAL latency exceeds 10ms | On demand |
| SQLite | sled (Rust-native) | If sled reaches production stability | 2027 Q1 |
| WinUI 3 | WinUI 4 | When WinUI 4 GA with migration path | TBD |

---

## References

- [Rust Book](https://doc.rust-lang.org/book/)
- [Tokio Documentation](https://tokio.rs/)
- [ONNX Runtime Docs](https://onnxruntime.ai/docs/)
- [Candle GitHub](https://github.com/huggingface/candle)
- [CPAL Documentation](https://docs.rs/cpal/)
- [WinUI 3 Documentation](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/)
- [windows-rs Documentation](https://microsoft.github.io/windows-rs/)

---

## Cross References

- See [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md) for module mapping.
- See [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) for language-specific conventions.
- See [36_DEPENDENCY_GUIDE.md](36_DEPENDENCY_GUIDE.md) for dependency management rules.
- See [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md) for Windows API usage patterns.
- See [38_LIBRARY_GUIDE.md](38_LIBRARY_GUIDE.md) for third-party library integration patterns.

---

## Best Practices

1. **Pin all dependency versions** in workspace Cargo.toml; no wildcard versions.
2. **Audit new dependencies** with `cargo-deny` before adding to workspace.
3. **Prefer pure Rust** over C/C++ bindings when performance is equivalent.
4. **Use feature flags** to minimize binary size (e.g., `symphonia` features).
5. **Keep DirectML as primary** ONNX EP; CPU EP is fallback only.
6. **Test on minimum spec hardware** (8GB RAM, integrated GPU) regularly.

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| Adding unvetted dependencies | Supply chain risk, license violations | Mandatory cargo-deny check |
| Using blocking I/O in async context | Thread pool exhaustion, latency spikes | Always use tokio async I/O |
| Loading models on main thread | UI freezes, poor responsiveness | Offload to tokio blocking pool |
| Ignoring feature flags | Bloated binaries, slow compile times | Audit features per crate |
| Mixing error types inconsistently | Confusing error propagation | thiserror for libs, anyhow for apps |

---

## Review Checklist

- [ ] All technologies have documented selection rationale.
- [ ] Version numbers are pinned and current.
- [ ] License compatibility is verified for all dependencies.
- [ ] Security advisories are checked (cargo-audit clean).
- [ ] Performance benchmarks confirm latency targets.
- [ ] Replacement policy is reviewed quarterly.
- [ ] CI/CD tools are integrated and functional.

---

*End of 03_TECH_STACK.md*
