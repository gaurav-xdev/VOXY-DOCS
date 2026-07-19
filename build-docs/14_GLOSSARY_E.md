# VOXY Glossary — E

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## E2E Testing (End-to-End Testing)

| Field | Value |
|-------|-------|
| **Category** | Quality / Testing |
| **Definition** | Testing methodology that validates the complete application flow from user input to system output, simulating real user scenarios. VOXY E2E tests verify voice command processing through the entire pipeline. |
| **Purpose** | Ensures that all modules work together correctly and that the system meets user-facing requirements. |
| **Used By** | QA Team, CI/CD Pipeline |
| **Related Terms** | Integration Test, Acceptance Test, Voice Command Test |
| **Related Modules** | All modules |
| **Related Specifications** | [06_DEVELOPER_WORKFLOW.md](06_DEVELOPER_WORKFLOW.md), [39_ENGINEERING_CHECKLIST.md](39_ENGINEERING_CHECKLIST.md) |
| **Architecture Notes** | VOXY E2E tests use synthetic audio files (pre-recorded voice commands) fed into the audio pipeline. Tests verify: wake word detection, transcription accuracy, intent resolution, action execution, and feedback delivery. Tests run on Windows 10 and Windows 11 VMs. |

---

## Embedding

| Field | Value |
|-------|-------|
| **Category** | AI / NLP |
| **Definition** | A dense vector representation of text (or audio) in a high-dimensional space where semantically similar items are closer together. VOXY uses embeddings for knowledge base retrieval and semantic search. |
| **Purpose** | Enables semantic understanding of text beyond keyword matching, supporting context-aware responses and document retrieval. |
| **Used By** | MOD-KB, MOD-NLU, MOD-LLM |
| **Related Terms** | Vector, Similarity, Cosine Distance, Transformer |
| **Related Modules** | MOD-KB |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY generates embeddings using a local ONNX model (e.g., all-MiniLM-L6-v2 quantized to INT8). Embeddings are 384-dimensional floats stored in an HNSW index for approximate nearest neighbor search. |

---

## Entity Extraction

| Field | Value |
|-------|-------|
| **Category** | AI / NLU |
| **Definition** | The process of identifying and classifying named entities (people, places, dates, applications) from text. In VOXY, entity extraction populates action parameters from user commands. |
| **Purpose** | Transforms unstructured user input into structured data that the Action Engine can use to execute commands. |
| **Used By** | MOD-NLU, MOD-ACTION |
| **Related Terms** | Named Entity Recognition, Slot Filling, Intent Classification |
| **Related Modules** | MOD-NLU |
| **Related Specifications** | [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY uses a lightweight ONNX model for entity extraction, running locally. Entities are typed (e.g., Application, FilePath, DateTime) and validated against the action schema before execution. |

---

## Epoch

| Field | Value |
|-------|-------|
| **Category** | AI / Training |
| **Definition** | One complete pass through the entire training dataset during model training. While VOXY primarily uses pre-trained models, epoch is relevant for fine-tuning custom wake word or intent models. |
| **Purpose** | Measures training progress and determines when to stop training (early stopping). |
| **Used By** | MOD-WAKE, MOD-NLU (training scripts) |
| **Related Terms** | Batch, Iteration, Learning Rate, Overfitting |
| **Related Modules** | MOD-WAKE, MOD-NLU |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY's training scripts use early stopping with a patience of 5 epochs. Models are fine-tuned on synthetic and real user data with data augmentation (noise injection, speed perturbation). |

---

## Error Propagation

| Field | Value |
|-------|-------|
| **Category** | Programming / Rust |
| **Definition** | The mechanism by which errors are passed up the call stack from the point of failure to a handler that can manage them. In Rust, this is done via the Result type and the ? operator. |
| **Purpose** | Ensures that errors are not silently ignored and that callers have the information needed to handle failures appropriately. |
| **Used By** | All Rust modules |
| **Related Terms** | Result, anyhow, thiserror, Context |
| **Related Modules** | All modules |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY uses thiserror for library errors (specific, typed) and anyhow for application errors (ergonomic, context-rich). The ? operator propagates errors with automatic conversion via From implementations. |

---

## Event Bus

| Field | Value |
|-------|-------|
| **Category** | Architecture / Pattern |
| **Definition** | See Bus (Event Bus) in [11_GLOSSARY_B.md](11_GLOSSARY_B.md). |
| **Purpose** | Central messaging infrastructure for decoupled module communication. |
| **Used By** | All modules |
| **Related Terms** | Pub/Sub, Message Queue, Event-Driven |
| **Related Modules** | MOD-EVENT |
| **Related Specifications** | [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md) |
| **Architecture Notes** | The Event Bus is implemented using Tokio broadcast channels with typed events. It supports fan-out (one publisher, many subscribers) and guarantees at-most-once delivery. |

---

*End of 14_GLOSSARY_E.md*
