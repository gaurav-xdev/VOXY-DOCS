SECTION 1 & 7: Windows Native & Desktop Vision Architecture

1. Problem Statement

Windows is a heavily fragmented ecosystem (Win32, COM, WPF, UWP, WinUI3). Abstraction layers (Node.js/Electron) introduce unacceptable memory overhead and limit access to low-level APIs (Hooks, ETW). To achieve OpenClaw-grade automation with Siri-level latency, VOXY must interface directly with the Windows kernel and user subsystems.

2. Windows API Analysis & Trade-offs

A. UI Interaction & Automation

UIAutomation (UIA) vs. Win32 vs. Accessibility:

UIA (COM-based): The modern standard. Provides the accessibility tree. Highly structured, but querying the tree can be synchronous and slow.

Win32 Messages (SendMessage): Fast, but legacy. Only works on classic apps.

Raw Input / SendInput: Required for mouse/keyboard emulation, but blind.

Recommendation: Use a background thread to maintain a cached UIAutomation tree. Use SendInput for execution only when UIA InvokePattern is unavailable.

B. Screen Capture & Vision (Section 7)

BitBlt vs. DXGI Desktop Duplication vs. Windows.Graphics.Capture:

BitBlt: Slow, CPU intensive, fails on hardware-accelerated windows (Chrome).

DXGI: Fast, but complex GPU state management.

Windows.Graphics.Capture (WinRT): Modern, zero-copy GPU capture.

Recommendation: Use Windows.Graphics.Capture for screen grabbing.

Vision Pipeline: Screen Capture -> Windows.Media.Ocr (Local, zero-latency) -> Layout Analysis (combining OCR bounds with UIA tree). Do not rely solely on Vision-Language Models (VLMs) due to latency.

C. Audio Subsystem (WASAPI)

Core Audio / WASAPI: For sub-500ms voice interactions, VOXY must bypass the Windows audio mixer (DirectSound).

Recommendation: Use WASAPI in Exclusive Mode for microphone capture to bypass the Windows audio engine completely, ensuring sub-10ms capture latency. Combine with Windows built-in AEC (Acoustic Echo Cancellation) via IMediaObject if WebRTC AEC proves too heavy.

3. Security & Execution Boundary (Section 12)

Sandboxing: Running AI-generated commands as the primary user is a catastrophic security risk (OpenClaw's main flaw).

Recommendation: VOXY Core runs as a standard User process. Automation scripts (Python/Powershell) requested by the LLM must run inside a Windows Sandbox (WSB) or a tightly constrained AppContainer, communicating via Named Pipes. Use Windows DPAPI (Data Protection API) for storing local API keys/secrets.