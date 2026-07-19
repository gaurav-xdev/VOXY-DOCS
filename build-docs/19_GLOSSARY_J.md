# VOXY Glossary — J

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## JSON (JavaScript Object Notation)

| Field | Value |
|-------|-------|
| **Category** | Technology / Data Format |
| **Definition** | A lightweight, text-based data interchange format that is easy for humans to read and write and easy for machines to parse and generate. VOXY uses JSON for configuration, IPC messages, and telemetry. |
| **Purpose** | Provides a universal, language-agnostic format for data serialization in configuration files, API responses, and log structured data. |
| **Used By** | MOD-CONFIG, MOD-TELEM, MOD-EVENT, MOD-UI |
| **Related Terms** | serde_json, TOML, Serialization, Schema |
| **Related Modules** | MOD-CONFIG, MOD-TELEM |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY uses serde_json for JSON handling in Rust and System.Text.Json in C#. JSON schemas are defined for all configuration files and IPC message types. Pretty-printed JSON is used for human-readable config files; compact JSON for IPC and telemetry. |

---

## Jitter

| Field | Value |
|-------|-------|
| **Category** | Audio / Network |
| **Definition** | Variation in the timing of audio packets or data delivery. In audio systems, jitter causes distortion; in networks, it affects real-time communication quality. |
| **Purpose** | Measured and minimized to ensure consistent audio quality and low-latency voice processing. |
| **Used By** | MOD-AUDIO, MOD-TELEM |
| **Related Terms** | Latency, Buffer, Dropout, Real-Time |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY measures audio capture jitter via WASAPI timestamps. Target jitter: <1ms. High jitter triggers buffer size adjustment or backend switching (WASAPI -> ASIO). Jitter metrics are reported in telemetry for quality monitoring. |

---

## JIT Compilation (Just-In-Time Compilation)

| Field | Value |
|-------|-------|
| **Category** | Programming / Runtime |
| **Definition** | Compilation of code during program execution rather than beforehand. ONNX Runtime uses JIT compilation for graph optimization and kernel fusion. |
| **Purpose** | Optimizes model execution at runtime based on the specific hardware and input shapes, improving inference performance. |
| **Used By** | MOD-ASR, MOD-LLM, MOD-NLU |
| **Related Terms** | ONNX Runtime, Graph Optimization, Kernel Fusion, AOT |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | ONNX Runtime's JIT compilation fuses operators and generates optimized kernels for the target hardware (CPU SIMD, GPU shaders). First inference may be slower due to JIT warmup; subsequent inferences use cached compiled kernels. |

---

*End of 19_GLOSSARY_J.md*