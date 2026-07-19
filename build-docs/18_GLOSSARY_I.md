# VOXY Glossary — I

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## Intent Classification

| Field | Value |
|-------|-------|
| **Category** | AI / NLU |
| **Definition** | The process of determining the user's goal or intention from natural language input. VOXY classifies intents into categories like open_application, search_files, send_message. |
| **Purpose** | Maps user utterances to executable action categories, enabling the Action Engine to route commands to appropriate handlers. |
| **Used By** | MOD-NLU, MOD-ACTION |
| **Related Terms** | Entity Extraction, Slot Filling, Command Classification |
| **Related Modules** | MOD-NLU |
| **Related Specifications** | [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY uses a lightweight ONNX intent classification model (DistilBERT-based, quantized to INT8) running locally. The model supports 50+ built-in intents and allows custom intent definitions via configuration. Classification latency target: <50ms. |

---

## IPC (Inter-Process Communication)

| Field | Value |
|-------|-------|
| **Category** | Architecture / OS |
| **Definition** | Mechanisms that allow separate processes to exchange data and synchronize operations. VOXY uses IPC between the Rust backend and the WinUI 3 frontend. |
| **Purpose** | Enables separation of the UI process from the engine process for stability (UI crashes don't affect voice processing) and security (sandboxed plugins). |
| **Used By** | MOD-UI, MOD-PLUG, MOD-EVENT |
| **Related Terms** | Named Pipe, gRPC, Message Passing, Process Isolation |
| **Related Modules** | MOD-UI, MOD-PLUG |
| **Related Specifications** | [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md) |
| **Architecture Notes** | VOXY uses gRPC over Windows named pipes for IPC. The protocol is defined in Protobuf and supports streaming for real-time audio data and event notifications. IPC messages are serialized with serde_json for debugging. |

---

## Inference

| Field | Value |
|-------|-------|
| **Category** | AI / ML |
| **Definition** | The process of running a trained machine learning model on new input data to produce predictions or outputs. In VOXY, inference happens locally for ASR, NLU, LLM, and wake word detection. |
| **Purpose** | Converts raw input (audio, text) into structured output (transcripts, intents, responses) using trained AI models. |
| **Used By** | MOD-ASR, MOD-NLU, MOD-LLM, MOD-WAKE, MOD-TTS |
| **Related Terms** | Forward Pass, Prediction, Model Execution, ONNX Runtime |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY inference uses ONNX Runtime with DirectML EP as primary and CPU EP as fallback. Models are quantized (INT8, Q4, Q5) for memory efficiency. Inference runs on background threads to avoid blocking the UI. |

---

## INT8 Quantization

| Field | Value |
|-------|-------|
| **Category** | AI / Optimization |
| **Definition** | A model optimization technique that converts 32-bit floating-point weights to 8-bit integers, reducing model size by 4x and improving inference speed on hardware with INT8 support. |
| **Purpose** | Enables larger models to run on devices with limited memory and improves inference throughput via hardware-accelerated integer operations. |
| **Used By** | MOD-ASR, MOD-NLU, MOD-LLM, MOD-WAKE |
| **Related Terms** | Quantization, QDQ, Calibration, Model Compression |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY uses ONNX Runtime's quantization tools with QDQ (Quantize-Dequantize) format. Calibration is performed on representative data. INT8 models are validated against FP32 baselines to ensure accuracy degradation is <1% WER for ASR and <2% for intent classification. |

---

## Interop

| Field | Value |
|-------|-------|
| **Category** | Programming |
| **Definition** | Short for interoperability, the ability of different software systems, programming languages, or components to work together. VOXY's interop layer connects Rust to Windows APIs and C++ libraries. |
| **Purpose** | Enables VOXY to leverage existing Windows platform capabilities and optimized C++ ML libraries while maintaining a Rust-centric architecture. |
| **Used By** | MOD-WINAPI, MOD-ASR, MOD-LLM |
| **Related Terms** | FFI, Binding, COM, P/Invoke |
| **Related Modules** | MOD-WINAPI |
| **Related Specifications** | [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md) |
| **Architecture Notes** | VOXY's interop strategy: windows-rs for Win32/WinRT, ort for ONNX Runtime, whisper-rs for Whisper.cpp, and custom C ABI for UI-to-backend communication. All interop is wrapped in safe Rust with comprehensive error handling. |

---

*End of 18_GLOSSARY_I.md*