# VOXY Glossary — A

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## ASR (Automatic Speech Recognition)

| Field | Value |
|-------|-------|
| **Category** | Technology / AI |
| **Definition** | The computational process of converting spoken language audio into written text. In VOXY, ASR is implemented using local inference engines (Whisper.cpp, Vosk) with ONNX Runtime acceleration. |
| **Purpose** | Transforms user voice input into machine-readable text for downstream natural language understanding. |
| **Used By** | MOD-ASR, MOD-NLU, MOD-ACTION |
| **Related Terms** | STT, Wake Word, NLU, Transcription |
| **Related Modules** | MOD-ASR, MOD-AUDIO |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | ASR operates in streaming mode for real-time transcription. The engine processes audio frames as they arrive, emitting partial results for low-latency feedback and final results for command execution. Two backends are supported: Whisper.cpp (primary, higher accuracy) and Vosk (fallback, lower resource usage). |

---

## ASIO (Audio Stream Input/Output)

| Field | Value |
|-------|-------|
| **Category** | Technology / Audio |
| **Definition** | A computer sound card driver protocol for digital audio specified by Steinberg, providing a low-latency and high fidelity interface between a software application and a computer's sound card. |
| **Purpose** | Enables sub-10ms audio latency for professional audio applications and VOXY's voice-first interaction model. |
| **Used By** | MOD-AUDIO |
| **Related Terms** | WASAPI, CPAL, Audio Latency |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | ASIO support in VOXY is optional and enabled via CPAL's asio feature flag. It requires the ASIO SDK to be installed and compatible hardware. When ASIO is unavailable, VOXY falls back to WASAPI event-driven mode. |

---

## Accessibility (A11Y)

| Field | Value |
|-------|-------|
| **Category** | Standard / Compliance |
| **Definition** | The design and development of products, devices, services, or environments so that people with disabilities can use them. In VOXY, this encompasses screen reader support, keyboard navigation, high contrast modes, and voice-first alternatives to visual UI. |
| **Purpose** | Ensures VOXY is usable by people with visual, motor, cognitive, and auditory disabilities, complying with WCAG 2.1 AA and Section 508. |
| **Used By** | MOD-UI, MOD-UIAUTO, MOD-TELEM |
| **Related Terms** | UI Automation, Screen Reader, WCAG, Section 508 |
| **Related Modules** | MOD-UI, MOD-UIAUTO |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (A11Y-* rules) |
| **Architecture Notes** | VOXY leverages Windows UI Automation (UIA) to expose all interactive elements to assistive technologies. The MOD-UIAUTO module programmatically interacts with accessibility trees, enabling voice commands to control any accessible application on the system. |

---

## Action Engine

| Field | Value |
|-------|-------|
| **Category** | Component / Architecture |
| **Definition** | The central routing and execution component that maps resolved intents (from NLU) to concrete system operations, manages execution lifecycle, handles errors, and coordinates multi-step workflows. |
| **Purpose** | Bridges the gap between user intent ("open Visual Studio") and system execution (launching devenv.exe, focusing the window). |
| **Used By** | MOD-ACTION, MOD-APPCTL, MOD-FILE, MOD-PROC, MOD-WINAPI |
| **Related Terms** | Intent Router, Command Execution, Workflow Orchestrator |
| **Related Modules** | MOD-ACTION, MOD-NLU, MOD-LLM |
| **Related Specifications** | [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | The Action Engine maintains a registry of action handlers, each implementing the ActionHandler trait. Actions are executed asynchronously with configurable timeouts, retry policies, and circuit breakers. The engine supports both atomic actions (single operation) and composite actions (sequences with conditionals). |

---

## ADR (Architecture Decision Record)

| Field | Value |
|-------|-------|
| **Category** | Process / Documentation |
| **Definition** | A document that captures an important architectural decision made along with its context and consequences. VOXY uses ADRs for all significant design choices, technology selections, and rule exceptions. |
| **Purpose** | Provides traceability for architectural decisions, enables new team members to understand design rationale, and prevents repeated deliberation on resolved questions. |
| **Used By** | Architecture Review Board, Engineering Team |
| **Related Terms** | RFC, Decision Log, Technical Debt |
| **Related Modules** | All modules (cross-cutting) |
| **Related Specifications** | [29_ADR_TEMPLATE.md](29_ADR_TEMPLATE.md), [28_DECISION_LOG_TEMPLATE.md](28_DECISION_LOG_TEMPLATE.md) |
| **Architecture Notes** | ADRs are stored in docs/architecture/adr/ with sequential numbering (e.g., ADR-0012-onnx-runtime-selection.md). Each ADR has a status: Proposed, Accepted, Deprecated, or Superseded. Superseded ADRs reference their replacements. |

---

## Agentic AI

| Field | Value |
|-------|-------|
| **Category** | Concept / AI |
| **Definition** | Artificial intelligence systems that can autonomously perceive their environment, make decisions, and take actions to achieve goals without continuous human intervention. In VOXY, this manifests as the ability to break down complex user requests into multi-step plans and execute them. |
| **Purpose** | Enables VOXY to handle complex, multi-step user requests ("Set up my development environment for Rust") rather than simple one-shot commands. |
| **Used By** | MOD-LLM, MOD-ACTION, MOD-PLUG |
| **Related Terms** | LLM, Tool Calling, Planning, Autonomous Agent |
| **Related Modules** | MOD-LLM, MOD-ACTION |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [07_AI_AGENT_PLAYBOOK.md](07_AI_AGENT_PLAYBOOK.md) |
| **Architecture Notes** | VOXY's agentic capabilities are built on local LLM inference with tool calling. The LLM generates a plan (sequence of tool invocations), which the Action Engine executes with feedback loops. The system maintains execution state and can handle partial failures through replanning. |

---

*End of 10_GLOSSARY_A.md*
