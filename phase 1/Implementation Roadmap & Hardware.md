Implementation Roadmap & Hardware Specs

1. Hardware Compatibility Matrix (Section 10)

VOXY is designed to scale across hardware tiers:

Hardware Tier

STT (Whisper)

LLM

TTS

Note

Minimum (Intel Core i5, 8GB RAM)

Whisper.cpp (CPU - Base)

Cloud Fallback / Phi-3 4B (CPU)

CPU ONNX

Will heavily utilize RAM. Battery impact high.

Recommended (RTX 3060+, 16GB RAM)

DirectML / CUDA

Llama-3 8B (4-bit GGUF)

DirectML

Sub-500ms latency easily achievable.

Enterprise / Next-Gen (Copilot+ PCs)

NPU (DirectML)

NPU (ONNX)

NPU

Zero battery impact. Ideal state.

2. Implementation Roadmap

Phase 1: Foundation (Month 1)

Setup Rust/WinUI3 workspace.

Implement WASAPI Exclusive Mode audio loop.

Integrate openWakeWord and Silero VAD.

Milestone: VOXY can reliably wake up and detect speech end without UI.

Phase 2: Inference & Voice Pipeline (Month 2-3)

Integrate Whisper.cpp (streaming).

Integrate Llama.cpp with DirectML support for Windows.

Integrate StyleTTS2.

Milestone: VOXY can hold a continuous, sub-500ms latency voice conversation.

Phase 3: Desktop Automation (Month 4-5)

Build the UIA / OCR parsing engine.

Implement the MCP (Model Context Protocol) tool router.

Milestone: VOXY can open apps, read the screen, and click deterministic elements.

Phase 4: Memory & Hermes Loop (Month 6)

Integrate SQLite-vec.

Build the background reflection agent.

Milestone: VOXY remembers user preferences across reboots and optimizes its own workflows.

3. Top Risks & Open Questions

Risk: Windows 11 updates frequently break undocumented UIA behaviors.

Mitigation: Extensive regression testing against Insider Preview builds.

Risk: Audio exclusive mode blocks other apps (e.g., OBS) from capturing the mic.

Mitigation: Implement a software audio loopback/splitter if Exclusive mode causes user friction.

Open Question: Can we achieve reliable multi-monitor high-DPI scaling OCR in under 100ms? Requires benchmarking Windows.Media.Ocr vs custom ONNX models.