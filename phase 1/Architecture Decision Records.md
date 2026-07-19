Architecture Decision Records (ADRs)

ADR 001: Core Language & Runtime

Status: Accepted

Context: VOXY needs access to low-level Windows APIs, strict memory safety, and high performance. Electron/Node.js is too heavy; Python is too slow for real-time audio.

Decision: The core daemon and audio pipeline will be written in Rust using the windows-rs crate. The AI inference layers will wrap C++ libraries (Llama.cpp, Whisper.cpp). The UI (System Tray/Overlay) will be built in C# / WinUI 3 communicating via gRPC.

Consequences: Steeper learning curve for developers, but guarantees sub-500ms latency and <200MB idle RAM footprint.

ADR 002: Local-First Fallback Strategy

Status: Accepted

Context: Users have varying hardware. An RTX 4090 can run Llama-3 locally, while an Intel Iris laptop cannot without massive latency.

Decision: VOXY will implement a dynamic execution router.

Tier 1 (NPU/Dedicated GPU): 100% local processing (Phi-3 / Llama-3 8B).

Tier 2 (Low-end Hardware): Local Wake Word & VAD -> Cloud Groq (LLM) / Deepgram (STT).

Consequences: Requires cloud infrastructure fallback, but ensures VOXY is usable on standard enterprise hardware.

ADR 003: UI Understanding Mechanism

Status: Accepted

Context: Relying strictly on Vision Language Models (VLMs) for screen interaction is too slow (>2s latency) and costly.

Decision: Use a Hybrid Grounding approach. Primary extraction uses Windows UIAutomation tree. Fallback uses local Tesseract/Windows.Media.Ocr. Screenshots are only fed to a VLM if the user explicitly asks a visual question ("What is in this picture?").

Consequences: Massive speedup in automation; deterministic clicking.