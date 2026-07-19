# 001 - System Overview Specification

## Purpose

Define the architectural vision, design principles, and high-level system structure of VOXY - a Windows-native AI agent runtime. This document serves as the north star for all subsystem specifications.

## Scope

- High-level architecture and component topology
- Design principles and non-functional requirements
- Technology stack and platform choices
- Integration patterns and data flows
- Security and privacy architecture
- Deployment and operational model

## Responsibilities

- Establish architectural consistency across all subsystems
- Define the boundary between VOXY and the Windows platform
- Specify integration contracts with external AI services
- Document the threat model and security posture
- Define performance and reliability targets

## Non-Goals

- Detailed API specifications (covered in subsystem specs)
- Implementation code or algorithms
- UI/UX design specifications
- Marketing or user-facing documentation

## Architecture Position

VOXY sits between the Windows platform APIs and external AI services, orchestrating audio capture, cognitive processing, UI automation, and memory persistence.

## Design Principles

### 1. Windows-Native First
VOXY is built for Windows. Every subsystem leverages native Windows APIs before considering cross-platform abstractions. This ensures optimal performance, deep OS integration, and access to platform-specific AI accelerators (NPU, DirectML).

### 2. Local-First AI
Where possible, inference happens on-device using Windows AI APIs (Phi Silica), ONNX Runtime with DirectML, or local model servers. Cloud APIs are used as fallbacks or for capabilities exceeding local hardware.

### 3. Event-Driven Architecture
All subsystems communicate through the Event Bus using strongly-typed messages. This decouples producers from consumers and enables reactive, asynchronous processing.

### 4. Memory as a First-Class Citizen
VOXY maintains persistent, structured memory across sessions. Memory is not an afterthought - it is a core architectural pillar enabling long-term context and personalization.

### 5. Security by Design
Every subsystem implements defense in depth. Process isolation, capability-based permissions, encrypted storage, and audit logging are mandatory, not optional.

### 6. Extensibility Through Plugins
Third-party capabilities are integrated via a sandboxed plugin system with strict permission models. Plugins cannot compromise core runtime integrity.

## Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Language | Rust (primary), C++ (Win32 interop), C# (WinUI) | Memory safety, performance, Windows ecosystem |
| Async Runtime | Tokio | Industry-standard async runtime for Rust |
| Audio | WASAPI (Windows Audio Session API) | Low-latency, loopback capture, event-driven |
| AI Inference | ONNX Runtime + DirectML | Cross-hardware acceleration (CPU/GPU/NPU) |
| Local LLM | Windows AI APIs (Phi Silica) | Native NPU-optimized SLM integration |
| UI Automation | Microsoft UI Automation (UIA) | Standard Windows accessibility framework |
| Screen Capture | Windows Graphics Capture API | Hardware-accelerated, privacy-respecting |
| Database | SQLite + sqlite-vec | Embedded, ACID, vector search |
| Embeddings | ONNX Runtime (local) or OpenAI API (cloud) | Flexible deployment |
| IPC | Named Pipes + Memory-Mapped Files | Fast, secure, Windows-native |
| Packaging | MSIX | Modern Windows packaging with identity |

## Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Wake Word Latency | < 200ms | End-to-end, microphone to detection |
| STT Latency | < 500ms | First transcript for 5-second utterance |
| TTS Latency | < 300ms | First audio chunk for 50-token response |
| UI Action Latency | < 2s | Command to action completion |
| Memory Query | < 100ms | Vector search + ranking |
| Cold Start | < 3s | Process launch to ready state |
| Memory Footprint | < 2GB RAM | Steady-state working set |
| CPU Usage | < 15% | Idle, background monitoring |

## Deployment Model

VOXY deploys as a Windows desktop application with optional Windows Service component for background operation. The MSIX package provides:

- Application identity and capability declarations
- Automatic updates via Microsoft Store or private app installer
- Clean uninstall with no registry residue
- Optional background task registration

## Cross References

- [010_KERNEL_SPEC.md](010_KERNEL_SPEC.md)
- [090_SECURITY_MODEL_SPEC.md](090_SECURITY_MODEL_SPEC.md)
- [120_PERFORMANCE_TARGETS.md](120_PERFORMANCE_TARGETS.md)

## References

- Windows App SDK 2.2 Documentation
- Windows AI APIs - Phi Silica
- Microsoft UI Automation
- WASAPI Documentation
- ONNX Runtime
- Tokio Async Runtime
- sqlite-vec
