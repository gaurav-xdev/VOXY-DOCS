VOXY Master Blueprint: Security, Observability, DX & Implementation

Covers: Security, Plugin SDK, Observability, Developer Experience, Implementation Strategy

1. Purpose

To ensure VOXY is secure by default, easily extensible, maintainable by a senior engineering team, and possesses a clear path to production.

2. Security Architecture & Threat Model

Threat: Malicious LLM outputs (Prompt Injection) executing destructive system commands.

Mitigation (Sandboxing):

VOXY Core has NO direct execution permissions.

Tools are executed via Model Context Protocol (MCP).

High-risk tools (e.g., Python execution, filesystem formatting) are run inside Windows Sandbox (WSB)—a lightweight, disposable VM.

Credential Vault: All API keys and secrets are stored using Windows DPAPI (CryptProtectData), tied to the user's Windows login hash.

3. Observability & Testing Architecture

Telemetry: Built on OpenTelemetry (OTel). Data flows to a local, rolling ring-buffer log file (max 50MB) for privacy. No cloud telemetry by default.

Testing:

Unit: Rust cargo test for core logic.

Voice/Latency: Inject synthetic WAV files directly into the WASAPI buffer mock to verify <500ms TTFT.

Agent: Evaluation against a golden dataset of user intents (Accuracy must > 95%).

4. Developer Experience (DX)

Monorepo Structure:

/core (Rust Kernel & Event Bus)

/audio (WASAPI, C++ AI bindings)

/automation (UIA, OCR, MCP toolset)

/ui (WinUI 3 C# app)

CI/CD: GitHub Actions generating .msix Windows installer packages, code signing via Azure Key Vault.

5. Implementation Roadmap

gantt
    title VOXY Implementation Critical Path
    dateFormat  YYYY-MM-DD
    section Phase 1: Kernel & Audio
    Rust Core & Event Bus       :2026-08-01, 14d
    WASAPI / VAD Pipeline       :2026-08-15, 14d
    section Phase 2: AI & Voice
    Whisper / Llama Integration :2026-09-01, 21d
    StyleTTS2 Integration       :2026-09-22, 14d
    section Phase 3: Desktop
    UIA & OCR Grounding Layer   :2026-10-06, 21d
    MCP Sandbox & Execution     :2026-10-27, 21d
    section Phase 4: Memory
    SQLite-vec & Reflection     :2026-11-17, 30d


6. Failure Modes & Recovery

Supply Chain Attack: Dependency poisoning in cargo crates.

Recovery: Strict lockfiles, manual review of dependency updates, and requiring dependencies to build entirely offline without reaching out to external build scripts.

7. Trade-offs

Strict Sandboxing (WSB): Adds ~1 second of cold-start latency for complex scripting tools compared to running them natively. We accept this for absolute security.

8. Future Extension Points

An Enterprise Policy module allowing IT administrators to inject Group Policy Objects (GPOs) that restrict which desktop applications VOXY's automation engine is permitted to interact with.