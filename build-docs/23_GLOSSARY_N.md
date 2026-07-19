# VOXY Glossary — N

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## NLU (Natural Language Understanding)

| Field | Value |
|-------|-------|
| **Category** | AI / Technology |
| **Definition** | The branch of artificial intelligence that enables machines to understand and interpret human language meaning. In VOXY, NLU encompasses intent classification, entity extraction, and context management. |
| **Purpose** | Transforms transcribed speech into structured actionable data (intent + entities) that the Action Engine can execute. |
| **Used By** | MOD-NLU, MOD-ACTION, MOD-LLM |
| **Related Terms** | Intent, Entity, Slot Filling, Context, ASR |
| **Related Modules** | MOD-NLU |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY's NLU pipeline runs entirely locally using ONNX models. Intent classification uses a quantized DistilBERT model. Entity extraction uses a lightweight token classification model. Context is maintained in a session store for multi-turn dialogue. |

---

## NVDA (NonVisual Desktop Access)

| Field | Value |
|-------|-------|
| **Category** | Technology / Accessibility |
| **Definition** | A free, open-source screen reader for Windows that enables blind and visually impaired users to interact with the operating system and applications. |
| **Purpose** | Primary assistive technology for testing VOXY's accessibility compliance and screen reader compatibility. |
| **Used By** | MOD-UI, MOD-UIAUTO, QA Team |
| **Related Terms** | Screen Reader, JAWS, Narrator, Accessibility |
| **Related Modules** | MOD-UI |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (A11Y-004) |
| **Architecture Notes** | VOXY is tested with NVDA during accessibility audits. All UI elements must expose correct UIA properties (Name, ControlType, HelpText) for NVDA to announce them properly. Voice feedback is synchronized with NVDA announcements. |

---

## Neural Network

| Field | Value |
|-------|-------|
| **Category** | AI / Technology |
| **Definition** | A computational model inspired by biological neural networks, consisting of interconnected nodes (neurons) organized in layers that process information through weighted connections. |
| **Purpose** | Provides the computational foundation for all AI capabilities in VOXY: speech recognition, language understanding, text generation, and voice synthesis. |
| **Used By** | MOD-ASR, MOD-NLU, MOD-LLM, MOD-WAKE, MOD-TTS |
| **Related Terms** | Deep Learning, Transformer, CNN, RNN, Model |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY uses transformer-based neural networks for language tasks (LLM, NLU) and convolutional/recurrent networks for audio tasks (ASR, wake word). All networks run locally via ONNX Runtime or Candle. |

---

## Namespace

| Field | Value |
|-------|-------|
| **Category** | Programming / Organization |
| **Definition** | A declarative region that provides a scope to identifiers (types, functions, variables) to prevent name collisions. In Rust, modules serve as namespaces; in C++, the namespace keyword is used. |
| **Purpose** | Organizes code into logical groups and prevents naming conflicts between different components. |
| **Used By** | All modules |
| **Related Terms** | Module, Crate, Package, Scope |
| **Related Modules** | All modules |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY uses Rust's module system for namespacing. Each module is a separate crate with its own namespace. Public APIs are exported through lib.rs. Internal modules use pub(crate) visibility. |

---

## Noise Gate

| Field | Value |
|-------|-------|
| **Category** | Audio / Processing |
| **Definition** | An audio processor that attenuates signals below a defined threshold, effectively muting background noise when the user is not speaking. |
| **Purpose** | Reduces false wake word triggers and improves ASR accuracy by eliminating low-level ambient noise from the audio stream. |
| **Used By** | MOD-AUDIO, MOD-WAKE |
| **Related Terms** | VAD, Threshold, Attenuation, Gating |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY's noise gate operates before the wake word detector. It uses an adaptive threshold based on ambient noise floor estimation. The gate has configurable attack and release times to avoid cutting off speech beginnings. |

---

*End of 23_GLOSSARY_N.md*