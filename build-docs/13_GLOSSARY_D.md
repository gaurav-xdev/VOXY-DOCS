# VOXY Glossary — D

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## DirectML

| Field | Value |
|-------|-------|
| **Category** | Technology / AI / GPU |
| **Definition** | Microsoft's hardware-accelerated API for machine learning on DirectX 12 capable GPUs. DirectML serves as an ONNX Runtime Execution Provider, enabling GPU-accelerated inference on Windows. |
| **Purpose** | Provides GPU acceleration for AI model inference (ASR, NLU, LLM) without requiring vendor-specific SDKs (CUDA, ROCm). |
| **Used By** | MOD-ASR, MOD-NLU, MOD-LLM, MOD-WAKE |
| **Related Terms** | ONNX Runtime, Execution Provider, GPU Acceleration, DX12 |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | DirectML is the primary ONNX EP in VOXY. It supports all major GPU vendors (Intel, AMD, NVIDIA) via DirectX 12 drivers. If DirectML is unavailable, VOXY falls back to the CPU EP automatically. DirectML is particularly efficient for quantized INT8 models. |

---

## DPAPI (Data Protection API)

| Field | Value |
|-------|-------|
| **Category** | Technology / Security |
| **Definition** | A Windows cryptographic API for protecting data using the user's login credentials. DPAPI encrypts data with a key derived from the user's password, making it transparent to applications. |
| **Purpose** | Securely stores sensitive user data (credentials, API keys, encryption keys) without requiring the application to manage encryption keys. |
| **Used By** | MOD-SEC, MOD-CONFIG |
| **Related Terms** | Credential Manager, Encryption, Key Derivation |
| **Related Modules** | MOD-SEC |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (SEC-004) |
| **Architecture Notes** | VOXY uses DPAPI via the windows-rs crate for credential storage. Data encrypted with DPAPI can only be decrypted by the same user on the same machine. For roaming scenarios, VOXY implements additional key wrapping. |

---

## DSP (Digital Signal Processing)

| Field | Value |
|-------|-------|
| **Category** | Technology / Audio |
| **Definition** | The mathematical processing of digital signals (such as audio) to modify, analyze, or extract information. VOXY uses DSP for audio preprocessing, voice activity detection, and feature extraction. |
| **Purpose** | Prepares raw audio data for AI model consumption by applying filtering, normalization, and transformation operations. |
| **Used By** | MOD-AUDIO, MOD-ASR, MOD-WAKE |
| **Related Terms** | FFT, Filter, Resampling, Spectrogram |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY's DSP pipeline includes: pre-emphasis filtering, windowing (Hamming/Hann), FFT for spectrogram generation, mel-scale filtering, and normalization. The dasp crate provides core DSP primitives. |

---

## Decibel (dB)

| Field | Value |
|-------|-------|
| **Category** | Audio / Measurement |
| **Definition** | A logarithmic unit used to measure audio signal level relative to a reference. In VOXY, decibels are used for voice activity detection thresholding and audio level metering. |
| **Purpose** | Provides a perceptually relevant scale for audio amplitude measurement, matching human hearing's logarithmic response. |
| **Used By** | MOD-AUDIO, MOD-WAKE |
| **Related Terms** | dBFS, RMS, Peak Level, Signal-to-Noise Ratio |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY calculates RMS energy in dBFS (decibels relative to full scale) for VAD decisions. Typical speech ranges from -40 dBFS to -10 dBFS. The VAD threshold is configurable, defaulting to -35 dBFS. |

---

## Device Enumeration

| Field | Value |
|-------|-------|
| **Category** | Audio / Windows |
| **Definition** | The process of querying the system for available audio input and output devices. VOXY enumerates devices via WASAPI to present user-selectable audio endpoints. |
| **Purpose** | Enables users to select specific microphones and speakers, and enables automatic fallback when the default device changes. |
| **Used By** | MOD-AUDIO |
| **Related Terms** | Audio Endpoint, MMDevice, Default Device, Hot-Plug |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md) |
| **Architecture Notes** | VOXY monitors device changes via Windows notifications (WM_DEVICECHANGE) and automatically updates the active audio stream when the default device changes. Device enumeration includes friendly names, formats, and channel counts. |

---

*End of 13_GLOSSARY_D.md*
